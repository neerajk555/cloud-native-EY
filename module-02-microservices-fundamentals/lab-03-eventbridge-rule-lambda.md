# Lab 2.3: EventBridge Rule → Lambda

**Module:** 2 — Microservices Architecture Fundamentals
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free Tier Eligible (EventBridge: 14M events free; Lambda: 1M requests/month free)

## Objective
Wire up an event-driven trigger: an EventBridge rule that invokes a Lambda function on a schedule, mirroring how cloud native services react to events instead of being polled.

## Prerequisites
- Completed Lab 2.1
- IAM permissions from Lab 1.1 (AdministratorAccess is sufficient)

## Steps

1. **Create a minimal Lambda function.** Create a file `index.js`:
   ```javascript
   exports.handler = async (event) => {
     console.log("Event received:", JSON.stringify(event));
     return { statusCode: 200, body: "Processed event" };
   };
   ```
2. **Zip it:**
   ```bash
   zip function.zip index.js
   ```
3. **Create an execution role for Lambda** (one-time, minimal role):
   ```bash
   aws iam create-role \
     --role-name lambda-eventbridge-demo-role \
     --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":"lambda.amazonaws.com"},"Action":"sts:AssumeRole"}]}'

   aws iam attach-role-policy \
     --role-name lambda-eventbridge-demo-role \
     --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
   ```
   Wait ~10 seconds for IAM role propagation, then get its ARN:
   ```bash
   ROLE_ARN=$(aws iam get-role --role-name lambda-eventbridge-demo-role --query 'Role.Arn' --output text)
   ```
4. **Create the Lambda function:**
   ```bash
   aws lambda create-function \
     --function-name eventbridge-demo-fn \
     --runtime nodejs20.x \
     --handler index.handler \
     --zip-file fileb://function.zip \
     --role $ROLE_ARN \
     --region us-east-1
   ```
5. **Create an EventBridge rule** that fires every 5 minutes:
   ```bash
   aws events put-rule \
     --name every-5-min-rule \
     --schedule-expression "rate(5 minutes)"
   ```
6. **Grant EventBridge permission to invoke the Lambda:**
   ```bash
   aws lambda add-permission \
     --function-name eventbridge-demo-fn \
     --statement-id eventbridge-invoke \
     --action lambda:InvokeFunction \
     --principal events.amazonaws.com \
     --source-arn $(aws events describe-rule --name every-5-min-rule --query 'Arn' --output text)
   ```
7. **Add the Lambda as the rule's target:**
   ```bash
   LAMBDA_ARN=$(aws lambda get-function --function-name eventbridge-demo-fn --query 'Configuration.FunctionArn' --output text)
   aws events put-targets \
     --rule every-5-min-rule \
     --targets "Id"="1","Arn"="$LAMBDA_ARN"
   ```

## Expected Result / Validation
1. Wait 5–10 minutes.
2. Check the Lambda's CloudWatch Logs:
   ```bash
   aws logs tail /aws/lambda/eventbridge-demo-fn --since 15m
   ```
   You should see log entries containing `"Event received:"` — proof the EventBridge rule successfully triggered the Lambda without any direct coupling between the two.

## Cleanup
Run these in order to remove every resource created in this lab:
```bash
aws events remove-targets --rule every-5-min-rule --ids "1"
aws events delete-rule --name every-5-min-rule
aws lambda delete-function --function-name eventbridge-demo-fn
aws iam detach-role-policy --role-name lambda-eventbridge-demo-role --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
aws iam delete-role --role-name lambda-eventbridge-demo-role
```
Verify no rules or functions remain:
```bash
aws events list-rules --name-prefix every-5-min
aws lambda list-functions --query "Functions[?FunctionName=='eventbridge-demo-fn']"
```
Both should return empty.

## Troubleshooting
- **`AccessDeniedException` creating the role** → Confirm you're using the `cloudnative-student` user with AdministratorAccess from Lab 1.1.
- **Lambda never triggers** → Double-check `put-targets` used the correct Lambda ARN and that `add-permission` succeeded without error.
- **Role not found when creating function** → Wait 10–15 seconds after creating an IAM role before referencing it; IAM propagation is eventually consistent.
