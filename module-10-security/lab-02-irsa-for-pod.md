# Lab 10.2: IRSA (IAM Roles for Service Accounts) on EKS

**Module:** 10 — Cloud Native Security on AWS
**Difficulty:** Intermediate
**Duration:** ~15 minutes
**Cost:** ⚠️ Requires a running EKS cluster (Module 4 costs apply). Recreate with Module 4, Lab 4.1 if torn down, and tear down again with Module 4, Lab 4.7 immediately after.

## Objective
Configure IAM Roles for Service Accounts (IRSA) so a single Pod can securely call AWS APIs (S3, in this case) using a scoped IAM role — **without** hardcoding any access keys inside the container, which is the cloud native security best practice.

## Prerequisites
- A running EKS cluster (Module 4, Lab 4.1) with `eksctl` and `kubectl` available
- `eksctl utils associate-iam-oidc-provider` must be run (one-time per cluster)

## Steps

1. **Associate an OIDC provider with your cluster** (required for IRSA; safe to re-run if already done):
   ```bash
   eksctl utils associate-iam-oidc-provider \
     --cluster cloudnative-training-cluster --region us-east-1 --approve
   ```
2. **Create an S3 bucket the pod will be allowed to read:**
   ```bash
   BUCKET_NAME="irsa-demo-bucket-$(date +%s)"
   aws s3 mb s3://$BUCKET_NAME --region us-east-1
   echo "IRSA works!" > irsa-test.txt
   aws s3 cp irsa-test.txt s3://$BUCKET_NAME/irsa-test.txt
   ```
3. **Create a Kubernetes service account with an attached IAM role**, scoped ONLY to this bucket, using `eksctl`:
   ```bash
   eksctl create iamserviceaccount \
     --name irsa-demo-sa \
     --namespace default \
     --cluster cloudnative-training-cluster \
     --region us-east-1 \
     --attach-policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess \
     --approve
   ```
   *(Using the AWS managed `AmazonS3ReadOnlyAccess` policy here for simplicity — in production you'd scope this to just the one bucket, as in Lab 10.1.)*
4. **Deploy a Pod that uses this service account** (no AWS credentials in the container at all):
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: irsa-demo-pod
   spec:
     serviceAccountName: irsa-demo-sa
     containers:
       - name: aws-cli
         image: amazon/aws-cli
         command: ["sleep", "3600"]
   ```
   Save as `irsa-pod.yaml` and apply:
   ```bash
   kubectl apply -f irsa-pod.yaml
   ```
5. **Exec into the pod and call S3 — with no access keys configured anywhere:**
   ```bash
   kubectl exec -it irsa-demo-pod -- aws s3 cp s3://$BUCKET_NAME/irsa-test.txt - --region us-east-1
   ```

## Expected Result / Validation
The command in step 5 should print `IRSA works!` to your terminal, even though **no AWS access key or secret was ever placed inside the container**. Run this to prove the identity being used:
```bash
kubectl exec -it irsa-demo-pod -- aws sts get-caller-identity
```
The `Arn` returned should reference the `irsa-demo-sa` role, not your personal IAM user — confirming the pod authenticated via its own scoped IAM role through the OIDC federation, exactly as production EKS workloads should.

## Cleanup
```bash
kubectl delete pod irsa-demo-pod
eksctl delete iamserviceaccount \
  --name irsa-demo-sa --namespace default \
  --cluster cloudnative-training-cluster --region us-east-1
aws s3 rm s3://$BUCKET_NAME --recursive
aws s3 rb s3://$BUCKET_NAME
```
Verify:
```bash
kubectl get pod irsa-demo-pod
```
Should return `NotFound`.

**If you spun up the EKS cluster only for this lab, run Module 4, Lab 4.7 now to fully tear it down.**

## Troubleshooting
- **`AccessDenied` inside the pod** → Confirm `eksctl create iamserviceaccount --approve` completed successfully and that the pod spec references `serviceAccountName: irsa-demo-sa` exactly.
- **`get-caller-identity` shows an unexpected identity** → Ensure no other AWS credentials/env vars are set inside the container image; `amazon/aws-cli` should rely solely on the IRSA-injected web identity token.
