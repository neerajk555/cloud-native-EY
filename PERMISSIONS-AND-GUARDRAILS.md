# Permissions & Guardrails

This course runs in **one shared AWS account** with 25 individual IAM
users (Ubuntu + AWS CLI, no per-participant AWS accounts). Two policies
govern everything you can and can't do — both live in
`aws-training-setup/policies/` and are mirrored here for reference. If
you get `AccessDenied`, start with the self-check at the bottom.

> This file must stay in sync with `aws-training-setup/policies/*.json`
> and `aws-training-setup/SYNC-NOTES.md`. If you edit one, edit all three.

## Your identity

Every script in these labs uses one exported shell variable, set once in
Module 1:

```bash
export PARTICIPANT=$(aws iam get-user --query "User.UserName" --output text)
```

Use `$PARTICIPANT` everywhere a resource name or `Owner` tag is needed —
never hardcode your username, and never use someone else's.

## What you have full access to (us-east-1 only)

EC2, RDS, S3, Lambda, EKS, ALB/NLB, DynamoDB, Bedrock (3 approved models
only — see below), API Gateway, CloudWatch, CloudWatch Logs.

## What you have scoped access to (us-east-1 only)

SQS, SNS, EventBridge, ECR, Step Functions, Cognito, App Mesh, X-Ray,
CodeBuild, CodePipeline, Secrets Manager, AWS Backup.

ElastiCache and Amazon Managed Grafana are granted **for the instructor
account only**, for the Module 9.4 / 11.1 demos. Participants use the
free alternatives in those two labs (local Docker Redis; a CloudWatch
Dashboard) instead.

## Hard limits (Deny — cannot be worked around)

- Everything must run in **us-east-1**.
- EC2 instances: `t2.micro` or `t3.micro` only. Max 10 per `RunInstances`
  call, max 10 running at a time per participant (soft-enforced by
  `quota_enforcer.py` on a 15–30 min cycle).
- RDS instances: `db.t3.micro` or `db.t4g.micro` only, single-AZ only.
- No NAT Gateways, ever.
- No EKS Fargate profiles.
- Redshift, EMR, OpenSearch/Elasticsearch Service, SageMaker, WorkSpaces,
  Global Accelerator are blocked entirely (not used by this course; belt
  and suspenders).
- Bedrock: only `amazon.nova-micro-v1:0`, `amazon.nova-lite-v1:0`, and
  `anthropic.claude-3-haiku-20240307-v1:0` can be invoked. Every other
  model is denied regardless of what the console shows as "available."
- `Owner` + `Course` tags are **required at creation time** for EC2
  instances, RDS instances, EKS clusters, and load balancers, or the
  create call is denied outright. Set `Owner` = your `$PARTICIPANT`
  value, `Course` = `cloudnative-agentic-ai-2026`.
- You cannot create IAM users, roles, or policies, attach policies to
  yourself, or modify your own permissions boundary.

## The 9 shared execution roles

Several labs need a Lambda/CodeBuild/CodePipeline/Step Functions/Backup
service role. You **cannot create your own** — instead, `iam:PassRole`
and `iam:GetRole` are granted on exactly these 9 pre-created ARNs, and
nothing else:

| Role | Used in |
|---|---|
| `course-lambda-basic-execution` | Modules 2, 6, 9 |
| `course-lambda-sqs-execution` | Module 11.2 |
| `course-lambda-xray-execution` | Module 9.3 |
| `course-apigw-lambda-execution` | Module 6.1 |
| `course-eventbridge-lambda-execution` | Module 2.3 |
| `course-backup-service-role` | Module 11.3 |
| `course-stepfunctions-execution` | Module 5.3 |
| `course-codebuild-execution` | Module 8.1 |
| `course-codepipeline-execution` | Module 8.2 |

Fetch a role's ARN when a lab needs it — never try to create one:

```bash
aws iam get-role --role-name course-lambda-basic-execution --query "Role.Arn" --output text
```

If this returns `NoSuchEntity`, the instructor hasn't run
`create_shared_execution_roles.ps1` yet — ask before continuing.

## Module 4 (EKS): one shared cluster, not one per participant

25 participants each running `eksctl create cluster` would blow past the
default 5-VPC/5-EIP region quota almost immediately, and cost 25×
$0.10/hr in control planes. Instead: the instructor provisions **one**
shared EKS cluster in Lab 4.0 (instructor-only), and you get your own
Kubernetes namespace, `ns-$PARTICIPANT`, with a `ResourceQuota` and
`LimitRange` sized for `t3.micro` nodes. Every Module 4–10 lab that
touches the cluster works inside your namespace only — you will not have
cluster-admin, and that's intentional.

## Self-check for AccessDenied errors

1. Is the region `us-east-1`? Check `aws configure get region` and any
   `--region` flag on the failing command.
2. Is the resource type/size on the approved list (instance type, DB
   class, Bedrock model)?
3. Did the create call include `Owner=$PARTICIPANT` and
   `Course=cloudnative-agentic-ai-2026` tags, if it's EC2/RDS/EKS/ALB?
4. If the error mentions `iam:PassRole`, are you passing one of the 9
   ARNs above, exactly? A typo'd role name or the wrong service reads as
   a denial, not a "not found."
5. Still stuck? Run
   `aws iam simulate-principal-policy --policy-source-arn <your-user-arn> --action-names <the-denied-action> --resource-arns <the-resource-arn>`
   — this is one of the few IAM actions you're explicitly granted, and it
   will tell you exactly which statement denied you.
