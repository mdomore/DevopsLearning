# Phase 0 — Tooling setup on macOS

Goal: install course tools on **macOS**.  
Commands run in your **local terminal**.

Index: [00-tooling-setup.md](00-tooling-setup.md)

---

## Step 1 — Install Homebrew (package manager)

### Explanation

Homebrew simplifies installing CLI tools on Mac.  
If you already have `brew`, skip to Step 2.

### Implementation

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Follow the installer instructions. It may ask you to add `brew` to your shell `PATH`.

### Verification

```bash
brew --version
```

---

## Step 2 — Install Docker

### Explanation

Two common options:

- **Docker Desktop** (recommended for beginners): GUI app + Docker CLI
- **Colima + Docker CLI**: lighter CLI-only alternative

Pick one. Do not install both at the same time.

### Implementation — Option A: Docker Desktop (recommended)

1. Download: https://www.docker.com/products/docker-desktop/
2. Install and open Docker Desktop.
3. Wait until the app shows **Docker is running**.

### Implementation — Option B: Colima + Docker CLI

```bash
brew install colima docker docker-compose
colima start
```

### Verification

```bash
docker version
docker run --rm hello-world
```

Expected: both client and server sections appear in `docker version`, and `hello-world` prints a success message.

---

## Step 3 — Install kubectl

### Explanation

`kubectl` sends requests to the Kubernetes API.  
You install the client now; it connects to a cluster in Step 5.

### Implementation

```bash
brew install kubectl
```

### Verification

```bash
kubectl version --client
```

Expected: client version is printed (server may be unavailable until a cluster exists — that is normal).

---

## Step 4 — Local Kubernetes cluster

### Explanation

You need **one** working local cluster before Phase 2. Install **`kubectl`** in Step 3 first.

**Pick one option below** — do not run both unless you know how to switch contexts.

| Option | When to choose | kubectl context |
|---|---|---|
| **A — kind** (CLI) | You use **Colima**, or you prefer a dedicated lab cluster named `kube-lab` | `kind-kube-lab` |
| **B — Docker Desktop Kubernetes** (GUI) | You installed **Docker Desktop** in Step 2 — **recommended** | `docker-desktop` |

**Course note:** Lessons often mention context `kind-kube-lab`. If you chose **Option B**, use **`docker-desktop`** instead (same cluster, different name in kubeconfig).

---

### Option A — kind (CLI)

`kind` = **K**ubernetes **in** **D**ocker — cluster nodes run as Docker containers.

```bash
brew install kind
kind version

kind create cluster --name kube-lab
kubectl config use-context kind-kube-lab
kubectl get nodes
```

Expected: one node **Ready**, context **`kind-kube-lab`**.

**Cleanup:**

```bash
kind delete cluster --name kube-lab
kubectl config delete-context kind-kube-lab   # optional
```

**Break & repair:**

- `kind create cluster` fails → Docker must run (`docker version`); delete stale cluster: `kind delete cluster --name kube-lab`
- Wrong context → `kubectl config use-context kind-kube-lab`

---

### Option B — Docker Desktop Kubernetes (GUI)

Uses Kubernetes built into Docker Desktop — no `brew install kind` required.

**Requirements:** Docker Desktop **4.37+** (Create cluster UI ~**4.51+**), macOS **14+**, **8 GB RAM** recommended.

1. Open **Docker Desktop** → wait for **Docker is running**
2. **Kubernetes** → **Create cluster**
3. Settings:

| Setting | Value for this course |
|---|---|
| **Cluster type** | **kind** (not kubeadm) |
| **Node(s)** | **1** |
| **Kubernetes version** | **Default** (e.g. 1.34.x) |
| **Show system containers** | **Off** (optional; keeps `docker ps` clean) |

4. Click **Create** — first start downloads images (several minutes)

**If creation fails:** enable **containerd image store** in Docker Desktop → Settings → General, then retry.

**Changing node count or version later resets the cluster** — all resources are deleted.

Verify:

