# Lab 4.3: Expose an App via ALB Service

**Module:** 4 — Container Orchestration with Amazon EKS
**Difficulty:** Intermediate
**Duration:** ~15 minutes (ALB provisioning takes a few extra minutes in the background)
**Cost:** ⚠️ **NOT Free Tier.** An Application Load Balancer costs ~$0.0225/hour plus a small per-GB data charge. **Complete the validation step promptly and then run the cleanup for this lab before moving on**, to avoid leaving the ALB running for the rest of the module.

## Objective
Expose your Deployment to the internet using a Kubernetes Service of type `LoadBalancer`, backed by an AWS Application Load Balancer, and understand how cloud native apps get external traffic.

## Prerequisites
- Completed Lab 4.2 concepts (this lab creates its own Deployment fresh)
- Cluster from Lab 4.1 still running

## Steps

1. **Install the AWS Load Balancer Controller add-on** (simplified path via `eksctl`):
   ```bash
   eksctl utils associate-iam-oidc-provider \
     --cluster cloudnative-training-cluster --region us-east-1 --approve
   ```
2. For this beginner lab, we'll use the simpler **Classic/NLB-style LoadBalancer Service** (no separate controller install needed) — Kubernetes' built-in cloud provider integration on EKS provisions an ELB automatically:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: web-app
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: web-app
     template:
       metadata:
         labels:
           app: web-app
       spec:
         containers:
           - name: web-app
             image: public.ecr.aws/nginx/nginx:latest
             ports:
               - containerPort: 80
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: web-app-service
   spec:
     type: LoadBalancer
     selector:
       app: web-app
     ports:
       - port: 80
         targetPort: 80
   ```
   Save as `service.yaml` and apply:
   ```bash
   kubectl apply -f service.yaml
   ```
3. **Wait for the load balancer's external address to be assigned** (1–3 minutes):
   ```bash
   kubectl get service web-app-service --watch
   ```
   Press `Ctrl+C` once you see a value under `EXTERNAL-IP`.
4. **Test the public endpoint:**
   ```bash
   curl http://<EXTERNAL-IP-value>
   ```

## Expected Result / Validation
`curl` should return the default nginx welcome page HTML, proving traffic flowed from the public internet → AWS Load Balancer → into your Kubernetes pods.

## Cleanup
**Delete this immediately after validating** — this is the highest-cost resource in the whole course per hour:
```bash
kubectl delete service web-app-service
kubectl delete deployment web-app
```
Confirm the load balancer is gone (can take a minute to disappear from AWS's side):
```bash
aws elbv2 describe-load-balancers --query "LoadBalancers[?contains(LoadBalancerName, 'a')].LoadBalancerName" --region us-east-1
```
Also double check in the console: **EC2 → Load Balancers** — there should be none left related to this lab.

## Troubleshooting
- **`EXTERNAL-IP` stuck on `<pending>`** → Wait a bit longer (up to 5 minutes); if still pending, run `kubectl describe service web-app-service` to check for IAM/subnet tagging errors (rare with `eksctl`-created clusters).
- **Load balancer not deleted after `kubectl delete service`** → Kubernetes should clean it up automatically within 1–2 minutes; if it persists after 5 minutes, delete it manually from the EC2 console under **Load Balancers**.
