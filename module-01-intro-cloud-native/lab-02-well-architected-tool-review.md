# Lab 1.2: AWS Well-Architected Tool — Workload Review

**Module:** 1 — Introduction to Cloud Native Computing on AWS
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free Tier Eligible (the Well-Architected Tool itself is always free)

## Objective
Use the AWS Well-Architected Tool to define a sample workload and run a lightweight review against the Cloud Native / Reliability best-practice questions, so you understand how AWS frames "cloud native maturity."

## Prerequisites
- Completed Lab 1.1 (CLI + console access)

## Steps

1. Sign in to the AWS Console (as your `cloudnative-student` IAM user) in `us-east-1`.
2. Search for **Well-Architected Tool** and open it.
3. Click **Define workload**.
4. Fill in the workload details:
   - Name: `sample-cloud-native-app`
   - Description: `Practice workload for cloud native training`
   - Review owner: your name
   - Environment: **Pre-production**
   - Region: `US East (N. Virginia)`
5. Click **Next**, then select the **lenses** to apply. Keep the default **AWS Well-Architected Framework** lens checked.
6. Click **Define workload** to save it.
7. Click **Start review**.
8. Work through the **Reliability** pillar questions (choose this pillar since it maps directly to cloud native resilience/scalability concepts from this module). For each question, select the best-practice choices that apply — it's fine to select "None of these" if you're just exploring.
9. Click **Save and exit** once you've answered a handful of questions.
10. From the workload dashboard, view the **Milestone** and **Improvement Plan** — this is the artifact real teams use to track cloud native maturity over time.

## Expected Result / Validation
You should see a workload dashboard showing risk counts (High/Medium/None) per pillar, and an Improvement Plan listing specific AWS recommendations tied to the questions you answered.

## Cleanup
1. Go back to the workload list in the Well-Architected Tool.
2. Select `sample-cloud-native-app`.
3. Click **Actions → Delete workload** to remove the practice workload (optional — it has no cost, but keeps your account tidy).

## Troubleshooting
- **Can't find the pillar questions** → Make sure you clicked "Start review," not just "Define workload."
- **Tool says "no lens selected"** → Go back into the workload's settings and re-add the default framework lens.
