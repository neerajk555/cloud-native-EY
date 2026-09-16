# Lab 6.3: Cognito Authorizer

**Module:** 6 — API Design & Management with Amazon API Gateway
**Difficulty:** Intermediate
**Duration:** ~15 minutes
**Cost:** ✅ Free Tier Eligible (Cognito: 50,000 MAUs free always; this lab uses 1 test user)

## Objective
Secure an API Gateway endpoint using an Amazon Cognito User Pool authorizer, so only authenticated users (holding a valid JWT) can call it — a standard pattern for user-facing cloud native APIs.

## Prerequisites
- Completed Lab 6.1 (reuses the same API and Lambda; redo Lab 6.1 first if you already cleaned it up)

## Steps

1. **Create a Cognito User Pool:**
   ```bash
   POOL_ID=$(aws cognito-idp create-user-pool --pool-name demo-user-pool --query 'UserPool.Id' --output text)
   echo $POOL_ID
   ```
2. **Create an app client** (no secret, for simplicity of CLI-based testing):
   ```bash
   CLIENT_ID=$(aws cognito-idp create-user-pool-client \
     --user-pool-id $POOL_ID \
     --client-name demo-app-client \
     --explicit-auth-flows ALLOW_USER_PASSWORD_AUTH ALLOW_REFRESH_TOKEN_AUTH \
     --query 'UserPoolClient.ClientId' --output text)
   ```
3. **Create a test user and set a permanent password:**
   ```bash
   aws cognito-idp admin-create-user \
     --user-pool-id $POOL_ID \
     --username testuser \
     --user-attributes Name=email,Value=testuser@example.com \
     --message-action SUPPRESS

   aws cognito-idp admin-set-user-password \
     --user-pool-id $POOL_ID \
     --username testuser \
     --password "TrainingPass123!" \
     --permanent
   ```
4. **Log in to get a JWT token:**
   ```bash
   AUTH_RESULT=$(aws cognito-idp initiate-auth \
     --auth-flow USER_PASSWORD_AUTH \
     --client-id $CLIENT_ID \
     --auth-parameters USERNAME=testuser,PASSWORD=TrainingPass123! \
     --query 'AuthenticationResult.IdToken' --output text)
   echo $AUTH_RESULT
   ```
5. **Attach a JWT authorizer to your HTTP API** from Lab 6.1 (reuse `$API_ID` — re-fetch it if starting a new terminal session):
   ```bash
   API_ID=$(aws apigatewayv2 get-apis --query "Items[?Name=='cloudnative-demo-api'].ApiId" --output text)
   AUTHORIZER_ID=$(aws apigatewayv2 create-authorizer \
     --api-id $API_ID \
     --authorizer-type JWT \
     --identity-source '$request.header.Authorization' \
     --name cognito-authorizer \
     --jwt-configuration Audience=$CLIENT_ID,Issuer=https://cognito-idp.us-east-1.amazonaws.com/$POOL_ID \
     --query 'AuthorizerId' --output text)
   ```
6. **Update the API's default route to require this authorizer:**
   ```bash
   ROUTE_ID=$(aws apigatewayv2 get-routes --api-id $API_ID --query "Items[0].RouteId" --output text)
   aws apigatewayv2 update-route \
     --api-id $API_ID --route-id $ROUTE_ID \
     --authorization-type JWT --authorizer-id $AUTHORIZER_ID
   ```

## Expected Result / Validation
1. **Call without a token** (should be rejected):
   ```bash
   curl -i $(aws apigatewayv2 get-apis --query "Items[?ApiId=='$API_ID'].ApiEndpoint" --output text)
   ```
   Expect `401 Unauthorized`.
2. **Call with the token** (should succeed):
   ```bash
   curl -i -H "Authorization: $AUTH_RESULT" $(aws apigatewayv2 get-apis --query "Items[?ApiId=='$API_ID'].ApiEndpoint" --output text)
   ```
   Expect `200 OK` with the Lambda's JSON response from Lab 6.1.

## Cleanup
```bash
aws apigatewayv2 update-route --api-id $API_ID --route-id $ROUTE_ID --authorization-type NONE
aws apigatewayv2 delete-authorizer --api-id $API_ID --authorizer-id $AUTHORIZER_ID
aws cognito-idp delete-user-pool-client --user-pool-id $POOL_ID --client-id $CLIENT_ID
aws cognito-idp delete-user-pool --user-pool-id $POOL_ID
```
Then follow Lab 6.1's cleanup steps to remove the API and Lambda entirely if you're done with this API.

Verify:
```bash
aws cognito-idp list-user-pools --max-results 20 --query "UserPools[?Name=='demo-user-pool']"
```
Should return an empty list.

## Troubleshooting
- **`NotAuthorizedException` during login** → Confirm `admin-set-user-password` used `--permanent`, otherwise Cognito forces a password-change flow on first login.
- **401 even with a valid token** → Double-check the `Authorization` header value includes the raw JWT with no `Bearer ` prefix needed for HTTP API JWT authorizers reading from `$request.header.Authorization` as configured here — but do confirm no extra whitespace was introduced when copying the token.
