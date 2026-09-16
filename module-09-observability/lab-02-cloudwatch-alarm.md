# Lab 9.2: CloudWatch Alarm

**Module:** 9 — Observability: Logging, Monitoring & Tracing on AWS
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free Tier Eligible (10 alarms free always; SNS email notifications free up to 1,000/month)

## Objective
Create a CloudWatch Alarm on a Lambda function's error metric, wired to an SNS topic, so you get notified automatically when something goes wrong — a core practice for SLO/SLA-driven operations.

## Prerequisites
- Completed Lab 1.1
- Access to a real email inbox

## Steps

1. **Create a Lambda function that sometimes fails** (simulating a real production error rate):
   `index.js`:
   ```javascript
   exports.handler = async () => {
     if (Math.random() < 0.5) {
       throw new Error("Simulated failure");
     }
     return { statusCode: 200, body: "OK" };
   };
   ```
   ```bash
   zip function.zip index.js
   aws iam create-role \
     --role-name alarm-demo-role \
     --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":"lambda.amazonaws.com"},"Action":"sts:AssumeRole"}]}'
   aws iam attach-role-policy --role-name alarm-demo-role --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
   ROLE_ARN=$(aws iam get-role --role-name alarm-demo-role --query 'Role.Arn' --output text)
   aws lambda create-function \
     --function-name alarm-demo-fn --runtime nodejs20.x --handler index.handler \
     --zip-file fileb://function.zip --role $ROLE_ARN --region us-east-1
   ```
2. **Create an SNS topic and subscribe your email** (skip if you still have the topic from Module 2, Lab 2.2 — otherwise recreate):
   ```bash
   TOPIC_ARN=$(aws sns create-topic --name alarm-notifications --query 'TopicArn' --output text)
   aws sns subscribe --topic-arn $TOPIC_ARN --protocol email --notification-endpoint your-email@example.com
   ```
   Check your inbox and click **Confirm subscription**.
3. **Create a CloudWatch Alarm** on the Lambda's `Errors` metric — trigger if 3+ errors occur in a 1-minute period:
   ```bash
   aws cloudwatch put-metric-alarm \
     --alarm-name lambda-error-alarm \
     --namespace AWS/Lambda \
     --metric-name Errors \
     --dimensions Name=FunctionName,Value=alarm-demo-fn \
     --statistic Sum \
     --period 60 \
     --threshold 3 \
     --comparison-operator GreaterThanOrEqualToThreshold \
     --evaluation-periods 1 \
     --alarm-actions $TOPIC_ARN \
     --treat-missing-data notBreaching
   ```
4. **Generate load to trigger errors** — invoke the function 10 times:
   ```bash
   for i in {1..10}; do aws lambda invoke --function-name alarm-demo-fn --cli-binary-format raw-in-base64-out /dev/null; done
   ```

## Expected Result / Validation
1. Check the alarm state:
   ```bash
   aws cloudwatch describe-alarms --alarm-names lambda-error-alarm --query 'MetricAlarms[0].StateValue'
   ```
   Given a ~50% failure rate over 10 invocations, this should show `"ALARM"` within a couple of minutes (metrics can take 1-2 minutes to publish).
2. Check your email — you should receive an SNS notification with subject like "ALARM: lambda-error-alarm" describing the breach.

## Cleanup
```bash
aws cloudwatch delete-alarms --alarm-names lambda-error-alarm
aws lambda delete-function --function-name alarm-demo-fn
aws iam detach-role-policy --role-name alarm-demo-role --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
aws iam delete-role --role-name alarm-demo-role
aws sns delete-topic --topic-arn $TOPIC_ARN
```
Verify:
```bash
aws cloudwatch describe-alarms --alarm-names lambda-error-alarm
```
Should show an empty `MetricAlarms` list.

## Troubleshooting
- **Alarm stays in `INSUFFICIENT_DATA`** → Metrics can take 1–2 minutes to appear after invocations; wait and re-check.
- **No email received** → Confirm you clicked the SNS subscription confirmation link; unconfirmed subscriptions silently drop notifications.
