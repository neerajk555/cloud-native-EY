# Lab 5.1: Sidecar Pattern on EKS

**Module:** 5 — Cloud Native Design Patterns
**Difficulty:** Intermediate
**Duration:** ~15 minutes
**Cost:** ⚠️ Requires an EKS cluster. If you tore down your Module 4 cluster, quickly recreate it with Module 4 Lab 4.1 first (~$0.10–0.15/hr while running), and tear it down again with Module 4 Lab 4.7 immediately after this lab.

## Objective
Implement the **Sidecar pattern**: run a helper container alongside your main application container in the same Pod, sharing the same network and storage — a foundational cloud native pattern used for logging, proxies, and service mesh.

## Prerequisites
- A running EKS cluster (Module 4, Lab 4.1) with `kubectl` connected

## Steps

1. **Create a Pod with two containers** sharing a volume — the main app writes logs, and a sidecar tails/ships them:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: sidecar-demo
   spec:
     volumes:
       - name: shared-logs
         emptyDir: {}
     containers:
       - name: main-app
         image: busybox
         command: ["/bin/sh", "-c"]
         args:
           - >
             while true; do
               echo "$(date) - app is running" >> /var/log/app.log;
               sleep 5;
             done
         volumeMounts:
           - name: shared-logs
             mountPath: /var/log
       - name: log-sidecar
         image: busybox
         command: ["/bin/sh", "-c", "tail -F /var/log/app.log"]
         volumeMounts:
           - name: shared-logs
             mountPath: /var/log
   ```
   Save as `sidecar-pod.yaml` and apply:
   ```bash
   kubectl apply -f sidecar-pod.yaml
   ```
2. **Verify both containers are running in the same Pod:**
   ```bash
   kubectl get pod sidecar-demo
   kubectl get pod sidecar-demo -o jsonpath='{.spec.containers[*].name}'
   ```
3. **View the sidecar's live-tailed logs** (which are actually written by the main-app container):
   ```bash
   kubectl logs sidecar-demo -c log-sidecar --follow
   ```
   Press `Ctrl+C` after a few lines appear.

## Expected Result / Validation
The `log-sidecar` container's logs should show lines like `<date> - app is running`, even though those lines were written by the **separate** `main-app` container. This proves the two containers share the same Pod network namespace and the `shared-logs` volume — the essence of the sidecar pattern (used in production for tools like Fluent Bit log shippers or Envoy proxies).

## Cleanup
```bash
kubectl delete pod sidecar-demo
```
Verify:
```bash
kubectl get pods
```
`sidecar-demo` should not appear.

If this was your only reason for having the cluster running, proceed to Module 4, Lab 4.7 to tear the cluster down.

## Troubleshooting
- **`log-sidecar` shows no output** → Give the `main-app` container 10–15 seconds to write its first log line before tailing.
- **Pod stuck in `ContainerCreating`** → Run `kubectl describe pod sidecar-demo` to check for image pull issues (busybox is a small public image and should pull quickly).
