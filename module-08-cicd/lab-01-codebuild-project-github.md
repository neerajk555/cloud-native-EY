# Lab 8.1: CodeBuild Project from a GitHub Repo

**Module:** 8 — CI/CD for Cloud Native Applications on AWS
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free Tier Eligible (100 build minutes/month free on `general1.small`, always free tier)

## Objective
Create an AWS CodeBuild project connected to a public GitHub repository, define a `buildspec.yml`, and run your first automated build.

## Prerequisites
- Completed Lab 1.1 (CLI configured)
- A GitHub account (free) — you'll fork a small public sample repo
- Fork this sample repo to your own GitHub account: https://github.com/aws-samples/aws-codebuild-samples (or use any public repo containing a Node.js/Python app — the buildspec below is generic)

## Steps

1. **Add a `buildspec.yml`** to the root of your forked repo (create it via the GitHub web UI if you don't have it locally):
   ```yaml
   version: 0.2
   phases:
     install:
       commands:
         - echo "Installing dependencies..."
     build:
       commands:
         - echo "Running build steps..."
         - echo "Build completed at $(date)"
   artifacts:
     files:
       - '**/*'
   ```
   Commit this to your forked repo's default branch.

2. **Create a CodeBuild service role:**
   ```bash
   aws iam create-role \
     --role-name codebuild-demo-role \
     --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":"codebuild.amazonaws.com"},"Action":"sts:AssumeRole"}]}'
   aws iam attach-role-policy \
     --role-name codebuild-demo-role \
     --policy-arn arn:aws:iam::aws:policy/AWSCodeBuildAdminAccess
   ROLE_ARN=$(aws iam get-role --role-name codebuild-demo-role --query 'Role.Arn' --output text)
   ```
   *(`AWSCodeBuildAdminAccess` is broad — fine for this training account; scope down in production.)*

3. **Create the CodeBuild project**, replacing `<your-github-username>` and `<your-repo-name>`:
   ```bash
   aws codebuild create-project \
     --name cloudnative-demo-build \
     --source type=GITHUB,location=https://github.com/<your-github-username>/<your-repo-name>.git \
     --artifacts type=NO_ARTIFACTS \
     --environment type=LINUX_CONTAINER,image=aws/codebuild/amazonlinux2-x86_64-standard:5.0,computeType=BUILD_GENERAL1_SMALL \
     --service-role $ROLE_ARN \
     --region us-east-1
   ```
   *(If your GitHub repo is private, you'll be prompted to connect a GitHub OAuth token via the console first — public repos work without extra setup.)*

4. **Start a build:**
   ```bash
   aws codebuild start-build --project-name cloudnative-demo-build
   ```
5. **Check build status:**
   ```bash
   aws codebuild batch-get-builds --ids $(aws codebuild list-builds-for-project --project-name cloudnative-demo-build --query 'ids[0]' --output text)
   ```

## Expected Result / Validation
The build status should progress from `IN_PROGRESS` to `SUCCEEDED`. View the logs:
```bash
aws logs tail /aws/codebuild/cloudnative-demo-build --since 10m
```
You should see the echoed output from your `buildspec.yml`, including the build completion timestamp.

## Cleanup
```bash
aws codebuild delete-project --name cloudnative-demo-build
aws iam detach-role-policy --role-name codebuild-demo-role --policy-arn arn:aws:iam::aws:policy/AWSCodeBuildAdminAccess
aws iam delete-role --role-name codebuild-demo-role
```
Verify:
```bash
aws codebuild list-projects --query "projects[?@=='cloudnative-demo-build']"
```
Should return an empty list.

## Troubleshooting
- **Source connection error for GitHub** → For public repos this should work directly; for private repos, connect your GitHub account first via **Developer Tools → Settings → Connections** in the console, then retry.
- **Build fails immediately** → Confirm `buildspec.yml` is valid YAML and sits at the repo root, not in a subfolder.
