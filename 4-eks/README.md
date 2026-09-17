# Module 4: Kubernetes on EKS (Shared Cluster)

One shared EKS cluster (Lab 4.0, instructor-only) with per-participant `ns-$PARTICIPANT` namespaces. Tag every EC2/RDS/EKS/ALB resource you create with `Owner=$PARTICIPANT` and `Course=cloudnative-agentic-ai-2026` or the create call will be denied by the permission boundary.

## Labs

- [4.0-INSTRUCTOR-ONLY-create-cluster.md](4.0-INSTRUCTOR-ONLY-create-cluster.md) — Lab 4.0 (Instructor Only): Provision the Shared EKS Cluster (⚠️ ~$0.10/hr control plane + node costs)
- [4.1-connect-to-shared-cluster.md](4.1-connect-to-shared-cluster.md) — Lab 4.1: Connect to the Shared Cluster and Verify Your Namespace (✅ Free (cluster cost is shared, already running))
- [4.2-deployments-services.md](4.2-deployments-services.md) — Lab 4.2: Deployments and Services in Your Namespace (✅ Free (shared cluster))
- [4.3-alb-ingress.md](4.3-alb-ingress.md) — Lab 4.3: Exposing a Service via ALB (⚠️ ALB hourly cost, minimal for lab duration)
- [4.4-rolling-updates.md](4.4-rolling-updates.md) — Lab 4.4: Rolling Updates and Rollbacks (✅ Free (shared cluster))
- [4.5-configmaps-secrets.md](4.5-configmaps-secrets.md) — Lab 4.5: ConfigMaps and Secrets (✅ Free (shared cluster))
- [4.6-hpa.md](4.6-hpa.md) — Lab 4.6: Horizontal Pod Autoscaling (✅ Free (shared cluster))
- [4.7-cleanup.md](4.7-cleanup.md) — Lab 4.7: Namespace Cleanup (Students) / Full Teardown (Instructor) (Varies)
