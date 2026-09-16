# Lab 4.1: Create an EKS Cluster with eksctl

**Module:** 4 — Container Orchestration with Amazon EKS
**Difficulty:** Beginner
**Duration:** ~15 minutes (cluster creation runs in the background for 10–15 min after your active steps)
**Cost:** ⚠️ **NOT Free Tier.** EKS control plane = $0.10/hour. Two `t3.small` worker nodes ≈ $0.0416/hour. **Budget ~$0.15/hour total** while this cluster exists. Plan to complete Labs 4.1–4.6 in one sitting and then immediately run Lab 4.7 to delete everything.

## Objective
Provision a minimal, cost-conscious EKS cluster using `eksctl`, which you'll reuse across all remaining Module 4 labs.

## Prerequisites
- Completed Lab 1.1 (AWS CLI configured, `us-east-1`)
- Install `eksctl`: https://eksctl.io/installation/
- Install `kubectl`: https://kubernetes.io/docs/tasks/tools/
- Verify installs:
  ```bash
  eksctl version
  kubectl version --client
  ```

## Steps

1. **Set a billing safety net first** (strongly recommended): create a CloudWatch billing alarm at a low threshold if you haven't already (see main README).

2. **Create the cluster** with the smallest practical, free-tier-eligible instance type and only 2 nodes:
   ```bash
   eksctl create cluster \
     --name cloudnative-training-cluster \
     --region us-east-1 \
     --nodegroup-name training-nodes \
     --node-type t3.small \
     --nodes 2 \
     --nodes-min 1 \
     --nodes-max 2 \
     --managed
   ```
   This takes **10–15 minutes** — `eksctl` provisions the control plane, VPC, and a managed node group. You'll see progress logged in your terminal.

3. **While it provisions**, note what's happening: `eksctl` is creating a dedicated VPC, subnets, an EKS control plane, an IAM role for nodes, and an Auto Scaling Group of EC2 instances — all normally configured by hand in raw Kubernetes.

4. Once complete, `eksctl` automatically updates your `kubeconfig`. Verify connectivity:
   ```bash
   kubectl get nodes
   ```

## Expected Result / Validation
`kubectl get nodes` should show **2 nodes** in `Ready` status, each a `t3.small` EC2 instance. This confirms your cluster is live and `kubectl` is correctly pointed at it.

## Cleanup
**Do NOT delete the cluster yet** if you plan to continue with Labs 4.2–4.6 — they all reuse this same cluster. Move on to the next lab now.

If you need to stop here and are not continuing today, skip ahead and run **Lab 4.7 (Full Cluster Teardown)** now rather than leaving the cluster running overnight.

## Troubleshooting
- **`eksctl` fails with IAM permission errors** → Confirm you're using the `cloudnative-student` user with `AdministratorAccess` from Lab 1.1.
- **Cluster creation times out** → Re-run the same `eksctl create cluster` command; it's idempotent and will resume from where it left off in most cases. If it fails a second time, run `eksctl delete cluster --name cloudnative-training-cluster --region us-east-1` and retry from scratch.
- **`kubectl get nodes` returns "Unable to connect"** → Run `aws eks update-kubeconfig --name cloudnative-training-cluster --region us-east-1` to refresh your local kubeconfig.