```bash
kubectl config use-context docker-desktop
kubectl get nodes
```

Expected: one node **Ready**, context **`docker-desktop`**.

**Cleanup:** Docker Desktop → Settings → Kubernetes → disable / remove cluster.

**Break & repair:**

- Connection refused → Kubernetes not running in Docker Desktop; recreate cluster
- Two `kubectl` binaries → `which -a kubectl` — Homebrew vs Docker Desktop; prefer one on your `PATH`

**Skip Option A** if you only use Option B.

---

## Step 5 — Verify cluster and namespace `kube-lab`

```bash
kubectl config current-context
kubectl get nodes
```

| Option | Expected context | Typical node name |
|---|---|---|
| A | `kind-kube-lab` | `kube-lab-control-plane` |
| B | `docker-desktop` | `desktop-control-plane` |

**What matters:** one node, `STATUS` **Ready**, `ROLES` includes `control-plane`.  
The **node name** and **INTERNAL-IP** differ between options — that is normal, not an error.

Example (Option B — Docker Desktop):

```text
NAME                    STATUS   ROLES           VERSION
desktop-control-plane   Ready    control-plane   v1.34.3
```

Prepare the course sandbox (same for both options):

```bash
kubectl create namespace kube-lab
kubectl config set-context --current --namespace=kube-lab
kubectl auth can-i create pods --namespace=kube-lab
```

Expected: `yes`.

Full undo: [00-tooling-setup-cleanup.md](00-tooling-setup-cleanup.md)

---

## Step 6 — Install Terraform

### Explanation

Terraform provisions cloud infrastructure from `.tf` files (used in later chapters).

### Implementation

```bash
brew install terraform
```

### Verification

```bash
terraform version
```

---

## Step 7 — Install and configure AWS CLI

### Explanation

The AWS CLI talks to your AWS account from the terminal.

A **profile** is a named set of settings (credentials + region). Think of it like saved connections:

| Profile | When it is used |
|---|---|
| **default** | When you run `aws ...` with no extra flag |
| **lab** (course) | When you run `aws ... --profile lab` |

**Important:** Adding a `lab` profile does **not** change or delete your existing setup.  
Profiles live side by side in two files in your home directory:

- `~/.aws/credentials` — access keys per profile
- `~/.aws/config` — region and output format per profile

If your command without `--profile` already works:

```bash
aws sts get-caller-identity
```

That uses the **default** profile. Other AWS work you already run keeps working the same way.

### Choose your path

**Path A — You already use AWS (recommended to read this)**  
You do **not** have to create a `lab` profile. You can use your existing **default** profile for this course.

Verification for Path A:

```bash
aws --version
aws sts get-caller-identity
```

When a later lab says `--profile lab`, either:
- run the same command **without** `--profile`, or
- replace `--profile lab` with `--profile default` (same effect if default is your main account).

**Path B — Separate lab profile (optional, safer for learning)**  
Create a dedicated profile so course commands never mix with production work.

1. **Create a lab IAM user** in AWS (console walkthrough): [00-aws-lab-iam-user.md](00-aws-lab-iam-user.md)
2. **Configure the CLI profile** with that user’s access keys (Step below)

This only **adds** a new section `[lab]` in your config files. It does not touch `[default]`.

### Implementation — Install CLI (if needed)

```bash
brew install awscli
```

Skip if `aws --version` already works.

### Implementation — Path B only: create `lab` profile

First complete: [Create an AWS lab IAM user](00-aws-lab-iam-user.md) (console steps + access keys).

Then run:

```bash
aws configure --profile lab
```

| Prompt | What to enter |
|---|---|
| AWS Access Key ID | From your **lab** IAM user (not required to be the same as default) |
| AWS Secret Access Key | From the same lab IAM user |
| Default region | e.g. `eu-west-1` |
| Default output format | `json` |

**Do not run** `aws configure` **without** `--profile` unless you intentionally want to overwrite your default profile.

If you have no AWS account yet, install the CLI now and configure before Phase 5/6.

