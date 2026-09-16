# Lab 9.3: Distributed Tracing with AWS X-Ray

**Module:** 9 — Observability: Logging, Monitoring & Tracing on AWS
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free Tier Eligible (100,000 traces recorded + 1,000,000 traces retrieved/scanned free/month, always free)

## Objective
Instrument a Lambda function with AWS X-Ray tracing and view the resulting trace, including timing breakdowns of an internal (simulated) downstream call.

## Prerequisites
- Completed Lab 1.1

## Steps

1. **Create a traced Lambda function.** `index.js`:
   ```javascript
   const AWSXRay = require('aws-xray-sdk-core');
   const https = AWSXRay.captureHTTPs(require('https'));

   exports.handler = async (event) => {
     const segment = AWSXRay.getSegment();
     const subsegment = segment.addNewSubsegment('simulated-downstream-call');
     await new Promise(resolve => setTimeout(resolve, 200));
     subsegment.close();
     return { statusCode: 200, body: "Traced request complete" };
   };
   ```
2. **Package with dependencies:**
   ```bash
   mkdir xray-demo && cd xray-demo
   # place index.js here
   npm init -y
   npm install aws-xray-sdk-core
   zip -r function.zip .
   ```
3. **Create the execution role with X-Ray write permissions:**
   ```bash
   aws iam create-role \
     --role-name xray-demo-role \
     --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":"lambda.amazonaws.com"},"Action":"sts:AssumeRole"}]}'
   aws iam attach-role-policy --role-name xray-demo-role --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
   aws iam attach-role-policy --role-name xray-demo-role --policy-arn arn:aws:iam::aws:policy/AWSXRayDaemonWriteAccess
   ROLE_ARN=$(aws iam get-role --role-name xray-demo-role --query 'Role.Arn' --output text)
   ```
4. **Create the function with active tracing enabled:**
   ```bash
   aws lambda create-function \
     --function-name xray-demo-fn \
     --runtime nodejs20.x \
     --handler index.handler \
     --zip-file fileb://function.zip \
     --role $ROLE_ARN \
     --tracing-config Mode=Active \
     --region us-east-1
   ```
5. **Invoke it a few times to generate traces:**
   ```bash
   for i in {1..5}; do aws lambda invoke --function-name xray-demo-fn --cli-binary-format raw-in-base64-out /dev/null; sleep 1; done
   ```

## Expected Result / Validation
1. Wait ~30 seconds for traces to be processed, then:
   ```bash
   aws xray get-trace-summaries \
     --start-time $(date -u -d '-10 minutes' +%s) \
     --end-time $(date -u +%s)
   ```
   *(On Mac, replace `date -u -d '-10 minutes'` with `date -u -v-10M`.)*
2. You should see trace summaries with your Lambda's duration. For a richer view, open the **AWS X-Ray console → Traces**, select a recent trace, and view the **timeline**, which should show the `simulated-downstream-call` subsegment taking ~200ms nested inside the overall Lambda invocation — exactly the kind of breakdown that helps diagnose latency in real distributed systems.

## Cleanup
```bash
aws lambda delete-function --function-name xray-demo-fn
aws iam detach-role-policy --role-name xray-demo-role --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
aws iam detach-role-policy --role-name xray-demo-role --policy-arn arn:aws:iam::aws:policy/AWSXRayDaemonWriteAccess
aws iam delete-role --role-name xray-demo-role
cd .. && rm -rf xray-demo
```
X-Ray trace data automatically expires after 30 days at no cost — no manual trace deletion is needed.

Verify:
```bash
aws lambda get-function --function-name xray-demo-fn
```
Should return a `ResourceNotFoundException`.

## Troubleshooting
- **No traces appear** → Confirm `--tracing-config Mode=Active` was set on function creation; this is the setting that actually enables X-Ray.
- **`get-trace-summaries` returns empty** → Double-check your `start-time`/`end-time` window actually covers when you ran the invocations, and that a minute has passed for trace processing.
