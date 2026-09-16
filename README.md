# Cloud Native Application Development on AWS — Hands-On Labs

This repository contains beginner-friendly, hands-on lab exercises that accompany the **"Cloud Native Application Development on AWS"** course. Every lab is self-contained, takes **15 minutes or less**, and includes step-by-step instructions plus mandatory **cleanup steps** so you never leave paid resources running by accident.

> **Current scope:** Module 0 (Linux fundamentals warm-up) plus Modules 1–11 (Cloud Native fundamentals through Data Management). Agentic AI modules (12–14) and the Capstone (15) will be added in a future update.

## How to Use This Repository

1. Clone or download this repo.
2. **If you're on a fresh Ubuntu machine**, run through [`SETUP-UBUNTU.md`](./SETUP-UBUNTU.md) first — it installs every tool used across all modules (AWS CLI, Docker, kubectl, eksctl, Helm, Node.js, etc.) in one pass.
3. **If you're new to the Linux command line**, start with `module-00-linux-command-line-basics/` before Module 1. If you're already comfortable in a terminal, skip straight to Module 1.
4. Work through the modules in order — each module folder maps 1:1 to a course module.
5. Inside each module folder you'll find numbered lab files: `lab-01-....md`, `lab-02-....md`, etc.
6. Open a lab file, read it top to bottom, and follow the steps in your own AWS account.
7. **Always run the Cleanup section at the end of every lab before moving on**, even if you plan to come back to it later that day.

## Prerequisites (do this once, before Module 1)

- An AWS account ([sign up here](https://aws.amazon.com/free/) if you don't have one)
- AWS root/IAM user with console access
- An Ubuntu machine (or any Linux/macOS/WSL2 environment) with the tools listed in [`SETUP-UBUNTU.md`](./SETUP-UBUNTU.md) installed
- A code editor (VS Code recommended)
- Basic command-line comfort (`cd`, `ls`, running scripts) — if this isn't you yet, do Module 0 first

Module 1, Lab 1 walks through AWS account and CLI setup in detail — start there once your machine is set up.

## Region Used in All Labs

All labs in this repository use **`us-east-1` (N. Virginia)**. Set this as your default region during CLI configuration so commands work exactly as written. If your organization mandates a different region, most commands will still work if you swap the region flag — just note that pricing/availability of some newer services can vary by region.

```bash
aws configure set region us-east-1
```

## ⚠️ Important: Read This Before You Start — AWS Free Tier Reality Check

This course is **free-tier-eligible wherever AWS makes that possible**, and every lab clearly states its cost status at the top. But please understand:

| Label | What it means |
|---|---|
| ✅ **Free Tier Eligible** | Fits within AWS's 12-month or "Always Free" tier limits, *if* you haven't already exhausted that allowance elsewhere in your account. |
| ⚠️ **Minimal Cost (not Free Tier)** | Some AWS services (notably **EKS control plane, Application Load Balancers, NAT Gateways, App Mesh, Amazon Managed Grafana, ElastiCache, MSK**) are **never** part of the Free Tier, regardless of usage. These labs are designed to minimize spend (smallest instance sizes, shortest possible runtime, immediate cleanup) and each one states an **estimated cost** up front. |

**You are responsible for monitoring your own AWS bill.** Recommended safety nets:
- Set up a [Billing Alarm](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html) at $5 or $10 before you begin Module 1.
- Enable **AWS Budgets** with an email alert.
- Never skip a lab's Cleanup section.
- After each session, check the [Billing Dashboard](https://us-east-1.console.aws.amazon.com/billing/home) for any unexpected running resources.

## Module Index & Cost Status at a Glance

| Module | Topic | Overall Cost Profile |
|---|---|---|
| 0 | Linux Command Line Basics (prerequisite warm-up) | ✅ Free (local only) |
| 1 | Introduction to Cloud Native Computing on AWS | ✅ Free Tier |
| 2 | Microservices Architecture Fundamentals | ✅ Free Tier |
| 3 | Containerization with Docker & Amazon ECR | ✅ Free Tier |
| 4 | Container Orchestration with Amazon EKS | ⚠️ Minimal Cost (EKS control plane + ALB are not free-tier) |
| 5 | Cloud Native Design Patterns | ✅ Mostly Free Tier (1 lab reuses EKS from Module 4) |
| 6 | API Design & Management with API Gateway | ✅ Free Tier |
| 7 | Service Mesh & Inter-Service Communication | ⚠️ Minimal Cost (App Mesh has no free tier) |
| 8 | CI/CD for Cloud Native Applications | ✅ Free Tier |
| 9 | Observability: Logging, Monitoring & Tracing | ⚠️ Mostly free; 1 lab (Managed Grafana) has minimal cost |
| 10 | Cloud Native Security on AWS | ✅ Free Tier |
| 11 | Cloud Native Data Management on AWS | ⚠️ Mostly free; 1 lab (ElastiCache) has minimal cost |

Each module folder also has its own `README.md` listing its labs and their individual cost labels.

## License

Free to use, adapt, and redistribute for training and educational purposes.
