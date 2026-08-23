# Docker to EKS: A Production Path for Developers

A hands-on workshop that takes a two-service application from bare EC2 instances, through Docker, to Amazon EKS — following the same path the industry itself took, one problem at a time.

This is not a tour of AWS services. It is one application, one journey, and every tool introduced as the answer to a problem you have already felt.

---

## The premise

Most container training starts with Docker on slide one and Kubernetes on slide forty, and never explains what either of them replaced. You come away knowing the commands and not the reasons.

This workshop starts where production actually started.

You will deploy two services onto two EC2 instances by hand — SSH in, install the runtime, copy the code, write a systemd unit. You will put an Application Load Balancer in front with path-based routing, and you will have a genuinely working microservice deployment.

Then we look honestly at what it costs to keep running: manual deployments, servers drifting apart, AMI rebuilds to scale, and nobody noticing when an instance dies. **That pain is what Docker solves**, so Docker comes next — the same two services, containerized, hardened, scanned, and pushed to ECR.

Then we break that too. Docker on a single host fixes packaging, but it will not restart a crashed container, spread load across machines, or help one service find another. **That pain is what Kubernetes solves.** So we finish on EKS: Deployments, Services, and an Ingress that lands right back at an Application Load Balancer — except this time nobody configured it by hand.

Nothing in this workshop appears because it is on a syllabus.

## The application

Two services, deliberately shaped like the ones behind any e-commerce platform.

**`catalog-api`** owns product data. **`orders-api`** owns orders and calls `catalog-api` over HTTP to enrich them with product names and prices.

That second service is the point. Because `orders-api` has to *find* `catalog-api` across the network, every stage of the workshop has to answer the same question — and the answer changes each time:

| Stage | How `orders-api` finds `catalog-api` |
|---|---|
| EC2 | A hardcoded private IP address |
| Docker | A container name on a Docker network |
| Kubernetes | A Service DNS name that survives pods being replaced |

The application code is written once in Module 00 and **never changes again**. Everything that follows is a deployment concern, not an application concern.

## What you will have built by the end

Two services running on Amazon EKS across multiple availability zones, behind an Application Load Balancer with TLS and path-based routing, discovering each other through Kubernetes DNS, built from hardened multi-stage images that fail their own pipeline if a critical CVE is introduced, and deployed by a GitHub Actions workflow that authenticates to AWS with OIDC rather than long-lived access keys.

## Who this is for

Developers who ship application code and need to own it through to production. You should be comfortable with a terminal, Git, and at least one programming language. You are **not** expected to have prior Kubernetes or container experience.

Infrastructure topics here are scoped to what a developer needs to be effective and safe — not to what a platform engineer needs to design a landing zone.

## Curriculum

### Part 1 — Traditional deployment

| # | Module | Focus |
|---|---|---|
| 00 | [Workshop kickoff](./00-workshop-kickoff.md) | The application, the journey, environment setup |
| 01 | [AWS networking recap](./01-aws-networking-recap.md) | VPC, subnets, route tables, IGW, NAT — the ground everything sits on |
| 02 | [EC2 hosting](./02-ec2-hosting-manual-deployment.md) | Deploy both services by hand, systemd units, hardcoded IPs |
| 03 | [Load balancing](./03-load-balancing-alb-nlb.md) | ALB vs NLB, target groups, health checks, path-based routing |
| 04 | [The limits of EC2](./04-limits-of-ec2-scaling.md) | Config drift, AMI baking, slow scale-out, manual deploys |

### Part 2 — Containers

| # | Module | Focus |
|---|---|---|
| 05 | [Docker fundamentals](./05-docker-fundamentals.md) | Images, containers, layers, build cache, Docker architecture |
| 06 | [Containerizing the application](./06-containerizing-the-application.md) | Both services in containers, Docker networks, service discovery by name |
| 07 | [Image hardening](./07-image-hardening-security.md) | Multi-stage builds, non-root, distroless, Trivy, ECR scan-on-push |
| 08 | [The limits of Docker](./08-limits-of-docker-hosts.md) | No self-healing, no scaling, no scheduling, one host to lose |

### Part 3 — Kubernetes

| # | Module | Focus |
|---|---|---|
| 09 | [Kubernetes concepts and EKS](./09-kubernetes-concepts-eks-cluster.md) | Control plane, nodes, the reconciliation loop, `eksctl` |
| 10 | [Deployments](./10-kubernetes-deployments.md) | Pods, ReplicaSets, rolling updates, self-healing, rollback |
| 11 | [Services](./11-kubernetes-services.md) | ClusterIP, NodePort, LoadBalancer, and DNS-based service discovery |
| 12 | [Ingress and HTTPS](./12-ingress-https.md) | AWS Load Balancer Controller, path routing, TLS via ACM |

### Part 4 — Automation and DevSecOps