### How to switch between profiles

**One command (safest — explicit):**

```bash
# Use default (your existing setup)
aws sts get-caller-identity

# Use lab profile
aws sts get-caller-identity --profile lab
```

**Whole terminal session (temporary):**

```bash
export AWS_PROFILE=lab          # all aws commands use lab
aws sts get-caller-identity

export AWS_PROFILE=default      # switch back to default
# or
unset AWS_PROFILE               # same as using default
```

**Check which profile is active in this shell:**

```bash
echo "${AWS_PROFILE:-default}"
```

Rule: `--profile` on a single command **overrides** `AWS_PROFILE` for that command only.

### Verification

Path A (existing default — your case):

```bash
aws --version
aws sts get-caller-identity
```

Path B (separate lab profile):

```bash
aws --version
aws sts get-caller-identity --profile lab
```

List configured profile names:

```bash
aws configure list-profiles
```

### Break & repair

**`The config profile (lab) could not be found`**  
→ Normal if you never created `lab`. Use Path A (default), or run `aws configure --profile lab`.

**Accidentally overwrote default credentials**  
→ Run `aws configure` (no profile flag) and re-enter your original keys, or restore from backup / IAM console new keys.

**Wrong account when running a command**  
→ Check identity: `aws sts get-caller-identity --profile <name>`  
→ Check active env: `echo "${AWS_PROFILE:-default}"`

**Course lab uses `--profile lab` but I use default**  
→ Drop `--profile lab` from the command, or set `export AWS_PROFILE=default`.

### Cleanup / revert (Step 7)

- **Remove local `lab` profile only:** edit `~/.aws/credentials` and `~/.aws/config` — delete the `[lab]` / `[profile lab]` blocks. Default profile stays unchanged.
- **Remove lab IAM user in AWS:** see [00-aws-lab-iam-user.md — Cleanup](00-aws-lab-iam-user.md#cleanup-when-you-finish-the-course).
- **Unset profile in current shell:** `unset AWS_PROFILE`

Full undo guide: [00-tooling-setup-cleanup.md](00-tooling-setup-cleanup.md)

---

## Cleanup / revert (Phase 0 overview)

| What | Revert |
|---|---|
| kind cluster `kube-lab` (Path A) | `kind delete cluster --name kube-lab` |
| Docker Desktop Kubernetes (Path B) | Disable in Docker Desktop → Settings → Kubernetes |
| kubectl context | `kubectl config delete-context kind-kube-lab` or switch away from `docker-desktop` |
| AWS profile `lab` (local) | Remove `[lab]` from `~/.aws/credentials` and `~/.aws/config` |
| Tools (optional) | `brew uninstall kind kubectl terraform awscli` — only if unused elsewhere |

Details: [00-tooling-setup-cleanup.md](00-tooling-setup-cleanup.md)

---

## Break & repair (macOS)

**`docker version` shows client but not server**  
→ Open Docker Desktop, or run `colima start`.

**`kubectl get nodes` connection refused**  
→ Option A: `kind create cluster --name kube-lab` or `kubectl config use-context kind-kube-lab`  
→ Option B: open Docker Desktop, ensure Kubernetes is running, `kubectl config use-context docker-desktop`

**`kind create cluster` fails**  
→ Fix Docker first; if name conflict: `kind delete cluster --name kube-lab`.

**AWS identity fails**  
→ See Step 7 break & repair (profile not found vs wrong keys).

---

## Phase 0 completion checklist (macOS)

- [ ] `docker version` (client + server)
- [ ] `kubectl version --client`
- [ ] `kubectl get nodes` → **Ready** (context `kind-kube-lab` or `docker-desktop`)
- [ ] Namespace `kube-lab` created; `kubectl auth can-i create pods --namespace=kube-lab` → **yes**
- [ ] `terraform version`
- [ ] `aws --version` and `aws sts get-caller-identity` (or `--profile lab`)

Next: [Phase 1 — Docker](../01-docker/01-docker-basics.md)
