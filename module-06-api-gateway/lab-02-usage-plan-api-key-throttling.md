# Lab 6.2: Usage Plans, API Keys & Throttling

**Module:** 6 — API Design & Management with Amazon API Gateway
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free Tier Eligible (this lab uses REST APIs, which also fall under the 1M free calls/month for 12 months)

## Objective
Protect an API using an API key and a usage plan with rate limiting/throttling — essential for controlling how much load consumers can put on your backend.

## Prerequisites
- Completed Lab 1.1
- This lab creates its own REST API from scratch (uses the REST API type specifically, since usage plans/API keys are a REST API Gateway feature)

## Steps

1. **Create a REST API:**
   ```bash
   API_ID=$(aws apigateway create-rest-api --name usage-plan-demo-api --query 'id' --output text)
   ROOT_ID=$(aws apigateway get-resources --rest-api-id $API_ID --query 'items[0].id' --output text)
   ```
2. **Create a `/hello` resource with a mock GET method** (no Lambda needed — API Gateway can mock responses directly, useful for this focused lab):
   ```bash
   RESOURCE_ID=$(aws apigateway create-resource --rest-api-id $API_ID --parent-id $ROOT_ID --path-part hello --query 'id' --output text)

   aws apigateway put-method \
     --rest-api-id $API_ID --resource-id $RESOURCE_ID \
     --http-method GET --authorization-type NONE --api-key-required

   aws apigateway put-integration \
     --rest-api-id $API_ID --resource-id $RESOURCE_ID \
     --http-method GET --type MOCK \
     --request-templates '{"application/json":"{\"statusCode\": 200}"}'

   aws apigateway put-method-response \
     --rest-api-id $API_ID --resource-id $RESOURCE_ID \
     --http-method GET --status-code 200

   aws apigateway put-integration-response \
     --rest-api-id $API_ID --resource-id $RESOURCE_ID \
     --http-method GET --status-code 200 \
     --response-templates '{"application/json":"{\"message\": \"Protected hello endpoint\"}"}'
   ```
   Note `--api-key-required` on `put-method` — this is what enforces key checking.
3. **Deploy the API to a stage:**
   ```bash
   aws apigateway create-deployment --rest-api-id $API_ID --stage-name dev
   ```
4. **Create an API key:**
   ```bash
   KEY_ID=$(aws apigateway create-api-key --name demo-key --enabled --query 'id' --output text)
   ```
5. **Create a usage plan with throttling** (limit: 2 requests/second, 5 request burst, 100 requests/month quota):
   ```bash
   PLAN_ID=$(aws apigateway create-usage-plan \
     --name demo-usage-plan \
     --api-stages apiId=$API_ID,stage=dev \
     --throttle burstLimit=5,rateLimit=2 \
     --quota limit=100,period=MONTH \
     --query 'id' --output text)
   ```
6. **Link the API key to the usage plan:**
   ```bash
   aws apigateway create-usage-plan-key \
     --usage-plan-id $PLAN_ID --key-id $KEY_ID --key-type API_KEY
   ```
7. **Get the actual key value:**
   ```bash
   API_KEY_VALUE=$(aws apigateway get-api-key --api-key $KEY_ID --include-value --query 'value' --output text)
   ```

## Expected Result / Validation
1. **Call without a key** (should be rejected):
   ```bash
   curl -i https://$API_ID.execute-api.us-east-1.amazonaws.com/dev/hello
   ```
   Expect `403 Forbidden` with `{"message":"Forbidden"}`.
2. **Call with the key** (should succeed):
   ```bash
   curl -i -H "x-api-key: $API_KEY_VALUE" https://$API_ID.execute-api.us-east-1.amazonaws.com/dev/hello
   ```
   Expect `200 OK` with `{"message": "Protected hello endpoint"}`.
3. **Trigger throttling** (fire more than 5 requests almost simultaneously):
   ```bash
   for i in {1..10}; do curl -s -o /dev/null -w "%{http_code}\n" -H "x-api-key: $API_KEY_VALUE" https://$API_ID.execute-api.us-east-1.amazonaws.com/dev/hello; done
   ```
   You should see a mix of `200` responses and `429` (Too Many Requests) responses once the burst limit is exceeded.

## Cleanup
```bash
aws apigateway delete-usage-plan-key --usage-plan-id $PLAN_ID --key-id $KEY_ID
aws apigateway delete-usage-plan --usage-plan-id $PLAN_ID
aws apigateway delete-api-key --api-key $KEY_ID
aws apigateway delete-rest-api --rest-api-id $API_ID
```
Verify:
```bash
aws apigateway get-rest-apis --query "items[?name=='usage-plan-demo-api']"
```
Should return an empty list.

## Troubleshooting
- **Every call returns 403 even with the key** → Double-check the usage plan (step 6) correctly links the key ID and that the API stage in step 5 matches `dev` exactly.
- **No 429s appear in step 3** → Requests may be too spaced out over the network; try increasing the loop count to `{1..30}` or reducing your machine's network latency variance.
