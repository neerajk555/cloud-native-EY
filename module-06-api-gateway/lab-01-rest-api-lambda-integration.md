# Lab 6.1: REST API + Lambda Integration

**Module:** 6 — API Design & Management with Amazon API Gateway
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free Tier Eligible (1M API Gateway calls + 1M Lambda requests free/month for 12 months)

## Objective
Build a simple public REST API backed by a Lambda function, understanding the core API Gateway → Lambda integration pattern used by most cloud native serverless APIs.

## Prerequisites
- Completed Lab 1.1 (CLI configured)

## Steps

1. **Create the Lambda function.** `index.js`:
   ```javascript
   exports.handler = async (event) => {
     return {
       statusCode: 200,
       headers: { "Content-Type": "application/json" },
       body: JSON.stringify({ message: "Hello from your cloud native API!" })
     };
   };
   ```
   ```bash
   zip function.zip index.js
   ```
2. **Create the execution role:**
   ```bash
   aws iam create-role \
     --role-name apigw-lambda-demo-role \
     --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":"lambda.amazonaws.com"},"Action":"sts:AssumeRole"}]}'
   aws iam attach-role-policy \
     --role-name apigw-lambda-demo-role \
     --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
   ROLE_ARN=$(aws iam get-role --role-name apigw-lambda-demo-role --query 'Role.Arn' --output text)
   ```
3. **Create the Lambda function** (wait ~10 seconds after step 2 for IAM propagation first):
   ```bash
   aws lambda create-function \
     --function-name api-demo-fn \
     --runtime nodejs20.x \
     --handler index.handler \
     --zip-file fileb://function.zip \
     --role $ROLE_ARN \
     --region us-east-1
   ```
4. **Create an HTTP API** (the simpler, cheaper API Gateway type) with a Lambda proxy integration:
   ```bash
   LAMBDA_ARN=$(aws lambda get-function --function-name api-demo-fn --query 'Configuration.FunctionArn' --output text)
   API_ID=$(aws apigatewayv2 create-api \
     --name cloudnative-demo-api \
     --protocol-type HTTP \
     --target $LAMBDA_ARN \
     --query 'ApiId' --output text)
   echo $API_ID
   ```
   *(Using `--target` auto-creates the route, integration, and a default `$default` stage in one command — the fastest beginner path.)*
5. **Grant API Gateway permission to invoke the Lambda:**
   ```bash
   aws lambda add-permission \
     --function-name api-demo-fn \
     --statement-id apigateway-invoke \
     --action lambda:InvokeFunction \
     --principal apigateway.amazonaws.com \
     --source-arn "arn:aws:execute-api:us-east-1:$(aws sts get-caller-identity --query Account --output text):$API_ID/*/*"
   ```
6. **Get your API's public endpoint:**
   ```bash
   aws apigatewayv2 get-apis --query "Items[?ApiId=='$API_ID'].ApiEndpoint" --output text
   ```

## Expected Result / Validation
```bash
curl $(aws apigatewayv2 get-apis --query "Items[?ApiId=='$API_ID'].ApiEndpoint" --output text)
```
Should return:
```json
{"message":"Hello from your cloud native API!"}
```

## Cleanup
```bash
aws apigatewayv2 delete-api --api-id $API_ID
aws lambda delete-function --function-name api-demo-fn
aws iam detach-role-policy --role-name apigw-lambda-demo-role --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
aws iam delete-role --role-name apigw-lambda-demo-role
```
Verify:
```bash
aws apigatewayv2 get-apis --query "Items[?Name=='cloudnative-demo-api']"
```
Should return an empty list.

## Troubleshooting
- **403 Forbidden calling the endpoint** → Re-check step 5's `add-permission` command ran without error; the `source-arn` must exactly match your API ID.
- **Lambda not found error creating the API** → Wait a few seconds after Lambda creation completes before running step 4.
