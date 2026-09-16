# Lab 10.3: Secrets Manager Retrieve-in-App

**Module:** 10 — Cloud Native Security on AWS
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free Tier Eligible (Secrets Manager: 30-day free trial per secret; this lab deletes the secret before any charge would apply)

## Objective
Write a small Node.js script simulating an application that retrieves a database credential from AWS Secrets Manager at runtime, instead of hardcoding it — and see automatic secret rotation configuration (without waiting for an actual rotation).

## Prerequisites
- Completed Lab 1.1
- Node.js installed locally

## Steps

1. **Create a secret:**
   ```bash
   aws secretsmanager create-secret \
     --name training/app-db-credentials \
     --secret-string '{"username":"appuser","password":"Tr@iningPass2024"}' \
     --region us-east-1
   ```
2. **Set up a small Node.js project:**
   ```bash
   mkdir secrets-demo && cd secrets-demo
   npm init -y
   npm install @aws-sdk/client-secrets-manager
   ```
3. **Create `app.js`** simulating an application startup that fetches its DB credentials:
   ```javascript
   const { SecretsManagerClient, GetSecretValueCommand } = require("@aws-sdk/client-secrets-manager");

   const client = new SecretsManagerClient({ region: "us-east-1" });

   async function getDbCredentials() {
     const response = await client.send(
       new GetSecretValueCommand({ SecretId: "training/app-db-credentials" })
     );
     const secret = JSON.parse(response.SecretString);
     console.log(`Connecting to DB as user: ${secret.username}`);
     console.log("Password retrieved securely at runtime (not shown in logs, as a best practice).");
     return secret;
   }

   getDbCredentials();
   ```
4. **Run it** (uses the CLI credentials from Lab 1.1 automatically via the default credential chain):
   ```bash
   node app.js
   ```
5. **(Optional) enable automatic rotation configuration** to see how it's set up, without waiting for an actual rotation cycle:
   ```bash
   aws secretsmanager describe-secret --secret-id training/app-db-credentials
   ```
   Note the `RotationEnabled: false` field — in production, you'd attach a rotation Lambda function and set a `RotationRules` schedule here (covered conceptually; full rotation Lambda setup is beyond this 15-minute lab).

## Expected Result / Validation
Running `node app.js` should print:
```
Connecting to DB as user: appuser
Password retrieved securely at runtime (not shown in logs, as a best practice).
```
This demonstrates the core pattern: **the password never appears in your application code, environment variables, or version control** — it's fetched fresh from Secrets Manager at runtime using the app's IAM identity.

## Cleanup
```bash
aws secretsmanager delete-secret --secret-id training/app-db-credentials --force-delete-without-recovery --region us-east-1
cd .. && rm -rf secrets-demo
```
Verify:
```bash
aws secretsmanager list-secrets --query "SecretList[?Name=='training/app-db-credentials']"
```
Should return an empty list.

## Troubleshooting
- **`AccessDeniedException` in Node.js** → Confirm your local AWS CLI credentials (Lab 1.1) are still valid: `aws sts get-caller-identity`.
- **`ResourceNotFoundException`** → Confirm the secret name matches exactly `training/app-db-credentials` and the region is `us-east-1`.
