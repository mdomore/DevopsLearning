# Cleanup and revert — undo course setup safely

Use this when you want to **tear down lab resources**, **free disk space**, or **undo something you created during the course** — without breaking unrelated work on your machine.

**Shell:** local terminal unless noted.

Index: [00-tooling-setup.md](00-tooling-setup.md)

---

## Explanation

Not everything is reversible the same way:

| What you created | Reversible? | How |
|---|---|---|
| kind cluster `kube-lab` (Path A) | Yes | Delete the cluster — removes cluster containers |
| Docker Desktop Kubernetes (Path B) | Yes | Disable/remove in Docker Desktop → Settings → Kubernetes |
| Kubernetes objects in `kube-lab` namespace | Yes | Delete namespace or individual resources |
| kubectl context `kind-kube-lab` or `docker-desktop` | Yes | Remove from kubeconfig after cluster delete / disable |
| AWS CLI profile `lab` (local files only) | Yes | Remove `[lab]` sections from `~/.aws/*` |
| AWS IAM user + keys (cloud) | Yes | Delete in IAM console |
| Docker images/containers from labs | Mostly yes | Stop/remove containers; optionally remove images |
| Homebrew packages (`kubectl`, `kind`, …) | Yes | `brew uninstall` — only if you want tools gone |
| Your **default** AWS profile | Do not delete | Course should never require changing it |

**Rule:** Revert **lab-specific** things first. Uninstall global tools only if you are sure nothing else on your machine needs them.

---

## 1 — Delete the local Kubernetes cluster (kind)

### When to use

- You finished a study session and want RAM/CPU back
- `kind create cluster` failed and you want a clean start
- You want to recreate the cluster from scratch

### Commands

List clusters:

```bash
kind get clusters
```

Delete the course cluster:

```bash
kind delete cluster --name kube-lab
```

Verify:

```bash
kind get clusters
kubectl config get-contexts
```

Expected: `kube-lab` no longer listed; context `kind-kube-lab` may still appear until you clean kubeconfig (optional, below).

### Recreate later

```bash
kind create cluster --name kube-lab
kubectl config use-context kind-kube-lab
```

---

## 1b — Disable Docker Desktop Kubernetes (Path B)

### When to use

- You use Docker Desktop’s built-in cluster (context `docker-desktop`)
- You want RAM/CPU back without uninstalling Docker Desktop

### Steps

1. Open **Docker Desktop**
2. **Settings → Kubernetes** — disable Kubernetes or remove the cluster
3. **Apply & restart** if prompted

Verify:

```bash
kubectl config get-contexts
kubectl get nodes   # should fail or point at another context if cluster is gone
```

To re-enable later, use **Kubernetes → Create cluster** in Docker Desktop, then:

```bash
kubectl config use-context docker-desktop
kubectl get nodes
```

More: [00-tooling-setup-macos.md](00-tooling-setup-macos.md) (Option B) · [00-tooling-setup-cleanup.md](00-tooling-setup-cleanup.md)

---

## 2 — Remove kubectl context (optional)

After deleting the kind cluster or disabling Docker Desktop Kubernetes, your kubeconfig may still list `kind-kube-lab` or `docker-desktop`.

View contexts:

```bash
kubectl config get-contexts
```

Remove stale contexts after deleting a cluster:

```bash
kubectl config delete-context kind-kube-lab      # Path A
kubectl config delete-context docker-desktop   # Path B (optional)
```

If you set `kube-lab` as default namespace on another context, switch namespace back:

```bash
kubectl config set-context --current --namespace=default
```

**Note:** `~/.kube/config` is a YAML file. The commands above edit it safely — you do not need to delete the whole file unless you want a completely fresh kubectl config.

---

## 3 — Delete Kubernetes lab objects (namespace)

### When to use

- Cluster still running but you want to wipe all course workloads
- Same as [Lab 12 — Cleanup](../02-kubernetes/04-hands-on-labs/12-cleanup.md)

```bash
kubectl config use-context kind-kube-lab
kubectl delete namespace kube-lab
```

Verify:

