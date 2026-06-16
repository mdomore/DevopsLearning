# Phase 0 — Tooling setup on Linux (Ubuntu LTS)

Goal: install course tools on **Ubuntu** (22.04 or 24.04 LTS).  
These steps also work on most **Debian-based** distros with small adjustments.

Commands run in your **local terminal**. Several steps need `sudo`.

Index: [00-tooling-setup.md](00-tooling-setup.md)

---

## Step 0 — Confirm your distro

### Explanation

Package names and repositories differ between distros. This guide uses **Ubuntu LTS** because it is the most common Linux choice for learning and labs.

### Implementation

```bash
cat /etc/os-release
uname -m
```

### Verification

You should see `Ubuntu` and version `22.04` or `24.04` (or similar LTS).  
`uname -m` is usually `x86_64` or `aarch64`.

---

## Step 1 — Update base packages

### Explanation

Refresh the system package index before adding new repositories.

### Implementation

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg lsb-release unzip
```

### Verification

```bash
apt-cache policy ca-certificates | head -n 2
```

---

## Step 2 — Install Docker Engine

### Explanation

You need the **Docker Engine** (daemon + CLI) to run containers.  
`kind` talks to Docker to create Kubernetes nodes.

We install from Docker’s official Ubuntu repository (more up to date than the default `docker.io` package).

### Implementation

```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Allow your user to run Docker without `sudo` every time:

```bash
sudo usermod -aG docker "$USER"
newgrp docker
```

### Verification

```bash
docker version
docker run --rm hello-world
```

Expected: client **and** server info; `hello-world` succeeds.

---

## Step 3 — Install kubectl

### Explanation

`kubectl` is the Kubernetes CLI. Install the client now; connect to a cluster in Step 5.

### Implementation (official Kubernetes apt repository)

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | \
  sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubectl
```

### Verification

```bash
kubectl version --client
```

---

## Step 4 — Install kind

### Explanation

`kind` is not always available as a distro package.  
We install the official binary into `/usr/local/bin`.

### Implementation

```bash
ARCH="$(uname -m)"
case "$ARCH" in
  x86_64) KIND_ARCH=amd64 ;;
  aarch64) KIND_ARCH=arm64 ;;
  *) echo "Unsupported architecture: $ARCH"; exit 1 ;;
esac

curl -Lo /tmp/kind https://kind.sigs.k8s.io/dl/v0.27.0/kind-linux-${KIND_ARCH}
chmod +x /tmp/kind
sudo mv /tmp/kind /usr/local/bin/kind
```

### Verification

```bash
kind version
```

---

## Step 5 — Create your local Kubernetes cluster

### Explanation

Creates cluster `kube-lab`. `kubectl` will use context `kind-kube-lab`.

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

```bash
kind delete cluster --name kube-lab
```

Recreate: `kind create cluster --name kube-lab`  
Full guide: [00-tooling-setup-cleanup.md](00-tooling-setup-cleanup.md)

---

## Step 6 — Install Terraform

### Explanation

Terraform is used in later chapters to provision AWS infrastructure.

### Implementation (HashiCorp apt repository)

```bash
wget -O- https://apt.releases.hashicorp.com/gpg | \
  sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt-get update
sudo apt-get install -y terraform
```

### Verification

```bash
terraform version
```

---

## Step 7 — Install and configure AWS CLI v2

### Explanation

The AWS CLI talks to your AWS account from the terminal.

A **profile** is a named set of settings (credentials + region). Think of it like saved connections:

| Profile | When it is used |
|---|---|
| **default** | When you run `aws ...` with no extra flag |
| **lab** (course) | When you run `aws ... --profile lab` |

**Important:** Adding a `lab` profile does **not** change or delete your existing setup.  
Profiles live side by side in:

- `~/.aws/credentials` — access keys per profile
- `~/.aws/config` — region and output format per profile

### Choose your path

**Path A — You already use AWS**  
You do **not** have to create a `lab` profile. Use your existing **default** profile.

When a later lab says `--profile lab`, run without `--profile` or use `--profile default`.

**Path B — Separate lab profile (optional)**  
1. Create a lab IAM user: [00-aws-lab-iam-user.md](00-aws-lab-iam-user.md)  
2. Configure CLI with that user’s keys (below)

This only adds `[lab]`; it does not touch `[default]`.

### Implementation — Install CLI

```bash
ARCH="$(uname -m)"
case "$ARCH" in
  x86_64) AWS_ARCH=x86_64 ;;
  aarch64) AWS_ARCH=aarch64 ;;
  *) echo "Unsupported architecture: $ARCH"; exit 1 ;;
