# Cloud Native Application Development on AWS — Hands-On Labs

Modules 1–11 of the course syllabus (Agentic AI modules 12–14 and the
Capstone are not included here). Region: **us-east-1** only. Platform:
**Ubuntu + AWS CLI + bash**. Runs against one shared AWS account
with 25 individually-provisioned IAM users.

**Start here, in order:**

1. `SETUP-UBUNTU.md` — install prerequisites, configure AWS CLI
2. `PERMISSIONS-AND-GUARDRAILS.md` — what you can/can't do and why, the
   9 shared execution roles, and the AccessDenied self-check
3. Module 0 (optional CLI warm-up), then Modules 1–11 in order

If you also want the non-AWS Docker/Kubernetes fundamentals track,
see the sibling folder `../local-docker-kubernetes-labs/` — it runs on
your own machine, needs no AWS account, and Module L3 (Kubernetes with
`kind`) is recommended **before** this repo's Module 4 (EKS) if you've
never used Kubernetes before.

## Modules

| Module | Topic | Notes |
|---|---|---|
| 0 | CLI Fundamentals (optional) | Not AWS-billed |
| 1 | Intro & Account Access | Sets `$PARTICIPANT`, used everywhere after |
| 2 | Microservices Fundamentals | Lambda, EventBridge — shared roles |
| 3 | Docker & ECR | Personal ECR repo per participant |
| 4 | Kubernetes on EKS | **Shared cluster** — see PERMISSIONS-AND-GUARDRAILS.md |
| 5 | Design Patterns | Sidecar, circuit breaker, saga |
| 6 | API Gateway & Cognito | Shared apigw role |
| 7 | Service Mesh (App Mesh) | No EKS dependency |
| 8 | CI/CD | CodeBuild/CodePipeline — 2 roles added, see SYNC-NOTES.md |
| 9 | Observability | Grafana is instructor-demo-only |
| 10 | Security | No self-service IAM role creation anywhere |
| 11 | Data Management | ElastiCache is instructor-demo-only |

## Every lab is cost-labeled

- ✅ Free Tier — no cost expected within lab-duration usage
- ⚠️ Minimal Cost — small real charge, called out explicitly (never
  mislabeled as free)

## Related documents

- `../aws-training-setup/README.md` — the operator side: how the 25
  users, boundary, and shared roles were set up
- `../aws-training-setup/SYNC-NOTES.md` — how conflicting earlier designs
  were reconciled into what's here now; read this if anything looks
  inconsistent with an older version you've seen
