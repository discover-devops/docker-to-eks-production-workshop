# Module 15 — Optimization and DevSecOps

**Duration:** 75 minutes
**You will finish this module with:** a pipeline that only builds what changed, cancels superseded runs, requires human approval before production, scans for secrets and code flaws, pins its own dependencies, and runs with least-privilege access to both AWS and the cluster.

---

## Context

The Module 14 pipeline works. It also has five problems that only appear once a team is actually using it.

**It is slow and it is slow for no reason.** Every push runs everything. A README change costs the same fourteen minutes as a code change.

**Concurrent pushes race each other.** Three commits in five minutes means three pipelines deploying to the same cluster, and whichever finishes last wins. That may not be the newest commit.

**Nothing stands between a commit and production.** A merge deploys. There is no moment where a human looks at what is about to ship.

**The CI role is cluster admin.** It can delete every node in the cluster. It needs to patch two Deployments.

**The pipeline trusts things it should verify.** Marketplace actions are pinned to moving tags. Nothing checks for committed secrets. Nothing runs static analysis on the code itself.

This module fixes all five.

![Pipeline optimization](./diagrams/svg/24-pipeline-optimization.svg)

<sub>Editable source: [`24-pipeline-optimization.excalidraw`](./diagrams/excalidraw/24-pipeline-optimization.excalidraw)</sub>

---

## Concept

### Change detection

Path filters at the workflow level are all-or-nothing: the workflow runs or it does not.

Change detection is finer. One small job inspects which paths changed and outputs booleans, and later jobs use those to decide whether to run. In a repository with two services, a change to `catalog-api` should not rebuild `orders-api`.

This is the same instinct as Dockerfile layer ordering from Module 06: do not redo work whose inputs did not change.

### Caching, three kinds

**Dependency cache** — `actions/setup-python` with `cache: pip` keyed on the requirements file.

**Docker layer cache** — `type=gha` stores buildx layers in the Actions cache. Scope it per service or they overwrite each other.

**Registry cache** — `type=registry` stores layers in ECR itself. Survives longer than the Actions cache and is shared across branches.

Actions cache has a per-repository size limit and evicts least-recently-used entries, so a busy repository can lose its cache. Registry cache does not have that problem.

### Concurrency groups

```yaml
concurrency:
  group: deploy-${{ github.ref }}
  cancel-in-progress: true
```

Runs sharing a group never run simultaneously. With `cancel-in-progress`, a new run cancels the older one.

For deployment this is important for correctness, not speed. Without it, two runs can reach `kubectl set image` in an unpredictable order, and the cluster can end up on an older commit than the newest one pushed.

Be careful with the value. For deploys, cancelling an in-flight run is usually right. For a release that publishes artifacts, cancelling halfway can leave things inconsistent — there you want `cancel-in-progress: false` so runs queue instead.

### Environments

A GitHub Environment is a named deployment target with its own protections and secrets.

**Required reviewers** pause the job until a named person approves. The job sits waiting, and the approval is recorded.

**Wait timers** enforce a delay before deployment.

**Deployment branches** restrict which branches may deploy to that environment.

**Environment secrets** override repository secrets, so staging and production use different credentials — including different IAM role ARNs.

This is how you get separate staging and production from one workflow file without duplicating it.

### Reusable workflows and composite actions

A **reusable workflow** is a complete workflow that other workflows call with `uses:`. It has its own jobs and runners. Use it for a whole pipeline stage shared across repositories.

A **composite action** bundles several steps into one. It runs inside the calling job. Use it for a sequence you repeat within a workflow.

The rule: if it needs its own runner, make it a reusable workflow. If it is a sequence of steps, make it a composite action.

### GITHUB_TOKEN permissions

Every run gets an automatic token. Depending on repository settings it may default to read and write on everything.

Set it explicitly at the workflow level and raise it only in the jobs that need it:

```yaml
permissions:
  contents: read
```

If a compromised action runs in your job, this is what limits what it can do.

