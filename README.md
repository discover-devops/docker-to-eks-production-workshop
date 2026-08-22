### `README.md`

### Docker to EKS Production Workshop

> Build, Secure, and Deploy a Production-Ready Application on Amazon EKS with Automated CI/CD

### Workshop Goal

By the end of this workshop, you will deploy a Flask Product Catalog API to Amazon EKS, expose it over HTTPS using an AWS Load Balancer, and automate the entire deployment using GitHub Actions, Amazon ECR, and Trivy.

One application. One Dockerfile. One EKS cluster. One CI/CD pipeline.

### Architecture Journey

> (We'll add the architecture diagram later.)

### Prerequisites

* AWS Account

* GitHub Account

* Docker Desktop (or Docker Engine on Linux)

* AWS CLI

* `kubectl`

* `eksctl`

* `git`

### Workshop Modules

|
Module

|

Topic

|
| --- | --- |
|

[00](https://./00-workshop-kickoff.md) 

|

Workshop Kickoff

|
|

[01](https://./01-docker-fundamentals.md) 

|

Docker Fundamentals: Images and Containers

|
|

[02](https://./02-production-docker-security.md) 

|

Production Docker Images and Security

|
|

[03](https://./03-ec2-hosting-alb.md) 

|

Hosting Containers on EC2

|
|

[04](https://./04-aws-networking-for-eks.md) 

|

AWS Networking for EKS

|
|

[05](https://./05-create-eks-cluster.md) 

|

Creating an Amazon EKS Cluster

|
|

[06](https://./06-kubernetes-deployments.md) 

|

Kubernetes Deployments

|
|

[07](https://./07-kubernetes-services.md) 

|

Kubernetes Services

|
|

[08](https://./08-ingress-https.md) 

|

Ingress and HTTPS

|
|

[09](https://./09-configmaps-secrets-irsa.md) 

|

Configuration and Secrets

|
|

[10](https://./10-github-actions-fundamentals.md) 

|

GitHub Actions Fundamentals

|
|

[11](https://./11-end-to-end-cicd.md) 

|

End-to-End CI/CD Pipeline

|
|

[12](https://./12-cicd-optimization-devsecops.md) 

|

CI/CD Optimization and DevSecOps

|
|

[13](https://./13-final-production-demo.md) 

|

Final Production Demo

|

### Final Outcome

By the end of the workshop, you will have built:

* A Dockerized Flask Product Catalog API

* A production-ready Docker image

* A private Amazon ECR repository

* A highly available Amazon EKS cluster

* Kubernetes Deployments and Services

* AWS ALB-based Ingress with HTTPS

* Secure configuration using IRSA and AWS Secrets Manager

* A complete GitHub Actions CI/CD pipeline with Trivy security scanning

### Repository Structure

``` docker-to-eks-production-workshop/ ├── README.md ├── 00-workshop-kickoff.md ├── 01-docker-fundamentals.md ├── ... └── 13-final-production-demo.md ```

