# Lab 0 — Setup

## Explanation

Before creating workloads, we prepare a **safe sandbox** inside the cluster.

A **namespace** is a logical partition in Kubernetes — like a folder that groups resources. We use `kube-lab` so lab objects stay separate from system components and are easy to delete later.

**Why set the default namespace on your context?**  
`kubectl` needs to know *where* to create resources. Setting `kube-lab` as the default means you can type `kubectl get pods` instead of `kubectl get pods -n kube-lab` every time.

**Prerequisites**

- [Phase 0 tooling](../../00-getting-started/00-tooling-setup.md) completed — local cluster and namespace `kube-lab` (OS setup guide).
- `kubectl get nodes` shows one node `Ready`.

**Shell:** local terminal, `kubectl` connected to your cluster (`kind-kube-lab` or `docker-desktop`).

## Implementation

Create the lab namespace:

```bash
kubectl create namespace kube-lab
```

Tell kubectl to use `kube-lab` by default for this context:

```bash
kubectl config set-context --current --namespace=kube-lab
```

Confirm the namespace exists:

```bash
kubectl get ns kube-lab
```

## Verification

```bash
kubectl config view --minify | grep namespace:
kubectl get ns kube-lab
```

Expected:

- `namespace: kube-lab` appears in your context
- `kube-lab` status is `Active`

## Break & repair

**`namespace/kube-lab already exists`**  
→ That is fine — it was created in a previous run. Continue to the next command.

**Commands affect `default` namespace instead of `kube-lab`**  
→ Re-run `kubectl config set-context --current --namespace=kube-lab`  
→ Check with `kubectl config view --minify | grep namespace:`

## You should now be able to…

- Explain what a namespace is in one sentence
- Create `kube-lab` and set it as your default namespace
- Confirm your kubectl context points at `kube-lab`

Next: [Lab 1 — Pod](01-pod.md)
