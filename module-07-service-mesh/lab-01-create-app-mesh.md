# Lab 7.1: Create an App Mesh (Console)

**Module:** 7 — Service Mesh & Inter-Service Communication
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ⚠️ The App Mesh resource itself is **free** — AWS does not charge for meshes, virtual nodes, virtual services, or virtual routers. You are only billed for the EKS cluster compute they run on top of (see Module 4 for those costs). This lab creates mesh **configuration objects only** — no compute is spun up here.

## Objective
Create your first AWS App Mesh and understand its core building blocks — mesh, virtual node, virtual service — before wiring up traffic routing in the next lab.

## Prerequisites
- Completed Lab 1.1 (CLI configured)
- No EKS cluster is strictly required for this specific lab, since we're only creating the mesh's logical configuration objects, not deploying mesh-enabled workloads yet

## Steps

1. **Create a mesh:**
   ```bash
   aws appmesh create-mesh --mesh-name cloudnative-demo-mesh --region us-east-1
   ```
2. **Create a virtual node** representing a hypothetical "orders" microservice:
   ```bash
   aws appmesh create-virtual-node \
     --mesh-name cloudnative-demo-mesh \
     --virtual-node-name orders-service-vn \
     --spec '{
       "listeners": [{"portMapping": {"port": 8080, "protocol": "http"}}],
       "serviceDiscovery": {"dns": {"hostname": "orders.svc.cluster.local"}}
     }'
   ```
3. **Create a virtual service** that points to that virtual node:
   ```bash
   aws appmesh create-virtual-service \
     --mesh-name cloudnative-demo-mesh \
     --virtual-service-name orders.svc.cluster.local \
     --spec '{
       "provider": {"virtualNode": {"virtualNodeName": "orders-service-vn"}}
     }'
   ```
4. **List everything you created:**
   ```bash
   aws appmesh list-meshes
   aws appmesh list-virtual-nodes --mesh-name cloudnative-demo-mesh
   aws appmesh list-virtual-services --mesh-name cloudnative-demo-mesh
   ```

## Expected Result / Validation
`list-virtual-services` should show `orders.svc.cluster.local` with `meshName: cloudnative-demo-mesh`, and `list-virtual-nodes` should show `orders-service-vn`. This confirms your mesh's logical topology is defined — the next lab adds a virtual router to control traffic splitting between multiple versions of this service.

## Cleanup
**Skip cleanup here if you're continuing directly to Lab 7.2**, which builds on this mesh. Otherwise:
```bash
aws appmesh delete-virtual-service --mesh-name cloudnative-demo-mesh --virtual-service-name orders.svc.cluster.local
aws appmesh delete-virtual-node --mesh-name cloudnative-demo-mesh --virtual-node-name orders-service-vn
aws appmesh delete-mesh --mesh-name cloudnative-demo-mesh
```
Verify:
```bash
aws appmesh list-meshes
```
`cloudnative-demo-mesh` should not appear.

## Troubleshooting
- **`ResourceInUseException` deleting the mesh** → Delete virtual services and virtual nodes first (as shown above) before deleting the mesh itself; App Mesh enforces this dependency order.
