# Lab 4.7: Full Cluster Teardown (Run This Last!)

**Module:** 4 — Container Orchestration with Amazon EKS
**Difficulty:** Beginner
**Duration:** ~15 minutes (deletion runs in the background for up to 15 minutes after you kick it off)
**Cost:** ✅ Running this lab **stops all Module 4 charges**. Skipping it will continue to cost ~$0.10–$0.15/hour indefinitely.

## Objective
Completely and verifiably delete the EKS cluster and every associated AWS resource created in Labs 4.1–4.6, so your account incurs no further charges from this module.

## Prerequisites
- Completed some or all of Labs 4.1–4.6
- This should be the **last thing you do** in Module 4, in the same sitting

## Steps

1. **Double-check for any leftover LoadBalancer services** first (these must be deleted before the cluster, or they can be orphaned and keep billing):
   ```bash
   kubectl get services --all-namespaces | grep LoadBalancer
   ```
   If anything appears, delete it:
   ```bash
   kubectl delete service <service-name> --namespace <namespace>
   ```
2. **Delete any remaining namespaces/workloads you created** (optional if you're deleting the whole cluster next, but good practice):
   ```bash
   kubectl get all --all-namespaces
   ```
3. **Delete the entire EKS cluster** (this removes the control plane, node group, Auto Scaling Group, and the dedicated VPC `eksctl` created):
   ```bash
   eksctl delete cluster --name cloudnative-training-cluster --region us-east-1
   ```
   This takes **10–15 minutes**. Do not close your terminal until it completes.
4. **Verify the cluster is gone:**
   ```bash
   aws eks list-clusters --region us-east-1
   ```
5. **Verify no orphaned load balancers remain:**
   ```bash
   aws elbv2 describe-load-balancers --region us-east-1
   ```
6. **Verify no orphaned EC2 instances remain:**
   ```bash
   aws ec2 describe-instances \
     --filters "Name=instance-state-name,Values=running" \
     --query "Reservations[].Instances[].{ID:InstanceId,Type:InstanceType}" \
     --region us-east-1
   ```
7. **Check the Billing Dashboard** in the console (https://us-east-1.console.aws.amazon.com/billing/home) under "Bills" to confirm EKS/EC2/ELB charges stop accruing after today.

## Expected Result / Validation
- `aws eks list-clusters` should return an empty list.
- `aws elbv2 describe-load-balancers` should return an empty list (or only load balancers unrelated to this course).
- `aws ec2 describe-instances` should show no running `t3.small` instances tagged for this cluster.

## Cleanup
This lab **is** the cleanup for all of Module 4. Nothing further to do once all three verification checks above come back clean.

## Troubleshooting
- **`eksctl delete cluster` hangs or fails** → Check the CloudFormation console for stacks named `eksctl-cloudnative-training-cluster-*` — if deletion is stuck, you can manually trigger stack deletion from **CloudFormation → Stacks → Delete**, starting with the nodegroup stack before the cluster stack.
- **VPC deletion fails with "has dependencies"** → Usually means a LoadBalancer service (step 1) wasn't deleted before `eksctl delete cluster` ran. Manually find and delete the orphaned ELB in the EC2 console, then retry `eksctl delete cluster`.
- **Still see charges a day later** → Re-check the EC2 console for any lingering Elastic IPs or NAT Gateways in the `eksctl`-created VPC, and delete them manually if `eksctl delete cluster` didn't fully clean up.
