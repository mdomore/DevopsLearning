# Prerequisites

**Before this chapter:** Complete [Phase 0 — Tooling setup](../../00-getting-started/00-tooling-setup.md) — cluster and namespace `kube-lab` are created in your OS guide (macOS Step 5 / Linux Step 5).

If you use **Docker Desktop Kubernetes** on macOS, context is **`docker-desktop`** instead of `kind-kube-lab` when lessons mention the latter.

Commands in this section run in your **local terminal** against your Kubernetes cluster unless noted.

---

## Explanation

This chapter assumes your machine can talk to a Kubernetes cluster using **`kubectl`** — the command-line client for the Kubernetes API.

**Problem this section solves:** Later labs create Pods, Deployments, and Services. Without a working cluster and `kubectl` context, those commands fail immediately. This page confirms the basics before you invest time in architecture and YAML.

### What is a namespace?

A **namespace** is a **logical partition** inside **one** cluster — like a folder that groups Kubernetes objects (Pods, Services, Deployments…).

```text
Cluster (your laptop or EKS)
├── namespace: kube-system    ← Kubernetes internals (DNS, etc.)
├── namespace: default        ← default if you specify nothing
└── namespace: kube-lab       ← your course sandbox
        ├── Pod / Deployment / Service / …
```

**Why the course uses `kube-lab`:**

| Benefit | Concrete example |
|---|---|
| **Isolation** | Lab Pods do not mix with `default` or `kube-system` |
| **Clean reset** | `kubectl delete namespace kube-lab` removes all lab objects at once ([Lab 12](../04-hands-on-labs/12-cleanup.md)) |
| **Same names as docs** | Tutorials can say `deploy-demo` without clashing with other work |
| **Safer experiments** | Breaking a Deployment in `kube-lab` does not touch system components |

**Concrete cases in real teams:**

| Use case | Typical namespaces |
|---|---|
| Separate **environments** | `dev`, `staging`, `prod` — same app, three copies |
| Separate **teams or apps** | `payments`, `catalog`, `monitoring` |
| **Platform / system** | `kube-system`, `ingress-nginx`, `cert-manager` |
| **CI preview** | `pr-1234` — ephemeral, deleted after merge |

**Important rules:**

- Resource names are unique **inside** a namespace — you can have Service `api` in `dev` and `api` in `prod`.
- **`kubectl get pods`** without `-n` uses the namespace stored in your **context** (we set `kube-lab`).
- **`kubectl get pods -n kube-lab`** always targets that namespace explicitly.
- Services are reachable as `service-name.namespace.svc.cluster.local` from other namespaces (short form `service-name.namespace`).

**What namespaces do not do:** they are not separate machines and not full security isolation alone — they organize resources and permissions; network policies and RBAC add stronger boundaries.

More detail: [Namespace — core concept](../02-core-concepts/namespace.md).

---

## Implementation

Verify the kubectl client is installed:

```bash
kubectl version --client
```

Confirm kubectl knows which cluster it talks to (**context** = cluster + user + default namespace):

```bash
kubectl config current-context
kubectl cluster-info
```

Check that at least one **node** (worker machine in the cluster) is ready:

```bash
kubectl get nodes
```

Create the lab namespace and make it the default for your current context (recommended for this course):

```bash
kubectl create namespace kube-lab
kubectl config set-context --current --namespace=kube-lab
```

**Note:** If `kube-lab` already exists (Phase 0 setup), you may see `AlreadyExists` — that is fine.

### List namespaces and verify you are in `kube-lab`

**Why:** Creating a namespace is not enough — you must confirm kubectl will send commands to **`kube-lab`**, not `default`.

List all namespaces:

```bash
kubectl get namespaces
# short form:
kubectl get ns
```

Check one namespace in detail:

```bash
kubectl get ns kube-lab
```

Expected: `NAME kube-lab`, `STATUS Active`.

**Which namespace is default for your current context?**

```bash
kubectl config view --minify | grep namespace:
```

Expected: `namespace: kube-lab`.

Alternative — show current context and its namespace:

```bash
kubectl config get-contexts
```

The row with `*` is active. The **NAMESPACE** column shows the default (should be `kube-lab`).

**Quick test — where do Pods go without `-n`?**

```bash
kubectl get pods
```

With default namespace set to `kube-lab`, this lists Pods **in `kube-lab`** (empty for now is OK).

To target a namespace explicitly (always works, even if default changes):

```bash
kubectl get pods -n kube-lab
kubectl get pods -n default
```

| Command | What it shows |
|---|---|
| `kubectl get ns` | All namespaces in the cluster |
| `kubectl config view --minify \| grep namespace:` | Default namespace on **current context** |
| `kubectl get pods` | Resources in the **default** namespace of the context |
| `kubectl get pods -n kube-lab` | Resources in `kube-lab` (explicit) |

Confirm you can create resources in that namespace:

```bash
kubectl auth can-i create pods --namespace=kube-lab
```

Expect `yes` if your user has normal lab permissions.

---

## Verification

You should see:

- `kubectl version --client` prints `Client Version` without “command not found”.
- `kubectl get nodes` shows at least one node with `STATUS` **Ready** (name may be `kube-lab-control-plane` with kind, or `desktop-control-plane` with Docker Desktop).
- `kubectl get ns` lists `kube-lab` with `STATUS` **Active**.
- `kubectl config view --minify | grep namespace:` shows `namespace: kube-lab`.
- `kubectl auth can-i create pods --namespace=kube-lab` returns **yes**.

---

## Break & repair

**Break:** Point kubectl at a missing or stopped cluster (wrong context):

```bash
kubectl config get-contexts
# If you have a bogus context, switch to it temporarily, or stop your kind cluster:
# kind stop cluster --name kube-lab   # example; your setup may differ
kubectl get nodes
```

**Observe:** Connection refused, “Unable to connect to the server”, or no nodes Ready.

**Repair:** Start your local cluster again (see your [macOS](../../00-getting-started/00-tooling-setup-macos.md) or [Linux](../../00-getting-started/00-tooling-setup-linux.md) setup guide), then switch context:

```bash
kubectl config get-contexts
kubectl config use-context kind-kube-lab      # kind (macOS Option A / Linux)
# or
kubectl config use-context docker-desktop   # Docker Desktop (macOS Option B)
kubectl get nodes
```

**Break:** Forget to set namespace and run lab commands in `default`:

```bash
kubectl config set-context --current --namespace=default
kubectl get pods
```

**Repair:** Switch back to the lab namespace:

```bash
kubectl config set-context --current --namespace=kube-lab
```

---

## You should now be able to…

- Run `kubectl version`, `kubectl cluster-info`, and `kubectl config current-context` and explain what each shows.
- Confirm nodes are Ready and you have permission to create Pods in `kube-lab`.
- List namespaces with `kubectl get ns` and confirm `kube-lab` is **Active**.
- Verify the default namespace with `kubectl config view --minify | grep namespace:`.
- Explain what a **namespace** is and why this course uses `kube-lab` (with two concrete real-world examples).

Next: [Where to run your cluster](01-where-to-run-cluster.md)
