# Lab 1.1: AWS Account & CLI Setup

**Module:** 1 — Introduction to Cloud Native Computing on AWS
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free Tier Eligible (no billable resources created)

## Objective
Set up everything you need to follow the rest of this course: a working AWS account, an IAM user (instead of using the root account), and the AWS CLI configured on your machine, pointed at `us-east-1`.

## Prerequisites
- A computer with internet access and permission to install software
- An email address and a payment card (required by AWS even for free-tier usage — you will not be charged if you follow the cleanup steps throughout this course)

## Steps

1. **Create an AWS account** (skip if you already have one)
   - Go to https://aws.amazon.com/free/ and click **Create a Free Account**.
   - Follow the prompts (email, password, account name, payment card, phone verification).
   - Choose the **Basic support plan (Free)**.

2. **Sign in to the AWS Console**
   - Go to https://console.aws.amazon.com/ and sign in as the **root user** (only for this one-time setup step).

3. **Set your default region to `us-east-1`**
   - In the top-right corner of the console, click the region dropdown and select **US East (N. Virginia) us-east-1**.
   - All labs in this course assume this region.

4. **Create an IAM user for daily work (best practice — never use root day-to-day)**
   - Search for **IAM** in the console search bar and open it.
   - Go to **Users → Create user**.
   - User name: `cloudnative-student`
   - Check **Provide user access to the AWS Management Console** (optional, for console labs).
   - Click **Next** → **Attach policies directly** → attach `AdministratorAccess` (fine for a personal training account; never do this in a production/work account).
   - Click **Next → Create user**.

5. **Create an access key for CLI use**
   - Open the new user → **Security credentials** tab → **Create access key**.
   - Select **Command Line Interface (CLI)** → acknowledge the warning → **Next → Create access key**.
   - **Copy the Access Key ID and Secret Access Key now** — the secret is shown only once.

6. **Install the AWS CLI v2** on your machine
   - Windows/Mac/Linux instructions: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
   - Verify install:
     ```bash
     aws --version
     ```

7. **Configure the CLI**
   ```bash
   aws configure
   ```
   Enter:
   - AWS Access Key ID: *(paste from step 5)*
   - AWS Secret Access Key: *(paste from step 5)*
   - Default region name: `us-east-1`
   - Default output format: `json`

## Expected Result / Validation
Run:
```bash
aws sts get-caller-identity
```
You should see JSON output showing your `Account`, `UserId`, and `Arn` (ending in `user/cloudnative-student`). This confirms the CLI is authenticated correctly.

## Cleanup
No billable resources were created in this lab — nothing to clean up. Keep the IAM user and CLI configuration; you'll use them throughout the course.

**Security tip:** Never commit your access keys to GitHub or share them. Add `.aws/` to your global gitignore if working in a repo folder.

## Troubleshooting
- **`aws: command not found`** → Restart your terminal after installation, or check your PATH.
- **`Unable to locate credentials`** → Re-run `aws configure` and confirm no extra spaces were pasted into the keys.
- **Region mismatch errors in later labs** → Re-run `aws configure set region us-east-1`.
