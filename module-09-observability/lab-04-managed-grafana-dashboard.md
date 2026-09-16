# Lab 9.4: Amazon Managed Grafana Dashboard

**Module:** 9 — Observability: Logging, Monitoring & Tracing on AWS
**Difficulty:** Intermediate
**Duration:** ~15 minutes
**Cost:** ⚠️ **NOT Free Tier.** Amazon Managed Grafana has no free tier — it charges per active user per license type (Editor ~$9/user/month, Viewer ~$5/user/month), prorated. Using it for 15 minutes and deleting the workspace immediately after typically costs well under $1, but it is **not zero**. If your budget is $0, you can skip this lab and instead view CloudWatch dashboards natively (free) as a substitute — see the note at the end.

## Objective
Stand up an Amazon Managed Grafana workspace, connect it to CloudWatch as a data source, and build a minimal dashboard visualizing a metric.

## Prerequisites
- Completed Lab 9.2 (or any lab that leaves CloudWatch metrics available — even just default EC2/Lambda metrics from earlier labs work)
- An AWS IAM Identity Center (formerly AWS SSO) instance enabled in your account — Managed Grafana requires it for authentication. If not already enabled, the console will prompt you to enable it (also free) when creating the workspace.

## Steps

1. **Create the Grafana workspace** via console (CLI creation requires several extra IAM setup steps beyond this lab's scope — console is faster for a beginner lab):
   - Go to **Amazon Managed Grafana** in the console.
   - Click **Create workspace**.
   - Name: `cloudnative-demo-grafana`
   - Authentication: choose **AWS IAM Identity Center**.
   - Permission type: **Service managed**.
   - Data sources: check **Amazon CloudWatch**.
   - Click through to **Create workspace**. This takes 2–3 minutes.
2. **Assign yourself as a user:**
   - Once the workspace is `ACTIVE`, click into it → **Assign new user or group**.
   - Select your IAM Identity Center user (or create one if prompted) → assign as **Admin**.
3. **Open the Grafana URL** shown on the workspace details page and sign in.
4. **Add a new dashboard:**
   - Click **Dashboards → New → New Dashboard → Add visualization**.
   - Select the **CloudWatch** data source.
   - Namespace: `AWS/Lambda`, Metric: `Invocations` (or `Errors` if you still have the Lambda from Lab 9.2).
   - Click **Run query** to see the graph populate.
5. **Save the dashboard** with a name like `Cloud Native Training Dashboard`.

## Expected Result / Validation
You should see a live line graph of your Lambda's invocation/error counts rendered inside Grafana, sourced directly from CloudWatch — demonstrating Grafana's role as a richer visualization layer on top of AWS-native metrics.

## Cleanup
**This step is important given the per-user billing — delete the workspace immediately after this lab:**
- Go to **Amazon Managed Grafana → Workspaces**.
- Select `cloudnative-demo-grafana` → **Delete**.
- Confirm deletion.

Verify:
```bash
aws grafana list-workspaces --query "workspaces[?name=='cloudnative-demo-grafana']"
```
Should return an empty list.

**Free alternative:** if you'd rather not incur any cost, skip this lab and instead explore **CloudWatch → Dashboards → Create dashboard** in the console directly — CloudWatch's native dashboards (up to 3 free/month) provide similar graphing capability at no cost, without Grafana's per-user licensing.

## Troubleshooting
- **IAM Identity Center not enabled** → The console will offer to enable it during workspace creation; this is a free, one-time account-level setting.
- **No data in the graph** → Confirm the metric namespace/name matches a Lambda function you've actually invoked recently (metrics only exist once at least one invocation has occurred).
