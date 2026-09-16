# Lab 9.1: Enable Container Insights on EKS

**Module:** 9 — Observability: Logging, Monitoring & Tracing on AWS
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ⚠️ Requires a running EKS cluster (Module 4 costs apply). CloudWatch Container Insights itself has a small per-GB ingestion/storage cost beyond the CloudWatch Logs free tier (5GB free/month) — for a 15-minute lab this is a few cents at most.

## Objective
Enable Container Insights on your EKS cluster to collect cluster/pod/node-level metrics and logs into CloudWatch, and view them in the console.

## Prerequisites
- A running EKS cluster (Module 4, Lab 4.1). Recreate it if torn down.

## Steps

1. **Install the CloudWatch Observability EKS add-on** (the modern, simplified way to enable Container Insights):
   ```bash
   aws eks create-addon \
     --cluster-name cloudnative-training-cluster \
     --addon-name amazon-cloudwatch-observability \
     --region us-east-1
   ```
2. **Wait for the add-on to become active** (1–3 minutes):
   ```bash
   aws eks describe-addon \
     --cluster-name cloudnative-training-cluster \
     --addon-name amazon-cloudwatch-observability \
     --region us-east-1 \
     --query 'addon.status'
   ```
   Wait until this returns `"ACTIVE"`.
3. **Verify the CloudWatch agent pods are running** in your cluster:
   ```bash
   kubectl get pods -n amazon-cloudwatch
   ```
4. **Generate some activity** so there's data to look at — deploy a simple workload:
   ```bash
   kubectl create deployment insights-demo --image=public.ecr.aws/nginx/nginx:latest --replicas=2
   ```
5. **View metrics in the console:**
   - Go to **CloudWatch → Insights → Container Insights** in the console.
   - Select **EKS Clusters** view, choose `cloudnative-training-cluster`.

## Expected Result / Validation
Within 3–5 minutes, the Container Insights dashboard should show CPU/memory utilization graphs for your cluster's nodes and the `insights-demo` pods, plus a resource map view of your workloads. This confirms metrics are flowing from the cluster into CloudWatch.

## Cleanup
```bash
kubectl delete deployment insights-demo
aws eks delete-addon \
  --cluster-name cloudnative-training-cluster \
  --addon-name amazon-cloudwatch-observability \
  --region us-east-1
```
Also delete the CloudWatch log groups created by the add-on to stop any further storage charges:
```bash
aws logs delete-log-group --log-group-name /aws/containerinsights/cloudnative-training-cluster/application --region us-east-1
aws logs delete-log-group --log-group-name /aws/containerinsights/cloudnative-training-cluster/performance --region us-east-1
aws logs delete-log-group --log-group-name /aws/containerinsights/cloudnative-training-cluster/host --region us-east-1
```
*(If a log group name errors as "not found," that's fine — it means the add-on didn't create that particular group.)*

Verify:
```bash
aws eks describe-addon --cluster-name cloudnative-training-cluster --addon-name amazon-cloudwatch-observability --region us-east-1
```
Should return a `ResourceNotFoundException`.

**If you spun up the EKS cluster only for this lab, run Module 4, Lab 4.7 next to tear it down fully.**

## Troubleshooting
- **Add-on stuck in `CREATING`** → Wait a few extra minutes; EKS add-ons can take 3–5 minutes on first install.
- **No data in the console yet** → Metrics ingestion has a short delay; wait 5 minutes and refresh, and confirm `insights-demo` pods are `Running`.
