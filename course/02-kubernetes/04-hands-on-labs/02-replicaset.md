# Lab 2 — ReplicaSet

## Explanation

A **Pod** is one running copy of your app. If that Pod crashes or you delete it, nothing brings it back unless you do it manually. That is fragile when you need **several identical copies** always running (for capacity or availability).

A **ReplicaSet** solves that: you tell Kubernetes “keep **N** Pods that match these labels,” and it continuously creates or removes Pods to hit that count. If you delete one Pod, the ReplicaSet notices the count dropped and starts a replacement.

**Why this lab?** You already created a single Pod in [Lab 1](01-pod.md). Here you see automatic replacement and steady replica count — the foundation for Deployments (Lab 3).

**Prerequisites**

- [Phase 0 tooling](../../00-getting-started/00-tooling-setup.md) — local **kind** cluster `kube-lab`, `kubectl` working.
- [Lab 0 — Setup](00-setup.md) — namespace `kube-lab` exists and your context points to it.
- Concept recap: [ReplicaSet](../02-core-concepts/replicaset.md), [Pod](../02-core-concepts/pod.md).

**Shell:** commands below run in your **local terminal**. `kubectl` talks to the kind cluster from Phase 0. Confirm namespace:

```bash
kubectl config view --minify | grep namespace:
```

Expected: `namespace: kube-lab`.

## Key fields (before you apply)

| Field | What it controls |
|---|---|
| `kind: ReplicaSet` | This object manages a set of identical Pods. |
| `replicas: 3` | Desired number of Pods — ReplicaSet tries to keep exactly three running. |
| `selector.matchLabels` | Which existing Pods “belong” to this ReplicaSet (must match Pod template labels). |
| `template` | Blueprint for new Pods (image, labels, containers) — same idea as a standalone Pod spec. |
| `template.metadata.labels.app` | Label on each Pod; must match `selector` so the ReplicaSet counts them. |

If selector labels and template labels disagree, the ReplicaSet cannot adopt its own Pods — a common misconfiguration.

## Implementation

Create a ReplicaSet with three nginx Pods:

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: rs-demo
  template:
    metadata:
      labels:
        app: rs-demo
    spec:
      containers:
        - name: web
          image: nginx:1.27
EOF
```

Inspect the ReplicaSet and its Pods:

```bash
kubectl get rs rs-demo
kubectl get pods -l app=rs-demo
```

Test self-healing — delete one Pod and watch replacements:

```bash
kubectl delete pod -l app=rs-demo --wait=false
kubectl get pods -l app=rs-demo -w
```

Press `Ctrl+C` to stop watching when all Pods are `Running` again.

## Verification

Success looks like this:

```bash
kubectl get rs rs-demo
```

- `DESIRED`, `CURRENT`, and `READY` columns all show **3**.

```bash
kubectl get pods -l app=rs-demo
```

- Three Pods, all `STATUS` **Running**.
- After the delete test, you still have three Pods (new names if old ones were removed).

## Break & repair

**Symptom:** `kubectl get rs rs-demo` shows `DESIRED 3` but `READY 0`, and Pods may be missing or stuck.

**Cause:** `selector.matchLabels` does not match `template.metadata.labels` (for example `app: rs-demo` vs `app: rs-wrong`).

**Repair:**

```bash
kubectl describe rs rs-demo
```

Look for events mentioning selector/template mismatch. Fix the YAML so selector and template labels match, then re-apply. If the ReplicaSet is broken beyond a quick fix, delete it and recreate:

```bash
kubectl delete rs rs-demo
```

Then run the `cat <<'EOF' | kubectl apply` block again with matching labels.

## You should now be able to…

- [ ] Explain what a ReplicaSet guarantees (steady count of identical Pods).
- [ ] Point to `replicas`, `selector`, and `template` in a ReplicaSet YAML.
- [ ] Delete a Pod and observe the ReplicaSet creating a replacement.
- [ ] Use `kubectl get rs` and `kubectl get pods -l app=rs-demo`.

---

Previous: [Pod](01-pod.md) · Next: [Deployment](03-deployment.md)
