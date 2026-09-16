# Lab 1.3: Console Tour — EC2 vs Elastic Beanstalk vs Lambda vs EKS

**Module:** 1 — Introduction to Cloud Native Computing on AWS
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free Tier Eligible (this lab only browses the console — no resources are launched)

## Objective
Build a mental map of AWS's IaaS/PaaS/serverless/container spectrum by browsing the console pages for EC2, Elastic Beanstalk, Lambda, and EKS, without launching anything.

## Prerequisites
- Completed Lab 1.1

## Steps

1. Sign in to the console in `us-east-1`.
2. **EC2 (IaaS)**
   - Search **EC2** and open the dashboard.
   - Click **Launch instance** to open the launch wizard (do **not** launch — just look).
   - Note everything *you* are responsible for: OS, patching, scaling, networking, storage — this is "raw compute."
   - Click **Cancel** or navigate away without launching.
3. **Elastic Beanstalk (PaaS)**
   - Search **Elastic Beanstalk** and open it.
   - Click **Create application** to view the wizard (do **not** create).
   - Note that you only choose a platform (e.g., Docker, Node.js, Python) and upload code — Beanstalk manages the EC2 instances, load balancer, and scaling for you.
   - Navigate away without creating anything.
4. **Lambda (Serverless / FaaS)**
   - Search **Lambda** and open it.
   - Click **Create function** to view the wizard (do **not** create).
   - Note there's no server concept at all — just a function, a trigger, and a runtime. This is the far serverless end of the spectrum.
   - Navigate away without creating anything.
5. **EKS (Managed Kubernetes / Cloud Native container orchestration)**
   - Search **EKS** and open **Amazon Elastic Kubernetes Service**.
   - Click **Add cluster → Create** to view the wizard (do **not** create — full cluster creation is covered with cost warnings in Module 4).
   - Note the concepts introduced: cluster, node groups, Fargate profiles — this is where "cloud native" workloads in this course will actually run.
   - Navigate away without creating anything.
6. Write a one-paragraph note to yourself (in a scratch file) comparing where each of the four services sits on the "how much do I manage vs. how much does AWS manage" spectrum. This mental model will be referenced throughout the course.

## Expected Result / Validation
You should be able to explain, in your own words, why EKS/containers are considered "more cloud native" than a single EC2 instance, and where Lambda and Beanstalk fit in between.

## Cleanup
No resources were created — nothing to clean up. Just make sure you clicked away from any "Launch/Create" wizard without confirming, so nothing was accidentally provisioned.

## Troubleshooting
- **Console asks to confirm a launch you didn't intend** → Always click **Cancel** rather than closing the browser tab, to make sure the wizard doesn't submit in the background.
