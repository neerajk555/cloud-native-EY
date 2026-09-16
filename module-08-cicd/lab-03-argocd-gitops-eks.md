# Lab 8.3: GitOps Basics with Argo CD on EKS

**Module:** 8 — CI/CD for Cloud Native Applications on AWS
**Difficulty:** Intermediate
**Duration:** ~15 minutes
**Cost:** ⚠️ Requires a running EKS cluster (Module 4). If torn down, recreate with Module 4, Lab 4.1 first, and tear down again with Module 4, Lab 4.7 immediately after this lab.

## Objective
Install Argo CD on your EKS cluster and deploy an application declaratively from a Git repository, understanding the GitOps principle: Git as the single source of truth for what's running in the cluster.

## Prerequisites
- A running EKS cluster (Module 4, Lab 4.1) with `kubectl` connected

## Steps

1. **Create a namespace for Argo CD and install it:**
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```
   This takes 1–2 minutes to pull images and start pods.
2. **Wait for the Argo CD server pod to be ready:**
   ```bash
   kubectl wait --for=condition=available --timeout=120s deployment/argocd-server -n argocd
   ```
3. **Get the initial admin password:**
   ```bash
   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
   ```
   Copy this password.
4. **Port-forward to access the Argo CD UI** (avoids provisioning a billable load balancer):
   ```bash
   kubectl port-forward svc/argocd-server -n argocd 8080:443
   ```
   Leave this running; open a second terminal for the next steps.
5. **In your browser**, go to `https://localhost:8080` (accept the self-signed certificate warning). Log in with username `admin` and the password from step 3.
6. **Create an Argo CD Application** pointing at a public sample repo (Argo CD's own guestbook example):
   ```bash
   kubectl apply -f - << 'YAML'
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: guestbook-demo
     namespace: argocd
   spec:
     project: default
     source:
       repoURL: https://github.com/argoproj/argocd-example-apps.git
       targetRevision: HEAD
       path: guestbook
     destination:
       server: https://kubernetes.default.svc
       namespace: default
     syncPolicy:
       automated:
         prune: true
         selfHeal: true
   YAML
   ```

## Expected Result / Validation
1. In the Argo CD UI, the `guestbook-demo` application should appear and, within a minute, show status **Synced** and **Healthy**.
2. Confirm from the CLI as well:
   ```bash
   kubectl get pods -n default -l app.kubernetes.io/instance=guestbook-demo
   ```
   You should see `guestbook-ui` pods running — deployed with zero manual `kubectl apply` commands by you, purely by Argo CD reading the Git repo's manifests.
3. **See self-healing in action:** manually scale down the guestbook deployment, and watch Argo CD revert it:
   ```bash
   kubectl scale deployment guestbook-ui --replicas=0 -n default
   ```
   Within ~1 minute (Argo CD's default sync interval), it should scale back up to match the Git-defined desired state.

## Cleanup
```bash
kubectl delete application guestbook-demo -n argocd
kubectl delete namespace argocd
```
Stop the `kubectl port-forward` command (Ctrl+C in that terminal).

Verify:
```bash
kubectl get namespace argocd
```
Should show the namespace terminating or already gone.

**If you spun up the EKS cluster just for this lab, now run Module 4, Lab 4.7 to fully tear it down and stop all charges.**

## Troubleshooting
- **Argo CD pods stuck in `Pending`** → Your 2-node `t3.small` cluster from Module 4 should have enough capacity, but check with `kubectl describe pod <pod-name> -n argocd` if issues arise.
- **Browser shows "connection refused" on localhost:8080** → Confirm the `port-forward` command from step 4 is still running in its terminal window.
- **Self-heal doesn't trigger immediately** → Default Argo CD reconciliation runs every 3 minutes by default; give it a little time, or trigger a manual sync from the UI ("Sync" button).