esac

curl -fsSL "https://awscli.amazonaws.com/awscli-exe-linux-${AWS_ARCH}.zip" -o /tmp/awscliv2.zip
unzip -q /tmp/awscliv2.zip -d /tmp
sudo /tmp/aws/install
```

Skip install if `aws --version` already works.

### Implementation — Path B only: create `lab` profile

First complete: [Create an AWS lab IAM user](00-aws-lab-iam-user.md).

Then run:

```bash
aws configure --profile lab
```

| Prompt | What to enter |
|---|---|
| AWS Access Key ID | From your **lab** IAM user |
| AWS Secret Access Key | From the same lab IAM user |
| Default region | e.g. `eu-west-1` |
| Default output format | `json` |

**Do not run** `aws configure` without `--profile` unless you want to overwrite default.

### How to switch between profiles

```bash
# One command — explicit (safest)
aws sts get-caller-identity                  # default
aws sts get-caller-identity --profile lab  # lab

# Whole shell session
export AWS_PROFILE=lab
unset AWS_PROFILE    # back to default

# See active profile in this shell
echo "${AWS_PROFILE:-default}"

# List all profile names
aws configure list-profiles
```

### Verification

Path A (existing default):

```bash
aws --version
aws sts get-caller-identity
```

Path B (lab profile):

```bash
aws --version
aws sts get-caller-identity --profile lab
```

### Break & repair (AWS profiles)

**`The config profile (lab) could not be found`**  
→ Use default (Path A), or run `aws configure --profile lab`.

**Wrong account**  
→ `aws sts get-caller-identity --profile <name>` and check `echo "${AWS_PROFILE:-default}"`.

### Cleanup / revert (Step 7)

- Remove local `lab` profile: edit `~/.aws/credentials` and `~/.aws/config` (delete `[lab]` blocks).
- Remove IAM user in AWS: [00-aws-lab-iam-user.md — Cleanup](00-aws-lab-iam-user.md#cleanup-when-you-finish-the-course).
- Unset shell profile: `unset AWS_PROFILE`

---

## Cleanup / revert (Phase 0 overview)

| What | Revert |
|---|---|
| kind cluster | `kind delete cluster --name kube-lab` |
| AWS profile `lab` | Edit `~/.aws/credentials` and `~/.aws/config` |
| Tools (optional) | `sudo apt-get remove` / package manager — only if unused |

Full guide: [00-tooling-setup-cleanup.md](00-tooling-setup-cleanup.md)

---

## Phase 0 completion checklist (Ubuntu)

- [ ] `docker version` (client + server)
- [ ] `kubectl version --client`
- [ ] `kind version`
- [ ] `kubectl get nodes` shows `Ready`
- [ ] `terraform version`
- [ ] `aws --version`

---

## Break & repair (Ubuntu)

**`permission denied` on `docker` commands**  
→ User not in `docker` group: `sudo usermod -aG docker "$USER"`, then log out/in or `newgrp docker`.

**`kubectl: command not found` after install**  
→ Re-run Step 3; check `/etc/apt/sources.list.d/kubernetes.list` exists.

**`kind create cluster` fails with cgroup or iptables errors**  
→ Ensure Docker daemon is running: `sudo systemctl status docker`  
→ Start if needed: `sudo systemctl enable --now docker`

**Terraform apt install fails on unknown release**  
→ Confirm `lsb_release -cs` output matches a HashiCorp-supported Ubuntu codename.

**AWS unzip/install fails**  
→ Install unzip: `sudo apt-get install -y unzip`

---

## Other Linux distros (short pointers)

| Distro | Package tool | Notes |
|---|---|---|
| **Debian** | `apt` | Same flow as Ubuntu; Docker repo uses `debian` URL instead of `ubuntu` |
| **Fedora / RHEL** | `dnf` | Use Docker CE docs for Fedora; `kubectl` via Kubernetes yum repo |
| **Arch** | `pacman` | `docker`, `kubectl`, `terraform`, `aws-cli` in official/extra repos |

If you use another distro, follow its official Docker + Kubernetes install docs, then return here for Steps 5–7 (kind cluster + AWS profile).

Previous: [Tooling index](00-tooling-setup.md) · Next: [Phase 1 — Docker](../01-docker/)
