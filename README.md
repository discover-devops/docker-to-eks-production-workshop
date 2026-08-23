# Docker to EKS: A Production Path for Developers

A hands-on workshop that takes a single application from a Dockerfile on your laptop to a public HTTPS endpoint served by Amazon EKS, redeployed automatically on every commit.

This is not a tour of AWS services. It is one application, one journey, thirteen steps.

---

## The premise

Most container training teaches Docker, then Kubernetes, then CI/CD as three unrelated subjects, and leaves the learner to figure out how they connect. This workshop inverts that. We start with an application that has a problem — it only runs on your machine — and every topic in the curriculum is introduced at the exact moment it becomes the obstacle in front of us.

You will containerise the app, discover the image is bloated and running as root, harden it, run it on a single EC2 instance, kill that instance and watch the outage, and only then reach for Kubernetes. By the time EKS appears, you will already know why you want it.

The same `catalog-api` application is used from module 1 to module 13. It is never replaced.

## What you will have built by the end

A product catalog API running on Amazon EKS across multiple availability zones, behind an Application Load Balancer with TLS, pulling configuration from AWS Secrets Manager via IRSA, built from a hardened distroless image that fails its own pipeline if a critical CVE is introduced, and deployed by a GitHub Actions workflow that authenticates to AWS with OIDC rather than long-lived access keys.

## Who this is for

Developers who ship application code and need to own it through to production. You are expected to be comfortable with a terminal, Git, and at least one programming language. You are **not** expected to have prior Kubernetes, Terraform, or AWS networking experience.

Infrastructure topics in this workshop are deliberately scoped to what a developer needs to be effective and safe — not to what a platform engineer needs to design a landing zone.

## Curriculum

| # | Module | Focus | Duration |
|---|--------|-------|----------|
| 00 | Orientation | The application, the goal, the shape of the day | 10 min |
| 01 | Docker images and containers | Dockerfile, layers, build cache, port mapping, ephemerality | 40 min |
| 02 | Image hardening | Multistage builds, distroless, non-root, Trivy, ECR scan-on-push | 40 min |
| 03 | EC2 hosting and load balancing | Single instance, ALB, and a deliberate outage | 20 min |
| 04 | VPC and networking basics | Subnets, AZs, route tables, security groups — read-a-diagram depth | 20 min |
| 05 | Launching the EKS cluster | Control plane vs. node groups, `eksctl`, kubeconfig, IAM OIDC | 30 min |
| 06 | Kubernetes Deployments | Pods, replicas, rolling updates, self-healing, rollback | 30 min |
| 07 | Kubernetes Services | ClusterIP, NodePort, LoadBalancer — and how it maps to module 3 | 25 min |
| 08 | Ingress and the AWS Load Balancer Controller | Host and path routing, TLS via ACM | 30 min |
| 09 | Configuration and secrets | ConfigMaps, Kubernetes Secrets, Secrets Manager, IRSA | 25 min |
| 10 | Introduction to GitHub Actions | Workflows, jobs, steps, runners, triggers | 20 min |
| 11 | Building the pipeline | Build, scan, push to ECR via OIDC, deploy to EKS | 45 min |
| 12 | Workflow efficiency and DevSecOps | Caching, path filters, concurrency, environments, reusable workflows | 25 min |
| 13 | Wrap-up | Ship a bad commit, watch it fail. Ship a good one, watch it go live. | 15 min |

Total: approximately 6 hours including breaks.

## Repository structure

```
.
├── app/                              # The single application, used end to end
│   ├── app.py
│   ├── requirements.txt
│   └── .dockerignore
├── module-01-docker-fundamentals/
├── module-02-image-hardening/
├── module-03-ec2-load-balancing/
├── module-04-vpc-networking/
├── module-05-eks-cluster/
├── module-06-deployments/
├── module-07-services/
├── module-08-ingress/
├── module-09-config-and-secrets/
├── module-10-github-actions-intro/
├── module-11-cicd-pipeline/
├── module-12-workflow-optimization/
├── module-13-wrap-up/
├── .github/workflows/                # The pipeline built in modules 11 and 12
└── README.md
```

Every module directory contains its own `README.md` with the concept briefing and the full lab. Each lab is self-contained: it creates every object it needs from scratch and removes them at the end. No lab assumes state left behind by a previous one, so you can drop into module 7 on a Tuesday evening without having run modules 1 through 6 that morning.

## Prerequisites

**Accounts**

- An AWS account with administrative access. A personal or sandbox account is strongly preferred over a corporate one.
- A GitHub account.

**Local tooling**

| Tool | Minimum version | Verify with |
|------|-----------------|-------------|
| Docker | 24.x | `docker version` |
| AWS CLI | 2.x | `aws --version` |
| eksctl | 0.190+ | `eksctl version` |
| kubectl | 1.30+ | `kubectl version --client` |
| Helm | 3.x | `helm version` |
| Trivy | 0.50+ | `trivy --version` |
| Git | any recent | `git --version` |

**Before you arrive**

Run `aws sts get-caller-identity` and confirm it returns your account ID. If it does not, fix your credentials before the session starts — we will not have time to debug IAM configuration live.

**A note on Apple Silicon:** if you are on an M-series Mac, images build for `arm64` by default and will not run on `amd64` EKS nodes. Either pass `--platform linux/amd64` on every build from module 1 onward, or run the entire workshop from a Linux EC2 instance. Decide this before module 1, not during module 6.

## Cost

This workshop provisions real, billable AWS infrastructure — an EKS control plane, managed node group, NAT gateway, and Application Load Balancer. Expect a cost in the low single-digit dollars per hour while the environment is running. Verify current pricing for your region before you begin.

**Tear down everything when you are finished.** Module 13 includes a complete teardown procedure. The NAT gateway and the EKS control plane bill continuously whether or not you are using them, and a cluster forgotten over a weekend is an expensive lesson in a different subject than the one this workshop teaches.

## How to use this repository

**During the live session:** clone the repo and follow the module README as the instructor works. Every command in the lab is copy-pasteable. Do not read ahead — several modules depend on you seeing a failure before you see the fix.

**Self-paced afterwards:** the modules stand alone. Start at module 1 and work through in order for the full arc, or jump to a specific module if you need one technique in isolation.

```bash
git clone https://github.com/discover-devops/docker-to-eks-production-workshop.git
cd docker-to-eks-production-workshop
```

## Design principles

**One application, thirteen modules.** Nothing is introduced with a throwaway `nginx` example that gets discarded.

**Every lab is self-contained.** Prerequisites are recreated, objects are cleaned up. State is never assumed.

**Security is not a module.** Image hardening, non-root execution, vulnerability gates, OIDC federation, and secrets management appear where they naturally belong in the delivery path, not in an appendix nobody reaches.

**Show the failure first.** The case for Kubernetes is made by killing an EC2 instance, not by a slide comparing orchestrators.

---

## Author

**Rahul Chaubey** — Solution Architect and AI & Cloud Transformation Leader.

Twenty years across AWS, Microsoft, and Oracle, designing and delivering cloud and container platforms for enterprise workloads. Rahul works at the intersection of cloud architecture, DevSecOps, and applied AI, and writes and teaches on infrastructure and platform engineering for practising developers.

- YouTube: [Discover DevOps](https://www.youtube.com/@BuildAutomateArchitectgit statu   )
- GitHub: [@discover-devops](https://github.com/discover-devops)

## License

MIT. Use it, fork it, teach from it. Attribution appreciated.