| # | Module | Focus |
|---|---|---|
| 13 | [GitHub Actions fundamentals](./13-github-actions-fundamentals.md) | Workflows, jobs, steps, runners, triggers |
| 14 | [End-to-end CI/CD](./14-end-to-end-cicd.md) | Build, scan, push to ECR via OIDC, deploy to EKS |
| 15 | [Optimization and DevSecOps](./15-cicd-optimization-devsecops.md) | Caching, path filters, concurrency, environments, secrets |
| 16 | [Final production demo](./16-final-production-demo.md) | Ship a bad commit, watch it fail. Ship a good one, watch it go live. |

Parts 1 to 3 form a complete session. Part 4 is typically delivered separately.

## Repository structure

```
.
├── 00-workshop-kickoff.md
├── 01-aws-networking-recap.md
├── 02-ec2-hosting-manual-deployment.md
├── 03-load-balancing-alb-nlb.md
├── 04-limits-of-ec2-scaling.md
├── 05-docker-fundamentals.md
├── 06-containerizing-the-application.md
├── 07-image-hardening-security.md
├── 08-limits-of-docker-hosts.md
├── 09-kubernetes-concepts-eks-cluster.md
├── 10-kubernetes-deployments.md
├── 11-kubernetes-services.md
├── 12-ingress-https.md
├── 13-github-actions-fundamentals.md
├── 14-end-to-end-cicd.md
├── 15-cicd-optimization-devsecops.md
├── 16-final-production-demo.md
└── README.md
```

Each module file contains its own context, concepts, and a complete step-by-step lab.

**Every lab is self-contained.** It creates every object it needs from scratch and removes them at the end. No lab assumes state left behind by a previous one, so you can drop into Module 07 on a Tuesday evening without having run Modules 00 through 06 that morning.

## Prerequisites

**Accounts**

- An AWS account with administrative access. A personal or sandbox account is strongly preferred over a corporate one.
- A GitHub account (needed from Module 13 onward).

**Local tooling**

| Tool | Minimum | Verify with |
|---|---|---|
| AWS CLI | 2.x | `aws --version` |
| Docker | 24.x | `docker version` |
| eksctl | 0.190+ | `eksctl version` |
| kubectl | 1.30+ | `kubectl version --client` |
| Helm | 3.x | `helm version` |
| Trivy | 0.50+ | `trivy --version` |
| Python | 3.10+ | `python3 --version` |
| Git | any recent | `git --version` |

**Before you arrive**

Run `aws sts get-caller-identity` and confirm it returns your account ID. If it does not, fix your credentials before the session starts — there will not be time to debug IAM configuration live.

**Apple Silicon Macs:** images build for `arm64` by default and will not run on `amd64` EKS nodes. Either pass `--platform linux/amd64` on every build from Module 06 onward, or run the whole workshop from a Linux EC2 instance. Decide this before Module 06, not during Module 10.

## Cost

This workshop provisions real, billable AWS infrastructure — EC2 instances, a NAT gateway, Application Load Balancers, an ECR repository, an EKS control plane, and a managed node group. Verify current pricing for your region before you begin.

**Tear down everything when you finish.** Every lab ends with a teardown section, and Module 16 includes a full sweep. The NAT gateway and the EKS control plane bill continuously whether or not you are using them, and a cluster forgotten over a weekend is an expensive lesson in an entirely different subject.

## How to use this repository

**During the live session:** clone the repo and follow the module file as the instructor works. Every command is copy-pasteable. Do not read ahead — several modules depend on you seeing a failure before you see the fix.

**Self-paced afterwards:** start at Module 00 and work through in order for the full narrative, or jump to a specific module if you need one technique in isolation.

```bash
git clone https://github.com/discover-devops/docker-to-eks-production-workshop.git
cd docker-to-eks-production-workshop
```

## Design principles

**Two services, seventeen modules.** No throwaway `nginx` examples that get discarded on the next slide.

**The application code never changes.** Written once in Module 00, deployed six different ways. Every difference you see is infrastructure, not code.

**Show the failure before the fix.** The case for Kubernetes is made by killing an EC2 instance and watching an outage, not by a slide comparing orchestrators.

**Security is not a module.** Image hardening, non-root execution, vulnerability gates, OIDC federation, and secrets management appear where they belong in the delivery path, not in an appendix nobody reaches.

**Every lab is self-contained.** Prerequisites are recreated, objects are cleaned up. State is never assumed.

---

## Author

**Rahul Chaubey** — Solution Architect and AI & Cloud Transformation Leader.

Twenty years across AWS, Microsoft, and Oracle, designing and delivering cloud and container platforms for enterprise workloads. Rahul works at the intersection of cloud architecture, DevSecOps, and applied AI, and writes and teaches infrastructure and platform engineering for practising developers.

- YouTube: [Build Automate Architect](https://www.youtube.com/@BuildAutomateArchitect)
- GitHub: [@discover-devops](https://github.com/discover-devops)

## License

MIT. Use it, fork it, teach from it. Attribution appreciated.