### Pinning actions by SHA

`uses: actions/checkout@v4` follows a tag. Tags move. If an action's maintainer account is compromised, a malicious version can be published under an existing tag and every workflow using it picks it up on the next run.

This has happened, to popular actions, more than once.

Pinning to a full commit SHA removes that path. The cost is that updates need Dependabot to propose them, which is exactly what you want — a reviewed pull request rather than a silent change.

### Secret scanning

Committed credentials are among the most common causes of cloud compromise. Once a secret is in Git history, deleting the file does not remove it, exactly as deleting a file from a Docker layer does not remove it.

`gitleaks` scans the working tree and history for credential patterns. Run it in CI as a gate, and ideally as a pre-commit hook so it fires before the commit exists.

### SAST and dependency scanning

Trivy scans the **image**. It says nothing about your own code.

**SAST** — `bandit` for Python, `semgrep` for many languages — analyses source for insecure patterns: shell injection, hardcoded credentials, unsafe deserialization.

**Dependency scanning** — Dependabot or `pip-audit` — checks your declared dependencies against advisory databases and opens pull requests.

Both catch things Trivy cannot.

### Least privilege, twice

**On the AWS side**, the CI role should push to two named repositories and describe one cluster. It should not have `ecr:*` on `*`.

**On the Kubernetes side**, cluster admin lets the pipeline delete nodes. It needs to patch two Deployments in one namespace. EKS access policies can be scoped to a namespace, which is the fastest correct fix.

The question to ask about any pipeline credential: if this token leaked right now, what is the worst thing it could do?

![DevSecOps gates](./diagrams/svg/25-devsecops-gates.svg)

<sub>Editable source: [`25-devsecops-gates.excalidraw`](./diagrams/excalidraw/25-devsecops-gates.excalidraw)</sub>

### None of these replaces the others

Worth saying plainly, because people treat scanning as if it were security.

A scanner tells you what is known today about what you already built. Runtime hardening does not stop a vulnerable dependency being shipped. A code review does not catch a CVE published next Tuesday against a base image you shipped last month.

They are layers, and each one assumes the others will sometimes fail.

---

## Lab 15 — Optimize and Harden

**Time:** 55 minutes

Continues from Module 14. You need the `shop-cicd` repository and the EKS cluster.

### Step 1 — Baseline

```bash
cd ~/shop-cicd
export AWS_REGION=ap-south-1
export ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
export CLUSTER=workshop-eks
export ROLE_ARN=$(aws iam get-role --role-name github-actions-shop-cicd --query 'Role.Arn' --output text)
```

Open the Actions tab and note the duration of your last successful `Deploy to EKS` run.

| | |
|---|---|
| Baseline pipeline duration | _______ |

### Step 2 — Add a concurrency group

```bash
python3 - << 'PY'
p = ".github/workflows/deploy.yaml"
s = open(p).read()
s = s.replace("""permissions:
  contents: read
  id-token: write""", """concurrency:
  group: deploy-${{ github.ref }}
  cancel-in-progress: true

permissions:
  contents: read
  id-token: write""", 1)
open(p, "w").write(s)
print("concurrency added")
PY

head -20 .github/workflows/deploy.yaml
git add . && git commit -m "ci: add concurrency group" && git push
```

Prove it works — push twice in quick succession:

```bash
echo "# note 1" >> catalog-api/app.py
git add . && git commit -m "test: first push" && git push
sleep 10
sed -i 's/# note 1/# note 2/' catalog-api/app.py
git add . && git commit -m "test: second push" && git push
```

**Point out:** the first run is cancelled. Only the newest commit reaches the cluster.

```bash
sed -i '/# note 2/d' catalog-api/app.py
git add . && git commit -m "chore: remove test comment" && git push
```

### Step 3 — Add change detection

