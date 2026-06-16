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

## Step 4 — Install kind

### Explanation

`kind` = **K**ubernetes **in** **D**ocker.  
It creates a cluster as Docker containers — free, fast, easy to delete.

### Implementation

```bash
brew install kind
```

### Verification

```bash
kind version
```

---

## Step 5 — Create your local Kubernetes cluster

### Explanation

Creates cluster `kube-lab`. After this, `kubectl` has a **context** pointing at that cluster.

### Implementation

```bash
kind create cluster --name kube-lab
kubectl cluster-info --context kind-kube-lab
kubectl get nodes
```

Optional default context:

```bash
kubectl config use-context kind-kube-lab
```

### Verification

```bash
kubectl config current-context
kubectl get nodes
```

Expected: context `kind-kube-lab`, one node `Ready`.

### Cleanup / revert

Delete the cluster when you want resources back or need a clean recreate:

```bash
kind delete cluster --name kube-lab
```

Recreate anytime: `kind create cluster --name kube-lab`

More: [00-tooling-setup-cleanup.md](00-tooling-setup-cleanup.md)

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
| kind cluster `kube-lab` | `kind delete cluster --name kube-lab` |
| kubectl context | `kubectl config delete-context kind-kube-lab` |
| AWS profile `lab` (local) | Remove `[lab]` from `~/.aws/credentials` and `~/.aws/config` |
| Tools (optional) | `brew uninstall kind kubectl terraform awscli` — only if unused elsewhere |

Details: [00-tooling-setup-cleanup.md](00-tooling-setup-cleanup.md)

---

## Break & repair (macOS)

**`docker version` shows client but not server**  
→ Open Docker Desktop, or run `colima start`.

**`kubectl get nodes` connection refused**  
→ `kind create cluster --name kube-lab` or `kubectl config use-context kind-kube-lab`.

**`kind create cluster` fails**  
→ Fix Docker first; if name conflict: `kind delete cluster --name kube-lab`.

**AWS identity fails**  
→ See Step 7 break & repair (profile not found vs wrong keys).

Previous: [Tooling index](00-tooling-setup.md) · Next: [Phase 1 — Docker](../01-docker/01-docker-basics.md)
