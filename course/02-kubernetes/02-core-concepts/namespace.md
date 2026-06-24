# Namespace

## Explanation

A **namespace** is a **scope** inside one Kubernetes cluster — a way to group and separate resources.

**Problem it solves:** Many teams and apps share the same cluster. Without namespaces, everyone would create Pods and Services in the same pile (`default`), with name collisions, no clear ownership, and painful cleanup.

**Plain analogy:** One cluster = one building. A namespace = one **floor** or **apartment**. Objects in `team-a` do not share names with objects in `team-b`, and you can delete everything on the “lab floor” without touching production.

### What a namespace is **not**

- **Not** a separate cluster — same nodes, same control plane
- **Not** strong security isolation by itself (RBAC and network policies add that)
- **Not** optional for thinking — even if you use `default`, you are still *in* a namespace

### Concrete use cases

| Situation | Namespaces (examples) | Why |
|---|---|---|
| **Environments** | `dev`, `staging`, `prod` | Same app, different config; limit blast radius |
| **Teams / projects** | `billing`, `frontend`, `data` | Ownership and quotas per team |
| **System vs apps** | `kube-system`, `kube-lab` | Keep your labs away from cluster internals |
| **This course** | `kube-lab` | All lab Pods/Services in one place; delete namespace = reset labs |
| **Helm releases** | install in `kube-lab` or `prod` | Release names are unique **per namespace** |

### Rules that matter day to day

1. **Names are unique per namespace** — `api` Deployment in `dev` and `api` in `prod` can coexist
2. **Same name, different namespace = different object**
3. **Services DNS (short form)** — inside the cluster, `my-svc` in namespace `kube-lab` is reachable as `my-svc.kube-lab.svc.cluster.local`
4. **Most objects are namespaced** — Pod, Deployment, Service, ConfigMap, Secret, Ingress…
5. **Some objects are cluster-wide** — Node, PersistentVolume, StorageClass, Namespace itself

### Example — two namespaces side by side

```bash
kubectl create namespace dev
kubectl create namespace prod

# Same Deployment name, different namespaces — OK
kubectl create deployment api --image=nginx:1.27 -n dev
kubectl create deployment api --image=nginx:1.27 -n prod

kubectl get deploy -n dev
kubectl get deploy -n prod
```

Cleanup one environment only:

```bash
kubectl delete namespace dev    # prod is untouched
```

### In YAML

```yaml
metadata:
  name: my-pod
  namespace: kube-lab   # optional if kubectl default is already kube-lab
```

If `namespace:` is omitted, Kubernetes uses the namespace from your **context** or `default`.

## Example — add to the cluster

**What's new:** everything in this chapter lives in **`kube-lab`** — a dedicated sandbox floor.

**Before this step:** cluster running ([Phase 0](../../00-getting-started/00-tooling-setup.md)).

Create and set as default for your context:

```bash
kubectl create namespace kube-lab --dry-run=client -o yaml | kubectl apply -f -
kubectl config set-context --current --namespace=kube-lab
kubectl config view --minify | grep namespace
```

All later concept examples use `namespace: kube-lab` in YAML or rely on this default.

### Verify

```bash
kubectl get namespace kube-lab
kubectl run ns-check --rm -it --restart=Never --image=busybox:1.36 -n kube-lab -- echo ok
```

### Break & repair

Create the same Deployment name in two namespaces — no conflict:

```bash
kubectl create deployment demo --image=nginx:1.27 -n default
kubectl create deployment demo --image=nginx:1.27 -n kube-lab
kubectl get deploy demo -n default
kubectl get deploy demo -n kube-lab
kubectl delete deployment demo -n default
kubectl delete deployment demo -n kube-lab
```

Start the stack: [Pod](pod.md) · Full order: [progressive-stack.md](progressive-stack.md)

## How it relates

- **Context (kubectl)** — stores your *default* namespace for commands without `-n`
- **Service** — DNS includes namespace when calling across namespaces
- **RBAC** — permissions are often granted per namespace
- **Helm release** — scoped to the namespace where you `helm install -n …`

Hands-on: [Lab 0 — Setup](../04-hands-on-labs/00-setup.md) · Cleanup: [Lab 12](../04-hands-on-labs/12-cleanup.md)

## You should now be able to…

- Explain a namespace in one sentence and give two real-world examples (envs, teams, lab sandbox).
- State why `kube-lab` is used in this course.
- List namespaced vs cluster-scoped object types (basic list).

Previous: [Core concepts index](README.md)