```bash
cat > .github/workflows/optimized.yaml << 'EOF'
name: Optimized pipeline

on:
  push:
    branches: [main]
  workflow_dispatch:

concurrency:
  group: optimized-${{ github.ref }}
  cancel-in-progress: true

permissions:
  contents: read
  id-token: write

jobs:
  changes:
    name: Detect changes
    runs-on: ubuntu-latest
    outputs:
      catalog: ${{ steps.filter.outputs.catalog }}
      orders: ${{ steps.filter.outputs.orders }}
      manifests: ${{ steps.filter.outputs.manifests }}
    steps:
      - uses: actions/checkout@v4

      - name: Filter paths
        id: filter
        uses: dorny/paths-filter@v3
        with:
          filters: |
            catalog:
              - 'catalog-api/**'
            orders:
              - 'orders-api/**'
            manifests:
              - 'k8s/**'

      - name: Report
        run: |
          echo "catalog changed  : ${{ steps.filter.outputs.catalog }}"
          echo "orders changed   : ${{ steps.filter.outputs.orders }}"
          echo "manifests changed: ${{ steps.filter.outputs.manifests }}"

  test-catalog:
    name: Test catalog-api
    needs: changes
    if: needs.changes.outputs.catalog == 'true'
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
          cache: pip
          cache-dependency-path: catalog-api/requirements.txt
      - working-directory: catalog-api
        run: |
          pip install -r requirements.txt -r requirements-dev.txt
          flake8 app.py --max-line-length=120
          pytest tests/ -v

  test-orders:
    name: Test orders-api
    needs: changes
    if: needs.changes.outputs.orders == 'true'
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
          cache: pip
          cache-dependency-path: orders-api/requirements.txt
      - working-directory: orders-api
        run: |
          pip install -r requirements.txt -r requirements-dev.txt
          flake8 app.py --max-line-length=120
          pytest tests/ -v

  security:
    name: Security scans
    needs: changes
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Secret scan
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        continue-on-error: true

      - name: Install SAST tools
        run: pip install bandit==1.7.9 pip-audit==2.7.3

      - name: Static analysis
        run: |
          bandit -r catalog-api/app.py orders-api/app.py -f txt || true

      - name: Dependency audit
        run: |
          pip-audit -r catalog-api/requirements.txt --desc || true
          pip-audit -r orders-api/requirements.txt --desc || true

  summary:
    name: Summary
    needs: [changes, test-catalog, test-orders, security]
    if: always()
    runs-on: ubuntu-latest
    steps:
      - run: |
          {
            echo "## What ran"
            echo ""
            echo "| Job | Result |"
            echo "|---|---|"
            echo "| catalog tests | ${{ needs.test-catalog.result }} |"
            echo "| orders tests | ${{ needs.test-orders.result }} |"
            echo "| security | ${{ needs.security.result }} |"
          } >> $GITHUB_STEP_SUMMARY
EOF

git add . && git commit -m "ci: change detection, timeouts and security scans" && git push
```

Now change only one service:

```bash
sed -i 's/"stock": 120/"stock": 150/' catalog-api/app.py
git add . && git commit -m "feat: update catalog stock" && git push
```

**Point out:** `Test orders-api` shows as skipped. It did not run and did not consume minutes.

Then change only docs:

```bash
echo "" >> README.md && echo "Notes." >> README.md
git add . && git commit -m "docs: notes" && git push
```

Both test jobs skipped.

### Step 4 — Prove the secret scanner works

```bash
cat > /tmp/leak-test.py << 'EOF'
AWS_ACCESS_KEY_ID = "AKIAIOSFODNN7EXAMPLE"
AWS_SECRET_ACCESS_KEY = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
EOF
cp /tmp/leak-test.py catalog-api/config_bad.py
git add . && git commit -m "test: deliberate secret" && git push
```

Watch the `Security scans` job flag it.

**Point out:** even after you delete the file, the secret is in Git history — exactly like the Docker layer problem in Module 07. The only real remediation is rotating the credential.

```bash
git rm catalog-api/config_bad.py
git commit -m "revert: remove test secret" && git push
```

### Step 5 — Create GitHub Environments

