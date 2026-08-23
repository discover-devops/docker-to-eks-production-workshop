# Module 00 — Workshop Kickoff

**Duration:** 30 minutes
**You will finish this module with:** a verified toolchain, a working AWS identity, and both workshop applications running on your own machine.

---

## Context

Open Flipkart and search for a laptop.

What comes back looks like one page, but it is not one application. The product listing came from a catalog service. The price and stock came from an inventory service. "Deliver by Tuesday" came from a logistics service. Your order history came from an orders service. Recommendations came from somewhere else entirely.

Dozens of separate applications, written by different teams, deployed independently, talking to each other over HTTP — and the whole thing has to survive Big Billion Days, when traffic goes up twenty times in a single evening and comes back down six hours later.

That is the destination. This workshop is the road there.

### How we are going to get there

We are not going to start with Kubernetes. We are going to start where the industry actually started, and move forward the same way it did — one problem at a time.

**First**, you will deploy a two-service application the traditional way: two EC2 instances, SSH in, install the runtime, copy the code, start the process, write a systemd unit. This is how production worked for most of the last two decades, and a great deal of it still works this way today.

**Then** you will put an Application Load Balancer in front, with path-based routing sending `/catalog` traffic to one service and `/orders` to the other. At this point you will have a genuinely working, publicly reachable microservice deployment.

**Then** we will break it. Not artificially — we will simply look honestly at what it costs to run. Deployments are manual. Servers drift apart from each other. Scaling means baking a new AMI and waiting minutes. An instance dies and nobody notices until a customer complains.

**That pain is what Docker solves**, so Docker is where we go next. We will containerize the exact same two services, harden the images to production standards, scan them for vulnerabilities, and push them to a registry.

**Then we will break that too.** Docker on a single host fixes packaging, but it does not restart a crashed container, does not spread load across machines, and does not help one service find another.

**That pain is what Kubernetes solves.** So we finish on EKS — the same two services, running as Deployments, exposed by Services, routed by an Ingress that lands us right back at an Application Load Balancer, except this time nobody configured it by hand.

Every tool in this workshop is introduced as the answer to a problem you have already felt. Nothing appears because it is on a syllabus.

---

## Concept

### The application

Two small services, deliberately shaped like the ones behind any e-commerce platform.

**`catalog-api`** — owns product data.

| Endpoint | Returns |
|---|---|
| `GET /health` | Liveness check |
| `GET /products` | All products, plus the hostname serving the request |
| `GET /products/<id>` | A single product |

**`orders-api`** — owns orders, and calls `catalog-api` to enrich them.

| Endpoint | Returns |
|---|---|
| `GET /health` | Liveness check |
| `GET /orders` | Orders with product names resolved from `catalog-api` |

That second service matters more than it looks. Because `orders-api` has to *find* `catalog-api` over the network, every stage of this workshop has to answer the question "how does one service locate another?" — and the answer changes each time.

| Stage | How `orders-api` finds `catalog-api` |
|---|---|
| EC2 | A hardcoded private IP address |
| Docker | A container name on a Docker network |
| Kubernetes | A Service DNS name that survives pods being replaced |

That progression is the spine of the workshop.

### The `served_by` field

Both services return the hostname of whatever is handling the request. This is our evidence, not decoration:

- In Module 03 it proves the load balancer is distributing traffic.
- In Module 10 it proves Kubernetes is running multiple replicas.
- In Module 12 it proves the Ingress is routing to the right service.

Watch that field throughout.

### The road map

| Module | Topic |
|---|---|
| 00 | Workshop kickoff — environment and applications |
| 01 | AWS networking recap — the ground everything sits on |
| 02 | EC2 hosting — deploy both services by hand |
| 03 | Load balancing — ALB, NLB, and path-based routing |
| 04 | The limits of EC2 — why this does not scale |
| 05 | Docker fundamentals — images, containers, architecture |
| 06 | Containerizing the application |
| 07 | Image hardening — multi-stage, non-root, Trivy, ECR |
| 08 | The limits of Docker on a host |
| 09 | Kubernetes concepts and launching EKS |
| 10 | Deployments |
| 11 | Services and service discovery |
| 12 | Ingress and HTTPS |

Modules 13 onward cover GitHub Actions, secrets, and DevSecOps in a later session.

### Vocabulary

Read once now. Each becomes concrete when you meet it.

