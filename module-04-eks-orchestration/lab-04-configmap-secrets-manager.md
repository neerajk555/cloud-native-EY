# Lab 4.4: ConfigMaps & Secrets Manager Integration

**Module:** 4 — Container Orchestration with Amazon EKS
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ⚠️ Uses the cluster from Lab 4.1 (no new billable resources; Secrets Manager first 30 days free per secret, then ~$0.40/month per secret — deleted at end of this lab).

## Objective
Separate configuration (ConfigMap) from sensitive data (a Secret sourced from AWS Secrets Manager) and inject both into a Pod as environment variables — a core 12-Factor App practice.

## Prerequisites
- Completed Lab 4.1 (cluster running)

## Steps

1. **Create a ConfigMap** for non-sensitive app settings:
   ```bash
   kubectl create configmap app-config \
     --from-literal=APP_ENV=training \
     --from-literal=LOG_LEVEL=debug
   ```
2. **Create a secret in AWS Secrets Manager** (simulating a database password):
   ```bash
   aws secretsmanager create-secret \
     --name training/db-password \
     --secret-string '{"password":"SuperSecretTraining123!"}' \
     --region us-east-1
   ```
3. **For simplicity in this beginner lab, mirror the secret into a native Kubernetes Secret** (in production you'd typically use the "AWS Secrets and Configuration Provider" CSI driver — mentioned here as the production-grade approach, but that requires extra add-ons beyond this 15-minute scope):
   ```bash
   SECRET_VALUE=$(aws secretsmanager get-secret-value --secret-id training/db-password --query SecretString --output text | python3 -c "import sys, json; print(json.load(sys.stdin)['password'])")
   kubectl create secret generic db-secret --from-literal=DB_PASSWORD=$SECRET_VALUE
   ```
4. **Create a Pod that consumes both:**
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: config-demo-pod
   spec:
     containers:
       - name: demo
         image: public.ecr.aws/nginx/nginx:latest
         envFrom:
           - configMapRef:
               name: app-config
         env:
           - name: DB_PASSWORD
             valueFrom:
               secretKeyRef:
                 name: db-secret
                 key: DB_PASSWORD
   ```
   Save as `config-pod.yaml` and apply:
   ```bash
   kubectl apply -f config-pod.yaml
   ```
5. **Verify the environment variables landed inside the container:**
   ```bash
   kubectl exec config-demo-pod -- printenv | grep -E "APP_ENV|LOG_LEVEL|DB_PASSWORD"
   ```

## Expected Result / Validation
The `printenv` output should show all three variables (`APP_ENV=training`, `LOG_LEVEL=debug`, `DB_PASSWORD=SuperSecretTraining123!`), confirming both the ConfigMap and the Secrets-Manager-sourced secret were successfully injected.

## Cleanup
```bash
kubectl delete pod config-demo-pod
kubectl delete configmap app-config
kubectl delete secret db-secret
aws secretsmanager delete-secret --secret-id training/db-password --force-delete-without-recovery --region us-east-1
```
Verify the Secrets Manager secret is gone:
```bash
aws secretsmanager list-secrets --query "SecretList[?Name=='training/db-password']"
```
Should return an empty list.

## Troubleshooting
- **`python3` not found in step 3** → Install Python 3, or manually copy the password value from `aws secretsmanager get-secret-value --secret-id training/db-password` output instead of scripting the extraction.
- **`printenv` shows nothing** → Confirm the pod is `Running`: `kubectl get pod config-demo-pod`.