GitHub → repo → **Settings** → **Environments** → **New environment**.

**Environment 1:** name `staging`. No protection rules.

**Environment 2:** name `production`.
- Tick **Required reviewers** and add yourself
- Tick **Wait timer** and set 0 minutes
- Under **Deployment branches**, select **Selected branches** and add `main`

**Point out:** production now needs a named human, and only `main` can deploy to it.

### Step 6 — Add the approval gate to the deploy workflow

```bash
python3 - << 'PY'
p = ".github/workflows/deploy.yaml"
s = open(p).read()
s = s.replace("""  deploy:
    name: Deploy to EKS
    runs-on: ubuntu-latest
    needs: build-scan-push""", """  deploy:
    name: Deploy to EKS
    runs-on: ubuntu-latest
    needs: build-scan-push
    environment:
      name: production
    timeout-minutes: 15""", 1)
open(p, "w").write(s)
print("environment added to deploy job")
PY

grep -A 6 "deploy:" .github/workflows/deploy.yaml | head -10
git add . && git commit -m "ci: require approval before production deploy" && git push
```

Watch the run. **The deploy job pauses** with a Review deployments button. Nothing has reached the cluster.

Click **Review deployments** → tick production → **Approve and deploy**.

**Point out:** the approval is recorded against your account, with a timestamp, permanently attached to that run.

### Step 7 — Registry-based build cache

```bash
python3 - << 'PY'
p = ".github/workflows/deploy.yaml"
s = open(p).read()
s = s.replace("""          cache-from: type=gha,scope=${{ matrix.service }}
          cache-to: type=gha,mode=max,scope=${{ matrix.service }}""",
"""          cache-from: |
            type=gha,scope=${{ matrix.service }}
            type=registry,ref=${{ vars.ECR_REGISTRY }}/${{ matrix.service }}:buildcache
          cache-to: |
            type=gha,mode=max,scope=${{ matrix.service }}""", 1)
open(p, "w").write(s)
print("registry cache added as a fallback source")
PY

git add . && git commit -m "ci: add registry cache fallback" && git push
```

The Actions cache is fast but evicts under pressure. The registry cache is a durable fallback.

### Step 8 — Pin actions by commit SHA

```bash
grep -rn "uses:" .github/workflows/ | grep -v "^\s*#" | sort -u
```

Every one of those is third-party code with access to your job. Pin the ones that touch credentials:

```bash
python3 - << 'PY'
import glob, re

# verified SHAs at time of writing - re-verify before you rely on these
pins = {
    "actions/checkout@v4": "actions/checkout@692973e3d937129bcbf40652eb9f2f61becf3332 # v4.1.7",
    "actions/setup-python@v5": "actions/setup-python@39cd14951b08e74b54015e9e001cdefcf80e669f # v5.1.1",
    "aws-actions/configure-aws-credentials@v4": "aws-actions/configure-aws-credentials@e3dd6a429d7300a6a4c196c26e071d42e0343502 # v4.0.2",
}

for f in glob.glob(".github/workflows/*.yaml"):
    s = open(f).read()
    orig = s
    for old, new in pins.items():
        s = s.replace("uses: " + old, "uses: " + new)
    if s != orig:
        open(f, "w").write(s)
        print("pinned:", f)
PY

grep -rn "actions/checkout" .github/workflows/ | head -3
```

**Verify those SHAs yourself before trusting them.** On each action's GitHub releases page, find the tag and copy the commit SHA it points at. Do not paste a SHA you have not checked.

```bash
git add . && git commit -m "ci: pin credential-touching actions by SHA" && git push
```

### Step 9 — Add Dependabot

Pinning by SHA means nothing updates unless something proposes updates.

