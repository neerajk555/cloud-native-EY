# Lab 11.2: SQS + Lambda Event Processing

**Module:** 11 — Cloud Native Data Management on AWS
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free Tier Eligible (SQS: 1M requests free; Lambda: 1M requests free, always/12-month free tier)

## Objective
Wire an SQS queue directly to a Lambda function as an event source, so messages are processed automatically and asynchronously — a foundational event-driven data processing pattern.

## Prerequisites
- Completed Lab 1.1

## Steps

1. **Create the processing queue:**
   ```bash
   QUEUE_URL=$(aws sqs create-queue --queue-name order-processing-queue --region us-east-1 --query 'QueueUrl' --output text)
   QUEUE_ARN=$(aws sqs get-queue-attributes --queue-url $QUEUE_URL --attribute-names QueueArn --query 'Attributes.QueueArn' --output text)
   ```
2. **Create the processing Lambda function.** `index.js`:
   ```javascript
   exports.handler = async (event) => {
     for (const record of event.Records) {
       const body = JSON.parse(record.body);
       console.log(`Processing order ${body.orderId} with status ${body.status}`);
     }
     return { statusCode: 200 };
   };
   ```
   ```bash
   zip function.zip index.js
   aws iam create-role \
     --role-name sqs-lambda-demo-role \
     --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":"lambda.amazonaws.com"},"Action":"sts:AssumeRole"}]}'
   aws iam attach-role-policy --role-name sqs-lambda-demo-role --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
   aws iam attach-role-policy --role-name sqs-lambda-demo-role --policy-arn arn:aws:iam::aws:policy/AWSLambdaSQSQueueExecutionRole
   ROLE_ARN=$(aws iam get-role --role-name sqs-lambda-demo-role --query 'Role.Arn' --output text)
   ```
3. **Create the Lambda function** (wait ~10 seconds for IAM propagation after step 2):
   ```bash
   aws lambda create-function \
     --function-name order-processor-fn \
     --runtime nodejs20.x \
     --handler index.handler \
     --zip-file fileb://function.zip \
     --role $ROLE_ARN \
     --region us-east-1
   ```
4. **Connect the queue to the Lambda as an event source:**
   ```bash
   aws lambda create-event-source-mapping \
     --function-name order-processor-fn \
     --event-source-arn $QUEUE_ARN \
     --batch-size 5
   ```
5. **Send a few messages to the queue** — no manual receive/process step needed, Lambda will do it automatically:
   ```bash
   aws sqs send-message --queue-url $QUEUE_URL --message-body '{"orderId":"2001","status":"CREATED"}'
   aws sqs send-message --queue-url $QUEUE_URL --message-body '{"orderId":"2002","status":"CREATED"}'
   aws sqs send-message --queue-url $QUEUE_URL --message-body '{"orderId":"2003","status":"CREATED"}'
   ```

## Expected Result / Validation
Wait ~10–20 seconds, then check the Lambda's logs:
```bash
aws logs tail /aws/lambda/order-processor-fn --since 5m
```
You should see log lines for all three orders (`Processing order 2001...`, `2002`, `2003`) — **without ever calling `receive-message` yourself**. Also confirm the queue has drained:
```bash
aws sqs get-queue-attributes --queue-url $QUEUE_URL --attribute-names ApproximateNumberOfMessages
```
Should show `ApproximateNumberOfMessages: "0"`.

## Cleanup
```bash
MAPPING_UUID=$(aws lambda list-event-source-mappings --function-name order-processor-fn --query 'EventSourceMappings[0].UUID' --output text)
aws lambda delete-event-source-mapping --uuid $MAPPING_UUID
aws lambda delete-function --function-name order-processor-fn
aws iam detach-role-policy --role-name sqs-lambda-demo-role --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
aws iam detach-role-policy --role-name sqs-lambda-demo-role --policy-arn arn:aws:iam::aws:policy/AWSLambdaSQSQueueExecutionRole
aws iam delete-role --role-name sqs-lambda-demo-role
aws sqs delete-queue --queue-url $QUEUE_URL
```
Verify:
```bash
aws lambda get-function --function-name order-processor-fn
```
Should return a `ResourceNotFoundException`.

## Troubleshooting
- **Messages never get processed** → Confirm `create-event-source-mapping` succeeded and check its `State` is `Enabled`: `aws lambda list-event-source-mappings --function-name order-processor-fn`.
- **`AccessDenied` errors in Lambda logs** → Confirm the `AWSLambdaSQSQueueExecutionRole` policy was attached — this grants the Lambda permission to poll SQS.
