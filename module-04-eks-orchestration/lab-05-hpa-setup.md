# Lab 4.5: Horizontal Pod Autoscaler (HPA)

**Module:** 4 — Container Orchestration with Amazon EKS
**Difficulty:** Intermediate
**Duration:** ~15 minutes
**Cost:** ⚠️ Uses the cluster from Lab 4.1 — no additional billable resources.

## Objective
Install the Kubernetes Metrics Server, configure an HPA that scales a Deployment based on CPU utilization, and generate synthetic load to trigger a scale-up.

## Prerequisites
- Completed Lab 4.1 (cluster running)

## Steps

1. **Install the Metrics Server** (required for HPA to read CPU metrics):
   ```bash
   kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
   ```
2. **Wait ~30 seconds, then verify it's running:**
   ```bash
   kubectl get deployment metrics-server -n kube-system
   ```
3. **Create a Deployment with CPU requests/limits set** (required for HPA to calculate utilization) — `hpa-deployment.yaml`:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: cpu-demo
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: cpu-demo
     template:
       metadata:
         labels:
           app: cpu-demo
       spec:
         containers:
           - name: cpu-demo
             image: public.ecr.aws/nginx/nginx:latest
             resources:
               requests:
                 cpu: "100m"
               limits:
                 cpu: "200m"
   ```
   Apply it:
   ```bash
   kubectl apply -f hpa-deployment.yaml
   ```
4. **Create an HPA** targeting 50% CPU utilization, scaling between 1 and 4 replicas:
   ```bash
   kubectl autoscale deployment cpu-demo --cpu-percent=50 --min=1 --max=4
   ```
5. **Watch the HPA** (leave this running in one terminal):
   ```bash
   kubectl get hpa cpu-demo --watch
   ```
6. **In a second terminal, generate CPU load** using a temporary busybox pod hammering the nginx pod with requests:
   ```bash
   kubectl run load-generator --image=busybox --restart=Never -- /bin/sh -c \
     "while true; do wget -q -O- http://cpu-demo; done"
   ```

## Expected Result / Validation
After 1–3 minutes of sustained load, the `kubectl get hpa` watch output should show the `TARGETS` column CPU percentage rising above 50%, followed by the `REPLICAS` count increasing beyond 1 (up to a max of 4) — confirming the HPA scaled the deployment automatically.

## Cleanup
```bash
kubectl delete pod load-generator
kubectl delete hpa cpu-demo
kubectl delete deployment cpu-demo
kubectl delete deployment metrics-server -n kube-system
kubectl delete -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
Verify:
```bash
kubectl get hpa
kubectl get deployment cpu-demo
```
Both should return `No resources found`.

## Troubleshooting
- **HPA shows `<unknown>` for TARGETS** → Metrics Server needs 1–2 minutes after install to start reporting; also confirm `kubectl top pods` returns data.
- **On EKS, Metrics Server may need `--kubelet-insecure-tls`** → If `kubectl top nodes` errors out, edit the metrics-server deployment args to add `--kubelet-insecure-tls` (common lab workaround, not recommended for production).