```bash
mkdir -p .github

cat > .github/dependabot.yaml << 'EOF'
version: 2
updates:
  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: weekly
    open-pull-requests-limit: 5
    commit-message:
      prefix: "ci"

  - package-ecosystem: pip
    directory: /catalog-api
    schedule:
      interval: weekly
    open-pull-requests-limit: 5

  - package-ecosystem: pip
    directory: /orders-api
    schedule:
      interval: weekly
    open-pull-requests-limit: 5

  - package-ecosystem: docker
    directory: /catalog-api
    schedule:
      interval: weekly

  - package-ecosystem: docker
    directory: /orders-api
    schedule:
      interval: weekly
EOF

git add . && git commit -m "ci: dependabot for actions, pip and docker" && git push
```

Check **Insights** → **Dependency graph** → **Dependabot** to confirm it is enabled.

**Point out:** updates now arrive as reviewable pull requests that must pass your pipeline, rather than as silent changes.

### Step 10 — Least privilege in the cluster

The CI role is currently cluster admin. Fix it.

```bash
aws eks list-associated-access-policies \
  --cluster-name $CLUSTER --region $AWS_REGION \
  --principal-arn $ROLE_ARN --output table
```

Remove admin and grant namespace-scoped edit:

```bash
aws eks disassociate-access-policy \
  --cluster-name $CLUSTER --region $AWS_REGION \
  --principal-arn $ROLE_ARN \
  --policy-arn arn:aws:eks::aws:cluster-access-policy/AmazonEKSClusterAdminPolicy

aws eks associate-access-policy \
  --cluster-name $CLUSTER --region $AWS_REGION \
  --principal-arn $ROLE_ARN \
  --policy-arn arn:aws:eks::aws:cluster-access-policy/AmazonEKSEditPolicy \
  --access-scope type=namespace,namespaces=default

aws eks list-associated-access-policies \
  --cluster-name $CLUSTER --region $AWS_REGION \
  --principal-arn $ROLE_ARN --output table
```

Now trigger a deploy and confirm it still works:

```bash
cd ~/shop-cicd
sed -i 's/"stock": 150/"stock": 175/' catalog-api/app.py
git add . && git commit -m "feat: stock update to test scoped permissions" && git push
```

Approve the deployment when prompted. It should succeed.

**Point out:** the pipeline can still patch Deployments in `default`. It can no longer delete nodes, read secrets in `kube-system`, or touch other namespaces.

Ask the question out loud: if this role's credentials leaked right now, what is the worst an attacker could do? The answer is now much shorter.

### Step 11 — Tighten the AWS policy too

```bash
aws iam get-role-policy --role-name github-actions-shop-cicd \
  --policy-name shop-cicd-ci-policy --query 'PolicyDocument.Statement[].Sid' --output text
```

The policy is already scoped to two ECR repositories and one cluster. Verify what it cannot do:

```bash
aws iam simulate-principal-policy \
  --policy-source-arn $ROLE_ARN \
  --action-names ecr:DeleteRepository eks:DeleteCluster ec2:TerminateInstances \
  --query 'EvaluationResults[].{Action:EvalActionName,Decision:EvalDecision}' \
  --output table
```

All denied.

### Step 12 — Set GITHUB_TOKEN to read-only by default

GitHub → **Settings** → **Actions** → **General** → **Workflow permissions** → select **Read repository contents and packages permissions** → **Save**.

Your workflows already declare their own permissions explicitly, so nothing breaks.

**Point out:** if a compromised marketplace action runs in your pipeline, this is what stops it pushing commits or opening releases.

### Step 13 — Add a composite action

Three workflows repeat the same AWS setup. Factor it out.

```bash
mkdir -p .github/actions/aws-login

cat > .github/actions/aws-login/action.yaml << 'EOF'
name: AWS login
description: Assume the CI role via OIDC and log in to ECR

inputs:
  role-arn:
    description: IAM role to assume
    required: true
  region:
    description: AWS region
    required: true

runs:
  using: composite
  steps:
    - name: Configure credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        role-to-assume: ${{ inputs.role-arn }}
        aws-region: ${{ inputs.region }}
        role-session-name: gha-${{ github.run_id }}

    - name: Log in to ECR
      uses: aws-actions/amazon-ecr-login@v2

    - name: Show identity
      shell: bash
      run: aws sts get-caller-identity
EOF
```

