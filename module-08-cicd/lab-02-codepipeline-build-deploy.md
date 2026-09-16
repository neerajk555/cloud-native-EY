# Lab 8.2: CodePipeline — Build → Deploy

**Module:** 8 — CI/CD for Cloud Native Applications on AWS
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free Tier Eligible (1 free active pipeline/month for the first year; S3 artifact storage falls within the 5GB free tier)

## Objective
Orchestrate the CodeBuild project from Lab 8.1 into a full CodePipeline, triggered automatically by GitHub commits, and observe an end-to-end pipeline execution.

## Prerequisites
- Completed Lab 8.1 (CodeBuild project `cloudnative-demo-build` still exists — recreate it first if you already cleaned it up)
- An S3 bucket for pipeline artifacts (created in step 1 below)

## Steps

1. **Create an S3 bucket for pipeline artifacts** (bucket names must be globally unique — add a random suffix):
   ```bash
   BUCKET_NAME="cloudnative-pipeline-artifacts-$(date +%s)"
   aws s3 mb s3://$BUCKET_NAME --region us-east-1
   ```
2. **Create a CodePipeline service role:**
   ```bash
   aws iam create-role \
     --role-name codepipeline-demo-role \
     --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":"codepipeline.amazonaws.com"},"Action":"sts:AssumeRole"}]}'
   aws iam attach-role-policy \
     --role-name codepipeline-demo-role \
     --policy-arn arn:aws:iam::aws:policy/AWSCodePipeline_FullAccess
   aws iam attach-role-policy \
     --role-name codepipeline-demo-role \
     --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
   aws iam attach-role-policy \
     --role-name codepipeline-demo-role \
     --policy-arn arn:aws:iam::aws:policy/AWSCodeBuildAdminAccess
   PIPELINE_ROLE_ARN=$(aws iam get-role --role-name codepipeline-demo-role --query 'Role.Arn' --output text)
   ```
3. **Create a pipeline definition** — `pipeline.json` (replace `<your-github-username>`, `<your-repo-name>`, and note this uses the simpler GitHub-via-CodeStar-connection approach for beginners is skipped here in favor of the classic S3-polling-free "GitHub (Version 2)" source action, which needs a one-time console connection):
   - **Simplify for this beginner lab**: instead of wiring a full GitHub webhook (which requires a console-based CodeStar Connection), trigger the pipeline manually to focus on the Build → Deploy orchestration concept.
   ```json
   {
     "pipeline": {
       "name": "cloudnative-demo-pipeline",
       "roleArn": "PIPELINE_ROLE_ARN_PLACEHOLDER",
       "artifactStore": { "type": "S3", "location": "BUCKET_NAME_PLACEHOLDER" },
       "stages": [
         {
           "name": "Source",
           "actions": [{
             "name": "SourceAction",
             "actionTypeId": {"category": "Source", "owner": "AWS", "provider": "S3", "version": "1"},
             "outputArtifacts": [{"name": "SourceOutput"}],
             "configuration": {"S3Bucket": "BUCKET_NAME_PLACEHOLDER", "S3ObjectKey": "source.zip"}
           }]
         },
         {
           "name": "Build",
           "actions": [{
             "name": "BuildAction",
             "actionTypeId": {"category": "Build", "owner": "AWS", "provider": "CodeBuild", "version": "1"},
             "inputArtifacts": [{"name": "SourceOutput"}],
             "outputArtifacts": [{"name": "BuildOutput"}],
             "configuration": {"ProjectName": "cloudnative-demo-build"}
           }]
         }
       ]
     }
   }
   ```
   Replace the placeholders using `sed` (or manually edit the file):
   ```bash
   sed -i "s|PIPELINE_ROLE_ARN_PLACEHOLDER|$PIPELINE_ROLE_ARN|g; s|BUCKET_NAME_PLACEHOLDER|$BUCKET_NAME|g" pipeline.json
   ```
4. **Upload a trivial "source" zip** to S3 to act as the pipeline's input artifact:
   ```bash
   mkdir source-demo && cd source-demo
   echo "console.log('hello from pipeline source');" > app.js
   zip ../source.zip app.js
   cd ..
   aws s3 cp source.zip s3://$BUCKET_NAME/source.zip
   ```
5. **Create the pipeline:**
   ```bash
   aws codepipeline create-pipeline --cli-input-json file://pipeline.json
   ```
6. **Watch the pipeline execute** (it auto-starts on creation):
   ```bash
   aws codepipeline get-pipeline-state --name cloudnative-demo-pipeline
   ```

## Expected Result / Validation
Re-run `get-pipeline-state` after ~1–2 minutes. Both the `Source` and `Build` stage statuses should show `"status": "Succeeded"`. This confirms the pipeline pulled your zipped source from S3 and successfully ran it through the CodeBuild project from Lab 8.1.

## Cleanup
```bash
aws codepipeline delete-pipeline --name cloudnative-demo-pipeline
aws s3 rm s3://$BUCKET_NAME --recursive
aws s3 rb s3://$BUCKET_NAME
aws iam detach-role-policy --role-name codepipeline-demo-role --policy-arn arn:aws:iam::aws:policy/AWSCodePipeline_FullAccess
aws iam detach-role-policy --role-name codepipeline-demo-role --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
aws iam detach-role-policy --role-name codepipeline-demo-role --policy-arn arn:aws:iam::aws:policy/AWSCodeBuildAdminAccess
aws iam delete-role --role-name codepipeline-demo-role
```
Also clean up Lab 8.1's CodeBuild project/role if you no longer need them (see that lab's Cleanup section).

Verify:
```bash
aws codepipeline list-pipelines --query "pipelines[?name=='cloudnative-demo-pipeline']"
```
Should return an empty list.

## Troubleshooting
- **Pipeline stuck on Source stage** → Confirm the S3 object key matches exactly (`source.zip`) and the bucket is in `us-east-1`.
- **Build stage fails** → Check that `cloudnative-demo-build` from Lab 8.1 still exists: `aws codebuild list-projects`.
- **Real-world note:** production pipelines typically use a **CodeStar Connection** to GitHub for automatic webhook-triggered builds on every push — this lab used S3-as-source to keep the 15-minute, no-extra-console-setup constraint; feel free to explore the GitHub-connected version afterward on your own time.
