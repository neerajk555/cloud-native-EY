# Module 4: Container Orchestration with Amazon EKS (Kubernetes)

⚠️ **Cost Notice:** Amazon EKS is **not** covered by the AWS Free Tier under any circumstances. The EKS control plane costs **$0.10/hour (~$2.40/day)** regardless of usage, and worker nodes / load balancers add further small charges. This module is designed to minimize cost: **create the cluster once at the start of Lab 4.1, do all labs 4.1–4.6 in one sitting (~90 minutes total), then immediately run Lab 4.7 to tear everything down.**

| Lab | Title | Difficulty | Duration | Cost |
|---|---|---|---|---|
| 01 | Create an EKS Cluster with eksctl | Beginner | 15 min | ⚠️ ~$0.10/hr control plane + node costs |
| 02 | Deploy a Pod & Deployment | Beginner | 15 min | ⚠️ Uses cluster from Lab 4.1 |
| 03 | Expose an App via ALB Service | Intermediate | 15 min | ⚠️ + ALB (~$0.0225/hr) |
| 04 | ConfigMaps & Secrets Manager Integration | Beginner | 15 min | ⚠️ Uses cluster from Lab 4.1 |
| 05 | Horizontal Pod Autoscaler (HPA) | Intermediate | 15 min | ⚠️ Uses cluster from Lab 4.1 |
| 06 | Install a Helm Chart | Beginner | 15 min | ⚠️ Uses cluster from Lab 4.1 |
| 07 | **Full Cluster Teardown (do this last!)** | Beginner | 15 min | ✅ Stops all charges |

**Estimated total cost if completed in one ~2-hour sitting and torn down immediately after: approximately $0.30–$0.60.**
