# Module 7: Service Mesh & Inter-Service Communication (AWS App Mesh)

⚠️ **Cost Notice:** AWS App Mesh's control plane configuration objects (mesh, virtual services, virtual routers) themselves are **free** — you only pay for the underlying EKS/ECS/EC2 compute they run on. However, this module's labs are written to require an EKS cluster (from Module 4), which does have a per-hour cost. **Recreate the EKS cluster (Module 4, Lab 4.1) just before Lab 7.1, and tear it down (Module 4, Lab 4.7) right after Lab 7.2.**

| Lab | Title | Difficulty | Duration | Cost |
|---|---|---|---|---|
| 01 | Create an App Mesh (Console) | Beginner | 15 min | ⚠️ Mesh itself is free; requires running EKS cluster |
| 02 | Configure a Virtual Service & Router | Intermediate | 15 min | ⚠️ Mesh itself is free; requires running EKS cluster |
