# Lab 4.2: Deploy a Pod & Deployment

**Module:** 4 — Container Orchestration with Amazon EKS
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ⚠️ Uses the cluster created in Lab 4.1 — no additional AWS resources billed beyond the running cluster/nodes.

## Objective
Deploy the container image you pushed to ECR in Module 3 as a Kubernetes Pod, then as a self-healing Deployment, and observe Kubernetes' core reconciliation loop in action.

## Prerequisites
- Completed Lab 4.1 (cluster `cloudnative-training-cluster` running, `kubectl` connected)
- The ECR image from Module 3, Lab 3.4 (`hello-cloudnative-app:1.0.0`). If you deleted that repository, quickly redo Module 3 Labs 3.3–3.4 first, or substitute the public image `public.ecr.aws/nginx/nginx:latest` in the YAML below.

## Steps

1. **Create a standalone Pod manifest** — `pod.yaml`:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: hello-app-pod
   spec:
     containers:
       - name: hello-app
         image: public.ecr.aws/nginx/nginx:latest
         ports:
           - containerPort: 80
   ```
   *(Using the public nginx image here keeps this lab runnable even if you didn't keep your Module 3 ECR repo. Swap in your own ECR image URI if you still have it.)*

2. **Apply it:**
   ```bash
   kubectl apply -f pod.yaml
   ```
3. **Check its status:**
   ```bash
   kubectl get pods
   ```
4. **Delete the pod manually** to observe that a standalone Pod does NOT self-heal:
   ```bash
   kubectl delete pod hello-app-pod
   kubectl get pods
   ```
   Notice: it's gone for good — nothing recreates it.

5. **Now create a Deployment instead** — `deployment.yaml`:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: hello-app-deployment
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: hello-app
     template:
       metadata:
         labels:
           app: hello-app
       spec:
         containers:
           - name: hello-app
             image: public.ecr.aws/nginx/nginx:latest
             ports:
               - containerPort: 80
   ```
6. **Apply it:**
   ```bash
   kubectl apply -f deployment.yaml
   kubectl get deployments
   kubectl get pods -l app=hello-app
   ```
7. **Simulate a failure** — delete one of the pods:
   ```bash
   kubectl delete pod $(kubectl get pods -l app=hello-app -o jsonpath='{.items[0].metadata.name}')
   kubectl get pods -l app=hello-app
   ```

## Expected Result / Validation
After step 7, you should immediately see a **new Pod being created** to replace the deleted one, keeping the replica count at 2. This is the ReplicaSet controller's reconciliation loop — the core self-healing behavior that distinguishes a Deployment from a standalone Pod.

## Cleanup
Remove just this lab's resources (keep the cluster running for the next lab):
```bash
kubectl delete deployment hello-app-deployment
kubectl delete pod hello-app-pod --ignore-not-found
```
Verify:
```bash
kubectl get pods
```
Should return `No resources found`.

## Troubleshooting
- **Pod stuck in `ImagePullBackOff`** → Confirm the image URI is correct and, if using your own ECR image, that your node IAM role has ECR pull permissions (the default `eksctl`-created node role includes this).
- **Pod stuck in `Pending`** → Run `kubectl describe pod <pod-name>` to see scheduling errors — usually insufficient node capacity; the 2-node `t3.small` cluster from Lab 4.1 can comfortably run a few small pods.
