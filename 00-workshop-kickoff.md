# Module 00 — Workshop Kickoff

**Duration:** 25 minutes
**You will finish this module with:** a verified toolchain, a working AWS identity, and the workshop application running on your own machine.

---

## Context

Think about what happens when India plays Pakistan in a World Cup match and everyone opens Hotstar at the same moment.

At 7:29 PM the platform is serving a few hundred thousand people. At 7:31 PM it is serving fifty million. Nobody at Hotstar is logging into a server, copying files, installing Python, and restarting a service. Nobody is on a call saying "it works on my laptop, I don't know why it's failing in production."

The traffic goes up, the platform grows to meet it, the traffic goes down, the platform shrinks. Then the match ends and the same thing happens in reverse.

That behaviour is not magic and it is not exclusive to companies with thousands of engineers. It comes from three ideas stacked on top of each other:

1. The application is packaged into an artifact that runs identically everywhere — a **container image**.
2. Something decides where those containers run, restarts them when they die, and adds more when demand rises — **Kubernetes**.
3. Getting new code from a developer's commit into that system is automatic and gated — a **CI/CD pipeline**.

Every serious platform you use daily runs on some version of this. Zomato routing your order. PhonePe settling a UPI transaction. Netflix deciding which encode of a video to serve you. Flipkart during Big Billion Days. Different companies, different languages, same three ideas.

By the end of this workshop, your application will work the same way.

### The problem we are actually solving

Here is the situation this workshop exists to fix.

You write code. It runs on your laptop. You hand it to someone else and it does not run on theirs, because they have a different Python version, or a missing library, or a different operating system. You eventually get it onto a server. That server dies at 2 AM and your application is offline until a human notices and fixes it. When traffic doubles, you have no answer except to buy a bigger server and take an outage while you migrate to it. And every deployment is a person following a checklist, which means every deployment is a person capable of skipping step four.

We are going to remove all four of those problems, in order, with the same application.

---

## Concept

### One application, thirteen modules

Most container training teaches you Docker on Monday, Kubernetes on Wednesday, and CI/CD on Friday, using three unrelated toy examples. You leave knowing three tools and not knowing how they connect.

This workshop does the opposite. There is **one** application — a product catalog API — and it appears in every single module. We never throw it away and start again with a fresh `nginx` example.

Each new topic is introduced at the exact moment it becomes the thing standing between our application and production. You will not learn Kubernetes because it is on the syllabus. You will learn it because in Module 03 we put our application on a single EC2 server, that server dies, and the application goes down — and at that moment you will want something better.

### The journey

| Where we are | What we add | What it fixes |
|---|---|---|
| App runs only on your laptop | **Docker image** | Runs identically anywhere |
| Image is 1 GB and runs as root | **Hardening + scanning** | Small, safe, auditable |
| One EC2 server | **Load balancer** | Traffic distribution |
| Server dies, app is down | **Kubernetes / EKS** | Self-healing and scaling |
| No public URL | **Service + Ingress + TLS** | Real HTTPS endpoint |
| Config baked into the image | **ConfigMaps + Secrets Manager** | Config separated from code |
| Deployment is manual | **GitHub Actions** | Automatic, gated, repeatable |

### The application

`catalog-api` is a small product catalog service — the kind of service that sits behind the listings page of any e-commerce platform. Flipkart has one. Amazon has one. Yours will be considerably smaller.

It exposes two endpoints:

| Endpoint | Purpose |
|---|---|
| `GET /health` | Liveness check. Kubernetes will use this in Module 06. |
| `GET /products` | Returns the catalog, plus the hostname of whatever is serving the request. |

That hostname field matters more than it looks. In Module 03 it will prove that a load balancer is distributing traffic across two servers. In Module 06 it will prove that Kubernetes is running multiple copies of your application. It is our evidence, not decoration.

### Vocabulary you will hear today

Read these once now. They will make sense properly when you meet them in a lab.

| Term | Working definition |
|---|---|
| **Image** | A packaged, read-only bundle of your app and everything it needs to run. |
| **Container** | A running instance of an image. One image, many containers. |
| **Registry (ECR)** | Where images are stored so other machines can pull them. |
| **Pod** | The smallest unit Kubernetes runs. Usually one container. |
| **Deployment** | Tells Kubernetes "keep N copies of this image running at all times." |
| **Service** | A stable network address for a set of pods that keep being replaced. |
| **Ingress** | HTTP routing from the internet into your Services. |
| **EKS** | Amazon's managed Kubernetes. AWS runs the control plane, you run the workloads. |
| **Pipeline** | Automation that builds, tests, scans, and deploys your code on every commit. |

### The rules of this workshop

**Every lab is self-contained.** Each one creates what it needs and cleans up after itself. If you get lost in Module 07, you can start Module 08 from a clean slate.

**We show the failure before the fix.** Several modules deliberately break things. When something goes wrong on screen, it is usually on purpose. Do not read ahead — the surprise is the teaching.