Use it:

```bash
cat > .github/workflows/composite-demo.yaml << 'EOF'
name: Composite action demo

on:
  workflow_dispatch:

permissions:
  contents: read
  id-token: write

jobs:
  demo:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: AWS login
        uses: ./.github/actions/aws-login
        with:
          role-arn: ${{ vars.AWS_ROLE_ARN }}
          region: ${{ vars.AWS_REGION }}

      - name: Use the credentials
        run: |
          aws ecr describe-repositories --query 'repositories[].repositoryName' --output text
EOF

git add . && git commit -m "ci: composite action for aws login" && git push
```

Run it from the Actions tab. Three steps became one.

### Step 14 — Measure the improvement

Push a code change and time it:

```bash
sed -i 's/"stock": 175/"stock": 200/' catalog-api/app.py
git add . && git commit -m "feat: final stock update" && git push
```

| | |
|---|---|
| Baseline (Step 1) | _______ |
| Optimized, code change | _______ |
| Optimized, docs-only change | _______ |

Then push a docs-only change and confirm almost nothing runs.

### Step 15 — What a production setup would add

Honest list of what this workshop does not cover.

| Practice | What it adds |
|---|---|
| GitOps with Argo CD or Flux | The cluster pulls from Git rather than CI pushing. Drift becomes visible. |
| Progressive delivery with Argo Rollouts | Canary with automatic rollback on metrics |
| Policy as code with Kyverno or Gatekeeper | Admission-time enforcement of image sources and pod rules |
| Signed images with cosign | Verify the image was built by your pipeline, not substituted |
| Provenance attestations | Cryptographic record of how an artifact was produced |
| Multiple environments | Separate clusters and IAM roles per stage |
| Self-hosted runners in a VPC | Reach private endpoints, at the cost of securing the runner |
| Terraform for the infrastructure | The cluster itself in version control |

### Step 16 — Teardown

Keep everything for Module 16, which is the final demonstration.

---

## Troubleshooting

**`dorny/paths-filter` outputs are always false.** For push events it compares against the previous commit. On the very first run there is nothing to compare, so it may behave unexpectedly. Push twice.

**A skipped job makes a downstream job skip too.** `needs` on a skipped job propagates. Use `if: always() && needs.x.result != 'failure'` where you want a job to run anyway.

**Concurrency cancels a run you wanted to finish.** Set `cancel-in-progress: false` for release-style workflows.

**The environment approval never appears.** Confirm the job has `environment: name: production` and that required reviewers are configured on that environment.

**Deploy fails with Forbidden after scoping RBAC.** The access policy scope does not include the namespace you are deploying to. Check with `aws eks list-associated-access-policies`.

**gitleaks fails on a false positive.** Add a `.gitleaksignore` file with the fingerprint, and review the entry rather than disabling the scan.

**A pinned action stops working.** The SHA is wrong or the action was deleted. Verify on the action's releases page.

---

## What you built

| Improvement | Where |
|---|---|
| Superseded runs cancelled | Step 2 |
| Only changed services rebuild | Step 3 |
| Secret scanning on every push | Step 4 |
| Human approval before production | Step 6 |
| Durable registry build cache | Step 7 |
| Supply chain actions pinned by SHA | Step 8 |
| Automated dependency updates | Step 9 |
| Cluster access scoped to one namespace | Step 10 |
| AWS permissions verified as denied | Step 11 |
| Read-only default token | Step 12 |
| Reusable AWS login step | Step 13 |

## The question to keep asking

For every credential in a pipeline: **if this leaked right now, what is the worst it could do?**

At the start of Module 14 the answer was "nothing, because there is no credential." At the start of this module the answer for the cluster was "delete every node." Now it is "patch two Deployments in one namespace."

That is what least privilege actually means in practice.

---

**Next:** [Module 16 — Final Production Demo](./16-final-production-demo.md)