| Term | Working definition |
|---|---|
| **AMI** | A snapshot of a disk used to launch EC2 instances |
| **Target group** | The set of servers a load balancer sends traffic to |
| **Image** | A packaged, read-only bundle of an app and its dependencies |
| **Container** | A running instance of an image |
| **Registry (ECR)** | Where images are stored so other machines can pull them |
| **Pod** | The smallest unit Kubernetes runs — usually one container |
| **Deployment** | "Keep N copies of this image running at all times" |
| **Service** | A stable network name for a changing set of pods |
| **Ingress** | HTTP routing from the internet into Services |
| **EKS** | Amazon's managed Kubernetes — AWS runs the control plane |

### The rules of this workshop

**Every lab is self-contained.** Each lab creates what it needs from scratch and removes it at the end. If you fall behind in Module 06, you can still start Module 07 cleanly.

**We show the failure before the fix.** Several modules deliberately break things. When something fails on screen, it is usually on purpose.

**Everything costs money from Module 02 onward.** EC2 instances, NAT gateways, load balancers, and the EKS control plane all bill by the hour whether you are using them or not. Every lab ends with teardown. Do not skip it.

---

## Lab 00 — Environment Verification

**Time:** 20 minutes
**Goal:** confirm your tools work, confirm AWS recognises you, and run both services locally so you know what "working" looks like before anything gets complicated.

### Step 1 — Verify your toolchain

Each command should return a version, not an error.

```bash
aws --version
docker version --format '{{.Server.Version}}'
eksctl version
kubectl version --client
helm version --short
trivy --version
git --version
python3 --version
```

Minimums:

| Tool | Minimum |
|---|---|
| AWS CLI | 2.x |
| Docker | 24.x |
| eksctl | 0.190+ |
| kubectl | 1.30+ |
| Helm | 3.x |
| Trivy | 0.50+ |
| Python | 3.10+ |

If `docker version` returns a client version but errors on the server, the daemon is not running:

```bash
sudo systemctl start docker     # Linux
```

On macOS or Windows, start Docker Desktop.

Docker is not needed until Module 05, but fix it now rather than mid-session.

### Step 2 — Verify your AWS identity

```bash
aws sts get-caller-identity
```

You should see your Account, UserId, and Arn. **Note your 12-digit account ID** — Module 07 needs it for ECR.

```bash
aws configure get region
```

If that returns nothing:

```bash
aws configure set region ap-south-1
```

Use whatever region your instructor is using. Mixing regions across modules produces failures that look like permission errors and are not.

If `get-caller-identity` fails, stop and fix your credentials. Everything from Module 02 depends on it.

### Step 3 — Create a clean working directory

```bash
rm -rf ~/workshop-app
mkdir -p ~/workshop-app/catalog-api ~/workshop-app/orders-api
cd ~/workshop-app
```

### Step 4 — Create `catalog-api`

```bash
cat > ~/workshop-app/catalog-api/app.py << 'EOF'
from flask import Flask, jsonify
import os, socket

app = Flask(__name__)

SERVICE_NAME = "catalog-api"
VERSION = os.getenv("APP_VERSION", "1.0.0")

PRODUCTS = {
    1: {"id": 1, "name": "Wireless Earbuds",    "price": 2499,  "stock": 120},
    2: {"id": 2, "name": "Mechanical Keyboard", "price": 5999,  "stock": 34},
    3: {"id": 3, "name": "27 inch Monitor",     "price": 18999, "stock": 12},
    4: {"id": 4, "name": "USB-C Hub",           "price": 3499,  "stock": 87},
}

def meta():
    return {
        "service": SERVICE_NAME,
        "version": VERSION,
        "served_by": socket.gethostname(),
    }

@app.route("/health")
def health():
    return jsonify(status="ok", **meta()), 200

@app.route("/products")
def products():
    return jsonify(products=list(PRODUCTS.values()), **meta())

@app.route("/products/<int:pid>")
def product(pid):
    p = PRODUCTS.get(pid)
    if not p:
        return jsonify(error="not found", **meta()), 404
    return jsonify(product=p, **meta())

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
EOF
```

```bash
cat > ~/workshop-app/catalog-api/requirements.txt << 'EOF'
flask==3.0.3
gunicorn==22.0.0
EOF
```

### Step 5 — Create `orders-api`

Note the `CATALOG_URL` environment variable. That one line is how this service finds the other, and it is the value that changes at every stage of the workshop.

