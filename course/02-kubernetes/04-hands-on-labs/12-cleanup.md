# Lab 12 — Cleanup

## Explanation

Throughout these labs you created many objects in namespace **kube-lab**: Pods, Deployments, Services, Secrets, PVCs, and more. Leaving them running wastes cluster resources and can confuse later exercises (duplicate names, old Services still bound to ports).

**Problem it solves:** Reset the lab environment to a clean slate without deleting the whole kind cluster.

**Why delete the namespace?** A **namespace** is a scope inside the cluster. Deleting `kube-lab` removes **all** objects inside it in one step — simpler than deleting each resource by hand.

**Prerequisites**

- You completed (or want to abandon) labs [02](02-replicaset.md)–[11](11-ingress.md).
- [Phase 0 tooling](../../00-getting-started/00-tooling-setup.md) — kind cluster still running.
- You understand this is **destructive for lab data** in `kube-lab` only — it does not remove the kind cluster itself.

**Shell:** local terminal, `kubectl` → kind cluster.

## Key fields (before you apply)

| Command | What it does |
|---|---|
| `kubectl config set-context --current --namespace=default` | Switches your kubectl default namespace away from `kube-lab` so you do not accidentally run commands in a namespace you are about to delete. |
| `kubectl delete namespace kube-lab` | Deletes namespace and all resources inside it (asynchronous — may take a minute). |

There is no YAML in this lab — only kubectl commands.

## Implementation

Switch default namespace to `default`:

```bash
kubectl config set-context --current --namespace=default
```

Delete the lab namespace and everything in it:

```bash
kubectl delete namespace kube-lab
```

Wait until deletion finishes:

```bash
kubectl get namespace kube-lab
```

Repeat until the namespace no longer appears (or shows `Terminating` then disappears).

## Verification

```bash
kubectl get namespace kube-lab
```

- Error `NotFound` or empty result — namespace is gone.

```bash
kubectl config view --minify | grep namespace:
```

- Shows `namespace: default` (or your chosen non-lab namespace).

```bash
kubectl get all -n kube-lab
```

- Cannot list resources — namespace does not exist.

To run labs again, start from [Lab 0 — Setup](00-setup.md) and recreate `kube-lab`.

## Break & repair

**Symptom:** `kubectl get namespace kube-lab` stays `Terminating` for a long time.

**Cause:** Some resources have **finalizers** (common with PVCs, LoadBalancers, or custom controllers) that block deletion until cleanup completes.

**Repair:**

```bash
kubectl api-resources --verbs=list --namespaced -o name | xargs -n 1 kubectl get -n kube-lab 2>/dev/null
kubectl get pvc -n kube-lab
kubectl describe namespace kube-lab
```

- Identify stuck objects (often PVCs or Pods).
- Delete stuck namespaced objects explicitly:

```bash
kubectl delete pvc --all -n kube-lab
kubectl delete pod --all -n kube-lab --force --grace-period=0
```

- On kind labs, if namespace is still stuck, you can recreate the whole cluster (Phase 0): `kind delete cluster --name kube-lab` then `kind create cluster --name kube-lab` — only if you are comfortable redoing Phase 0 cluster setup.

## You should now be able to…

- [ ] Explain why deleting a namespace cleans up all lab objects together.
- [ ] Switch kubectl context namespace before destructive cleanup.
- [ ] Confirm namespace removal with `kubectl get namespace`.
- [ ] Restart the lab path from [00-setup.md](00-setup.md) when needed.

---

Previous: [Ingress](11-ingress.md)
