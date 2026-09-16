# Lab 7.2: Configure a Virtual Service & Router (Traffic Splitting)

**Module:** 7 — Service Mesh & Inter-Service Communication
**Difficulty:** Intermediate
**Duration:** ~15 minutes
**Cost:** ⚠️ Mesh configuration objects are free. This lab builds purely on the logical mesh from Lab 7.1 — no EKS cluster is required to complete these specific CLI steps, since we're only defining routing rules, not deploying live mesh-enabled pods.

## Objective
Add a **virtual router** with a weighted routing rule that splits traffic 80/20 between two versions of the "orders" service — a canary deployment pattern enabled by service mesh traffic management.

## Prerequisites
- Completed Lab 7.1 (mesh `cloudnative-demo-mesh` with virtual node `orders-service-vn` still existing)

## Steps

1. **Create a second virtual node**, representing a "v2" canary version of the orders service:
   ```bash
   aws appmesh create-virtual-node \
     --mesh-name cloudnative-demo-mesh \
     --virtual-node-name orders-service-v2-vn \
     --spec '{
       "listeners": [{"portMapping": {"port": 8080, "protocol": "http"}}],
       "serviceDiscovery": {"dns": {"hostname": "orders-v2.svc.cluster.local"}}
     }'
   ```
2. **Create a virtual router** with a route that splits traffic:
   ```bash
   aws appmesh create-virtual-router \
     --mesh-name cloudnative-demo-mesh \
     --virtual-router-name orders-router \
     --spec '{
       "listeners": [{"portMapping": {"port": 8080, "protocol": "http"}}]
     }'
   ```
3. **Create a weighted route** — 80% to v1, 20% to the v2 canary:
   ```bash
   aws appmesh create-route \
     --mesh-name cloudnative-demo-mesh \
     --virtual-router-name orders-router \
     --route-name orders-canary-route \
     --spec '{
       "httpRoute": {
         "match": {"prefix": "/"},
         "action": {
           "weightedTargets": [
             {"virtualNode": "orders-service-vn", "weight": 80},
             {"virtualNode": "orders-service-v2-vn", "weight": 20}
           ]
         }
       }
     }'
   ```
4. **Update the virtual service from Lab 7.1** to route through this router instead of directly to a single virtual node:
   ```bash
   aws appmesh update-virtual-service \
     --mesh-name cloudnative-demo-mesh \
     --virtual-service-name orders.svc.cluster.local \
     --spec '{
       "provider": {"virtualRouter": {"virtualRouterName": "orders-router"}}
     }'
   ```

## Expected Result / Validation
```bash
aws appmesh describe-route \
  --mesh-name cloudnative-demo-mesh \
  --virtual-router-name orders-router \
  --route-name orders-canary-route
```
The output should show `weightedTargets` with `orders-service-vn` at weight `80` and `orders-service-v2-vn` at weight `20`. In a real EKS deployment with the App Mesh sidecar injected, this configuration would cause the mesh's Envoy proxies to route roughly 20% of live traffic to the v2 pods automatically — no application code changes required, which is the key value proposition of a service mesh for canary releases.

## Cleanup
```bash
aws appmesh update-virtual-service \
  --mesh-name cloudnative-demo-mesh \
  --virtual-service-name orders.svc.cluster.local \
  --spec '{"provider": {"virtualNode": {"virtualNodeName": "orders-service-vn"}}}'
aws appmesh delete-route --mesh-name cloudnative-demo-mesh --virtual-router-name orders-router --route-name orders-canary-route
aws appmesh delete-virtual-router --mesh-name cloudnative-demo-mesh --virtual-router-name orders-router
aws appmesh delete-virtual-service --mesh-name cloudnative-demo-mesh --virtual-service-name orders.svc.cluster.local
aws appmesh delete-virtual-node --mesh-name cloudnative-demo-mesh --virtual-node-name orders-service-vn
aws appmesh delete-virtual-node --mesh-name cloudnative-demo-mesh --virtual-node-name orders-service-v2-vn
aws appmesh delete-mesh --mesh-name cloudnative-demo-mesh
```
Verify:
```bash
aws appmesh list-meshes
```
Should return an empty list.

If you spun up an EKS cluster for this module, run Module 4, Lab 4.7 now to tear it down.

## Troubleshooting
- **`update-virtual-service` fails** → Ensure the virtual router (step 2) was created successfully before referencing it.
- **`ResourceInUseException` on final mesh deletion** → Delete child resources strictly in this order: routes → virtual routers → virtual services → virtual nodes → mesh.