```bash
cat > ~/workshop-app/orders-api/app.py << 'EOF'
from flask import Flask, jsonify
import os, socket, requests

app = Flask(__name__)

SERVICE_NAME = "orders-api"
VERSION = os.getenv("APP_VERSION", "1.0.0")
CATALOG_URL = os.getenv("CATALOG_URL", "http://localhost:8080")

ORDERS = [
    {"order_id": "ORD-1001", "product_id": 1, "qty": 2, "status": "DELIVERED"},
    {"order_id": "ORD-1002", "product_id": 3, "qty": 1, "status": "IN_TRANSIT"},
    {"order_id": "ORD-1003", "product_id": 2, "qty": 1, "status": "PLACED"},
]

def meta():
    return {
        "service": SERVICE_NAME,
        "version": VERSION,
        "served_by": socket.gethostname(),
        "catalog_url": CATALOG_URL,
    }

@app.route("/health")
def health():
    return jsonify(status="ok", **meta()), 200

@app.route("/orders")
def orders():
    enriched = []
    for o in ORDERS:
        item = dict(o)
        try:
            r = requests.get(
                f"{CATALOG_URL}/products/{o['product_id']}", timeout=2
            )
            if r.status_code == 200:
                p = r.json()["product"]
                item["product_name"] = p["name"]
                item["unit_price"] = p["price"]
                item["line_total"] = p["price"] * o["qty"]
            else:
                item["product_name"] = "UNKNOWN"
        except Exception as e:
            item["product_name"] = "CATALOG_UNAVAILABLE"
            item["error"] = str(e.__class__.__name__)
        enriched.append(item)
    return jsonify(orders=enriched, **meta())

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8081)
EOF
```

```bash
cat > ~/workshop-app/orders-api/requirements.txt << 'EOF'
flask==3.0.3
gunicorn==22.0.0
requests==2.32.3
EOF
```

### Step 6 — Run both services

**Terminal 1 — catalog-api on port 8080:**

```bash
cd ~/workshop-app/catalog-api
python3 -m venv venv && source venv/bin/activate
pip install -q -r requirements.txt
python3 app.py
```

**Terminal 2 — orders-api on port 8081:**

```bash
cd ~/workshop-app/orders-api
python3 -m venv venv && source venv/bin/activate
pip install -q -r requirements.txt
python3 app.py
```

Windows PowerShell activates with `venv\Scripts\Activate.ps1`.

### Step 7 — Test both services

**Terminal 3:**

```bash
curl -s localhost:8080/health
curl -s localhost:8080/products
curl -s localhost:8081/orders
```

The `/orders` response should show product names and line totals pulled live from `catalog-api`.

### Step 8 — Watch the dependency break

Stop `catalog-api` with `Ctrl+C` in Terminal 1, then call orders again:

```bash
curl -s localhost:8081/orders
```

Product names now read `CATALOG_UNAVAILABLE`. The orders service is still up, but degraded, because the service it depends on is gone.

Restart `catalog-api` and confirm recovery:

```bash
curl -s localhost:8081/orders
```

Remember this behaviour. You will see it again in Module 03 when a target group goes unhealthy, and in Module 10 when a pod is deleted — except by then, something will be fixing it for you automatically.

### Step 9 — Notice what this took

To get two services talking on one machine, you needed the right Python version, two virtualenvs, three packages downloaded from the internet, two free ports, and three terminals.

Now imagine doing that on a server you have never logged into, at 11 PM, over SSH, with a colleague reading the steps to you over a call.

That is Module 02.

### Step 10 — Clean up

Stop both services with `Ctrl+C`, then:

```bash
deactivate 2>/dev/null
rm -rf ~/workshop-app/catalog-api/venv ~/workshop-app/orders-api/venv
ls -R ~/workshop-app
```

**Keep `~/workshop-app`.** Both `app.py` files and both `requirements.txt` files are used unchanged for the rest of the workshop. Nothing about the application code changes — only where and how it runs.

---

## Checklist before Module 01

- [ ] All tools report a version
- [ ] `aws sts get-caller-identity` returns your account
- [ ] Account ID noted down
- [ ] Region set and matching the instructor's
- [ ] Both services ran locally
- [ ] `/orders` returned product names from `catalog-api`
- [ ] You saw `CATALOG_UNAVAILABLE` when catalog was stopped

### If you are on an Apple Silicon Mac

Read this now, not in Module 10.

Docker on M-series Macs builds `arm64` images by default. The EKS nodes in this workshop run `amd64`. An `arm64` image will push to ECR successfully, deploy to Kubernetes successfully, and then crash-loop with `exec format error` and no useful clue as to why.

Either add `--platform linux/amd64` to every `docker build` from Module 06 onward, or run the entire workshop from a Linux EC2 instance. Decide now.

---

**Next:** [Module 01 — AWS Networking Recap](./01-aws-networking-recap.md)