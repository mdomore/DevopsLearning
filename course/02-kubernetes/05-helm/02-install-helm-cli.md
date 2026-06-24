# Install Helm and the CLI

**Prerequisite:** [What is Helm?](01-what-is-helm.md) and a working cluster from [Phase 0](../../00-getting-started/00-tooling-setup.md). Namespace `kube-lab` exists ([prerequisites](../01-foundations/00-prerequisites.md)).

Commands run in your **local terminal**. Helm talks to the same cluster as `kubectl` (same kubeconfig context).

---

## Explanation

### What the Helm CLI is

**Helm** is a **command-line program** (`helm`) on your machine. It:

1. Reads **charts** (local folder or downloaded from a **repository**)
2. Renders **templates** with **values**
3. Sends the result to the Kubernetes API (like `kubectl apply`, but with **release** tracking)

Helm 3 has **no server component** in the cluster at install time — only your CLI + Kubernetes.

### CLI command map (this chapter)

| Command | Role |
|---|---|
| `helm version` | Check client (and server/kube connection) |
| `helm repo add / update / search` | **Repository** — find charts online |
| `helm install` | Create a **release** from a chart |
| `helm list` | List releases in a namespace |
| `helm status` | State of one release |
| `helm get manifest` | Rendered YAML for a release |
| `helm upgrade` | New **revision** of a release |
| `helm rollback` | Revert to an older revision |
| `helm uninstall` | Remove release + its resources |
| `helm template` | Preview YAML locally (no cluster change) |
| `helm create` | Scaffold a new chart folder |
| `helm package` | Build a `.tgz` **package** from a chart |

You will use each command in the following lessons.

---

## Implementation — Step 1: Install Helm

### macOS (Homebrew)

```bash
brew install helm
```

### Linux (official script)

```bash
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### Verification

```bash
helm version
```

Expected: `version.BuildInfo{Version:"v3.x..."}` on the **Client** line.

---

## Implementation — Step 2: Point Helm at your cluster

Helm uses the **same context** as `kubectl`:

```bash
kubectl config current-context
kubectl get nodes
helm version
```

If `kubectl get nodes` works, Helm can reach the API. The **Server** section in `helm version` may show Kubernetes version info when connected.

Set context if needed (Path A or B):

```bash
kubectl config use-context kind-kube-lab      # Path A
# or
kubectl config use-context docker-desktop   # Path B
```

Ensure lab namespace:

```bash
kubectl get ns kube-lab || kubectl create namespace kube-lab
```

---

## Implementation — Step 3: Explore the CLI

```bash
helm help
helm help install
helm env | grep KUBECONFIG
```

`helm env` shows environment variables Helm respects (`KUBECONFIG`, `HELM_NAMESPACE`, etc.). Usually you do not need to change them.

---

## Verification

- [ ] `helm version` shows client v3.x
- [ ] `kubectl get nodes` → Ready (same session)
- [ ] `helm help install` prints usage without error

---

## Break & repair

**`helm: command not found`**  
→ Re-run install; ensure `/usr/local/bin` or Homebrew bin is on your `PATH`.

**`Error: Kubernetes cluster unreachable`**  
→ Start cluster per your [macOS](../../00-getting-started/00-tooling-setup-macos.md) or [Linux](../../00-getting-started/00-tooling-setup-linux.md) guide; fix `kubectl get nodes` first.

**Mac: old Helm 2 (`helm init`, Tiller)**  
→ Uninstall Helm 2; this course requires **Helm 3** only.

---

## Cleanup / revert

Nothing to clean up yet — you only installed the CLI.

---

## You should now be able to…

- Install Helm 3 on macOS or Linux.
- Explain that Helm uses the same kubeconfig as `kubectl`.
- Name the main CLI commands used for **repository**, **install**, **upgrade**, **rollback**, **uninstall**, and **package**.

Next: [Chart, values, and templates](03-chart-values-templates.md)

Previous: [What is Helm?](01-what-is-helm.md)
