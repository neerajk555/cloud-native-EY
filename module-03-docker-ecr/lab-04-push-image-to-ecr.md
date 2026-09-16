# Lab 3.4: Push an Image to Amazon ECR

**Module:** 3 — Containerization with Docker & Amazon ECR
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free Tier Eligible (500 MB/month private ECR storage free for 12 months; this image is a few MB)

## Objective
Create a private Amazon ECR repository and push your locally built image to it, so it can later be pulled by ECS/EKS.

## Prerequisites
- Completed Lab 3.3 (image `hello-cloudnative-app:1.0.0` built locally)
- AWS CLI configured (Lab 1.1)

## Steps

1. **Get your AWS account ID:**
   ```bash
   ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
   echo $ACCOUNT_ID
   ```
2. **Create an ECR repository:**
   ```bash
   aws ecr create-repository \
     --repository-name hello-cloudnative-app \
     --region us-east-1
   ```
3. **Authenticate Docker to your ECR registry:**
   ```bash
   aws ecr get-login-password --region us-east-1 | \
     docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com
   ```
4. **Rebuild the image if needed** (skip if you still have it from Lab 3.3):
   ```bash
   cd hello-cloudnative-app
   docker build -t hello-cloudnative-app:1.0.0 .
   ```
5. **Tag the image for ECR:**
   ```bash
   docker tag hello-cloudnative-app:1.0.0 \
     $ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/hello-cloudnative-app:1.0.0
   ```
6. **Push the image:**
   ```bash
   docker push $ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/hello-cloudnative-app:1.0.0
   ```

## Expected Result / Validation
```bash
aws ecr describe-images --repository-name hello-cloudnative-app
```
The output should list one image with tag `1.0.0` and show its size in bytes, confirming the push succeeded.

## Cleanup
Delete the repository (this also deletes all images inside it — no separate image deletion needed):
```bash
aws ecr delete-repository --repository-name hello-cloudnative-app --force
```
Verify:
```bash
aws ecr describe-repositories --repository-names hello-cloudnative-app
```
This should return a `RepositoryNotFoundException`, confirming cleanup succeeded.

## Troubleshooting
- **`docker login` fails** → Confirm your CLI region matches `us-east-1` and your IAM user has ECR permissions (AdministratorAccess from Lab 1.1 covers this).
- **`no basic auth credentials`** on push → Re-run the `get-login-password` command; the auth token expires after 12 hours.
- **Push is slow** → Normal for the first push of a base layer; subsequent pushes of the same base image are much faster due to layer caching.
