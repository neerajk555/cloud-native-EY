# Lab 4.6: Install a Helm Chart

**Module:** 4 — Container Orchestration with Amazon EKS
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ⚠️ Uses the cluster from Lab 4.1 — no additional billable resources.

## Objective
Install Helm, add a public chart repository, and deploy a real application (`nginx`) using a Helm chart instead of raw YAML — understanding Kubernetes' package manager.

## Prerequisites
- Completed Lab 4.1 (cluster running)
- Install Helm: https://helm.sh/docs/intro/install/
- Verify:
  ```bash
  helm version
  ```

## Steps

1. **Add the Bitnami chart repository** (a large, well-maintained public chart collection):
   ```bash
   helm repo add bitnami https://charts.bitnami.com/bitnami
   helm repo update
   ```
2. **Search for the nginx chart:**
   ```bash
   helm search repo bitnami/nginx
   ```
3. **Install it** into its own namespace, with a release name:
   ```bash
   kubectl create namespace helm-demo
   helm install my-nginx bitnami/nginx --namespace helm-demo \
     --set service.type=ClusterIP
   ```
   *(Using `ClusterIP` instead of `LoadBalancer` here deliberately avoids provisioning another billable ALB/ELB in this lab.)*
4. **Check the release status:**
   ```bash
   helm list --namespace helm-demo
   kubectl get pods --namespace helm-demo
   ```
5. **Port-forward to test locally** (avoids needing a load balancer):
   ```bash
   kubectl port-forward --namespace helm-demo svc/my-nginx 8080:80
   ```
6. **In a second terminal, test it:**
   ```bash
   curl http://localhost:8080
   ```
   Press `Ctrl+C` in the first terminal to stop port-forwarding once done.

## Expected Result / Validation
`curl` should return the nginx welcome page HTML, and `helm list` should show `my-nginx` with `STATUS: deployed`. This demonstrates deploying a production-grade chart (with built-in best practices for probes, resource limits, etc.) with a single command, versus hand-writing all that YAML yourself.

## Cleanup
```bash
helm uninstall my-nginx --namespace helm-demo
kubectl delete namespace helm-demo
helm repo remove bitnami
```
Verify:
```bash
helm list --all-namespaces
kubectl get namespace helm-demo
```
The nginx release should be gone and the namespace deletion should be in progress or complete.

## Troubleshooting
- **`helm install` errors about namespace not found** → Confirm step 3's `kubectl create namespace helm-demo` ran successfully first.
- **Port-forward connection refused** → Wait for the pod to reach `Running`/`Ready` status first: `kubectl get pods --namespace helm-demo --watch`.