**Everything costs money after Module 05.** From the moment the EKS cluster exists, AWS is billing you by the hour, whether or not you are using it. Module 13 tears everything down. Do not skip it.

---

## Lab 00 — Environment Verification

**Time:** 15 minutes
**Goal:** confirm every tool is installed, confirm AWS recognises you, and run the application natively so you experience the problem Docker solves.

### Step 1 — Verify your toolchain

Run each command. You are looking for a version number, not an error.

```bash
docker version --format '{{.Server.Version}}'
aws --version
eksctl version
kubectl version --client
helm version --short
trivy --version
git --version
python3 --version
```

Expected minimums:

| Tool | Minimum |
|---|---|
| Docker | 24.x |
| AWS CLI | 2.x |
| eksctl | 0.190+ |
| kubectl | 1.30+ |
| Helm | 3.x |
| Trivy | 0.50+ |
| Python | 3.10+ |

If `docker version` reports a client version but errors on the server, the Docker daemon is not running. Start Docker Desktop, or on Linux:

```bash
sudo systemctl start docker
sudo systemctl status docker --no-pager
```

### Step 2 — Verify your AWS identity

```bash
aws sts get-caller-identity
```

You should get back your Account, UserId, and Arn. Note your account ID — you will need it in Module 02 when you push to ECR.

Confirm your region is set:

```bash
aws configure get region
```

If that returns nothing, set one now:

```bash
aws configure set region ap-south-1
```

Use whichever region your instructor is using. Mixing regions between modules causes failures that look like permissions problems and are not.

If `get-caller-identity` fails, stop here and fix your credentials. Every module from 02 onwards depends on it.

### Step 3 — Clone the workshop repository

```bash
cd ~
git clone https://github.com/discover-devops/docker-to-eks-production-workshop.git
cd docker-to-eks-production-workshop
ls
```

### Step 4 — Create the application

We build the application by hand rather than just cloning it, because you should know exactly what is inside the thing you spend the next six hours deploying.

```bash
mkdir -p ~/catalog-app && cd ~/catalog-app
```

```bash
cat > app.py << 'EOF'
from flask import Flask, jsonify
import os, socket

app = Flask(__name__)

PRODUCTS = [
    {"id": 1, "name": "Wireless Earbuds",    "price": 2499,  "stock": 120},
    {"id": 2, "name": "Mechanical Keyboard", "price": 5999,  "stock": 34},
    {"id": 3, "name": "27 inch Monitor",     "price": 18999, "stock": 12},
]

@app.route("/health")
def health():
    return jsonify(status="ok"), 200

@app.route("/products")
def products():
    return jsonify(
        served_by=socket.gethostname(),
        version=os.getenv("APP_VERSION", "1.0.0"),
        products=PRODUCTS,
    )

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
EOF
```

```bash
cat > requirements.txt << 'EOF'
flask==3.0.3
gunicorn==22.0.0
EOF
```

### Step 5 — Run it natively

This is the last time in this workshop you will run the application directly on your operating system.

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python3 app.py
```

On Windows PowerShell, activate with `venv\Scripts\Activate.ps1` instead.

In a second terminal:

```bash
curl -s localhost:8080/health
curl -s localhost:8080/products
```

You should see the catalog come back, along with your own machine's hostname in the `served_by` field.

Stop the application with `Ctrl+C` in the first terminal.

### Step 6 — Understand what just happened

Look at what it took to get that response.

You needed Python 3 installed at the right version. You needed `venv` available. You needed a working network connection to download two packages from PyPI. You needed port 8080 free. And the result runs on your machine and nowhere else.

Now imagine handing `app.py` to a colleague and telling them to run it. They have Python 3.8. Or they are on Windows and the activation command is different. Or their corporate proxy blocks PyPI. Or port 8080 is taken by something else.

That gap — between "it runs here" and "it runs anywhere" — is the entire subject of Module 01.

### Step 7 — Clean up

```bash
deactivate
rm -rf ~/catalog-app/venv
ls ~/catalog-app
```

Keep `app.py` and `requirements.txt`. Module 01 uses both, unchanged.

---

## Checklist before Module 01

- [ ] All tools report a version
- [ ] Docker daemon is running
- [ ] `aws sts get-caller-identity` returns your account
- [ ] AWS region is set and matches the instructor's
- [ ] `app.py` and `requirements.txt` exist in `~/catalog-app`
- [ ] You have seen `/products` return a response

### If you are on an Apple Silicon Mac

Read this now, not in Module 06.

Docker on M-series Macs builds `arm64` images by default. EKS nodes in this workshop run `amd64`. An image built on your Mac will push to ECR successfully, deploy to Kubernetes successfully, and then crash-loop with an `exec format error` that gives you no useful clue about the cause.

You have two options. Either add `--platform linux/amd64` to every `docker build` from Module 01 onward, or run the entire workshop from a Linux EC2 instance. Choose now.

---

**Next:** [Module 01 — Docker Fundamentals](./01-docker-fundamentals.md)