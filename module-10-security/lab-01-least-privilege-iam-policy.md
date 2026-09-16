# Lab 10.1: Least-Privilege IAM Policy

**Module:** 10 — Cloud Native Security on AWS
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free (IAM has no cost)

## Objective
Write and attach a tightly scoped, least-privilege IAM policy (instead of a broad managed policy) for a specific task, then verify it blocks unintended actions.

## Prerequisites
- Completed Lab 1.1

## Steps

1. **Create a test IAM user** representing a hypothetical "read-only S3 auditor":
   ```bash
   aws iam create-user --user-name s3-auditor-demo
   ```
2. **Write a least-privilege policy document** — `s3-readonly-policy.json` — that only allows listing and reading objects in one specific bucket, nothing else:
   ```bash
   BUCKET_NAME="cloudnative-least-priv-demo-$(date +%s)"
   aws s3 mb s3://$BUCKET_NAME --region us-east-1
   echo "test file" > test.txt
   aws s3 cp test.txt s3://$BUCKET_NAME/test.txt
   ```
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": ["s3:ListBucket"],
         "Resource": "arn:aws:s3:::BUCKET_NAME_PLACEHOLDER"
       },
       {
         "Effect": "Allow",
         "Action": ["s3:GetObject"],
         "Resource": "arn:aws:s3:::BUCKET_NAME_PLACEHOLDER/*"
       }
     ]
   }
   ```
   ```bash
   sed -i "s|BUCKET_NAME_PLACEHOLDER|$BUCKET_NAME|g" s3-readonly-policy.json
   ```
3. **Create and attach the policy:**
   ```bash
   POLICY_ARN=$(aws iam create-policy \
     --policy-name s3-readonly-demo-policy \
     --policy-document file://s3-readonly-policy.json \
     --query 'Policy.Arn' --output text)
   aws iam attach-user-policy --user-name s3-auditor-demo --policy-arn $POLICY_ARN
   ```
4. **Create access keys for this test user** so you can verify its permissions:
   ```bash
   aws iam create-access-key --user-name s3-auditor-demo
   ```
   Copy the `AccessKeyId` and `SecretAccessKey`.

## Expected Result / Validation
1. **Configure a separate CLI profile** for this restricted user:
   ```bash
   aws configure --profile s3-auditor-demo
   ```
   Enter the access key/secret from step 4, region `us-east-1`.
2. **Confirm it CAN read** (should succeed):
   ```bash
   aws s3 cp s3://$BUCKET_NAME/test.txt - --profile s3-auditor-demo
   ```
3. **Confirm it CANNOT delete** (should fail with `AccessDenied`):
   ```bash
   aws s3 rm s3://$BUCKET_NAME/test.txt --profile s3-auditor-demo
   ```
4. **Confirm it CANNOT access other buckets** (should fail with `AccessDenied`):
   ```bash
   aws s3 ls --profile s3-auditor-demo
   ```
   This last command actually requires a broader `s3:ListAllMyBuckets` permission the policy deliberately excludes — confirming least privilege is working as designed.

## Cleanup
```bash
aws iam detach-user-policy --user-name s3-auditor-demo --policy-arn $POLICY_ARN
aws iam list-access-keys --user-name s3-auditor-demo --query 'AccessKeyMetadata[0].AccessKeyId' --output text | xargs -I{} aws iam delete-access-key --user-name s3-auditor-demo --access-key-id {}
aws iam delete-user --user-name s3-auditor-demo
aws iam delete-policy --policy-arn $POLICY_ARN
aws s3 rm s3://$BUCKET_NAME --recursive
aws s3 rb s3://$BUCKET_NAME
rm -f ~/.aws/credentials.bak 2>/dev/null
```
Also remove the `s3-auditor-demo` profile from your local `~/.aws/credentials` and `~/.aws/config` files (open them in a text editor and delete that profile's block).

Verify:
```bash
aws iam get-user --user-name s3-auditor-demo
```
Should return a `NoSuchEntity` error.

## Troubleshooting
- **Policy creation fails with malformed JSON** → Validate with `cat s3-readonly-policy.json | python3 -m json.tool` before running `create-policy`.
- **`aws configure --profile` overwrote your main profile** → Always double check you're passing `--profile s3-auditor-demo` when testing, and your default profile remains `cloudnative-student` from Lab 1.1.