```bash
kubectl get ns kube-lab
```

Expected: `NotFound` or namespace disappears (deletion can take a minute).

Recreate before next lab:

```bash
kubectl create namespace kube-lab
kubectl config set-context --current --namespace=kube-lab
```

---

## 4 — Revert AWS CLI profile `lab` (local only)

This removes **local** saved keys for the lab profile. It does **not** delete anything in AWS unless you also remove the IAM user in the console.

### When to use

- You created Path B `lab` profile and want it gone
- You rotated keys and want to re-run `aws configure --profile lab`

### Steps

1. List profiles:

```bash
aws configure list-profiles
```

2. Open credentials file in an editor:

```bash
open -a TextEdit ~/.aws/credentials
```

On Linux, use `nano ~/.aws/credentials` or your preferred editor.

3. Delete the entire block:

```ini
[lab]
aws_access_key_id = ...
aws_secret_access_key = ...
```

4. Open config file and remove the block:

```ini
[profile lab]
region = ...
output = json
```

5. Verify:

```bash
aws configure list-profiles
aws sts get-caller-identity
```

Expected: `lab` no longer listed; `aws sts get-caller-identity` still works if **default** profile is intact.

**If you only used Path A (default):** nothing to revert for AWS profiles.

Full IAM user cleanup in AWS: [00-aws-lab-iam-user.md — Cleanup](00-aws-lab-iam-user.md#cleanup-when-you-finish-the-course).

---

## 5 — Undo default namespace shortcut

If kubectl always targets `kube-lab` and you want neutral behavior:

```bash
kubectl config set-context --current --namespace=default
kubectl config view --minify | grep namespace:
```

Or point at another context you use for non-course work:

```bash
kubectl config use-context <your-other-context>
```

---

## 6 — Clean Docker lab containers and images

### When to use

- Free disk space after Docker chapter labs
- Remove stopped containers and unused images

**Warning:** These commands affect Docker globally on your machine, not only the course. Read output before confirming.

List what is running:

```bash
docker ps
docker ps -a
```

Stop a specific container (replace name/id):

```bash
docker stop <container_id>
docker container prune
```

Remove unused images (prompts for confirmation):

```bash
docker image prune
```

For Colima users, stop the VM when done:

```bash
colima stop
```

Docker Desktop: quit the app from the menu bar when you do not need containers.

---

## 7 — Uninstall tools (optional, last resort)

Only if you want to remove packages installed **for this course** and nothing else needs them.

macOS (Homebrew) examples:

```bash
brew uninstall kind kubectl terraform awscli
# Docker: uninstall Docker Desktop from Applications, or:
brew uninstall colima docker docker-compose
```

Linux (apt) examples — uninstall only what you installed in Phase 0; package names may vary.

**Homebrew itself:** usually keep it — other software may depend on it.

---

## 8 — Terraform state (later chapters)

When you run Terraform labs, Terraform creates a **state file** (`.tfstate`) tracking what it built.

To destroy resources Terraform created:

```bash
cd <your-lab-directory>
terraform destroy
```

Only run `destroy` in directories **you** created for this course. Never run it against production paths.

Details: [Terraform — State and plan/apply](../03-terraform/02-state-plan-apply.md).

---

## Quick reference — “I want to undo X”

| I want to… | Command / action |
|---|---|
| Remove local K8s cluster | `kind delete cluster --name kube-lab` |
| Wipe lab workloads only | `kubectl delete namespace kube-lab` |
| Remove AWS `lab` profile locally | Edit `~/.aws/credentials` and `~/.aws/config` |
| Stop using `lab` in this shell | `unset AWS_PROFILE` |
| Free Docker disk | `docker container prune` / `docker image prune` |
| Destroy cloud resources from Terraform | `terraform destroy` (in lab folder) |

---

## You should now be able to…

- Explain the difference between deleting a **kind cluster** and deleting the **kube-lab namespace**
- Remove the AWS `lab` profile locally without touching **default**
- Know when to use `terraform destroy` vs `kubectl delete`

Previous: [Tooling setup index](00-tooling-setup.md)
