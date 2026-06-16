# Lab 3 — Deployment

## Explanation

A **ReplicaSet** keeps Pod count steady but does not help you **change** the app safely (for example swap nginx 1.27 → 1.26 with minimal downtime). A **Deployment** wraps ReplicaSets and adds **rolling updates** and **rollbacks**.

**Problem it solves:** “Run N copies of my app” plus “update the image or config without manual Pod surgery.”

**Why this lab?** Deployments are what you use in real clusters day to day. This lab builds on [Lab 2 — ReplicaSet](02-replicaset.md): same Pod template idea, plus update workflow.

**Prerequisites**

- [Phase 0 tooling](../../00-getting-started/00-tooling-setup.md) — kind cluster `kube-lab`.
- [Lab 0 — Setup](00-setup.md) — namespace `kube-lab`.
- Concept recap: [Deployment](../02-core-concepts/deployment.md), [ReplicaSet](../02-core-concepts/replicaset.md).

**Shell:** local terminal, `kubectl` → kind cluster, namespace `kube-lab`.

## Key fields (before you apply)

| Field | What it controls |
|---|---|
| `kind: Deployment` | Manages ReplicaSets (and thus Pods) for an app. |
| `replicas: 2` | How many Pod copies to run. |
| `selector` / `template.metadata.labels` | Same matching rule as ReplicaSet — labels must align. |
| `containers[].image` | Container image; changing this triggers a rollout. |
| `containers[].ports.containerPort` | Port the container listens on (used later by Services). |

Deployment updates are driven by changes to the Pod template (image, env, etc.).

## Implementation

Create a Deployment with two nginx Pods:

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: deploy-demo
  template:
    metadata:
      labels:
        app: deploy-demo
    spec:
      containers:
        - name: web
          image: nginx:1.27
          ports:
            - containerPort: 80
EOF
```

Check status:

```bash
kubectl get deploy deploy-demo
kubectl rollout status deploy/deploy-demo
```

Rolling update — change the image version:

```bash
kubectl set image deploy/deploy-demo web=nginx:1.26
kubectl rollout status deploy/deploy-demo
```

Rollback to the previous version:

```bash
kubectl rollout undo deploy/deploy-demo
```

## Verification

After create:

```bash
kubectl get deploy deploy-demo
```

- `READY` shows **2/2** (or `2` in the ready column depending on kubectl version).

After `set image`:

```bash
kubectl rollout status deploy/deploy-demo
```

- Ends with `successfully rolled out`.

```bash
kubectl get pods -l app=deploy-demo -o jsonpath='{range .items[*]}{.spec.containers[0].image}{"\n"}{end}'
```

- Pods show `nginx:1.26` after update, then `nginx:1.27` after undo.

## Break & repair

**Symptom:** `kubectl rollout status` hangs or fails; new Pods stay `ImagePullBackOff` or `ErrImagePull`.

**Cause:** Image name/tag is wrong (typo or non-existent tag), for example `nginx:9.99`.

**Repair:**

```bash
kubectl describe pod -l app=deploy-demo
```

Read `Events` for pull errors. Fix the image:

```bash
kubectl set image deploy/deploy-demo web=nginx:1.27
kubectl rollout status deploy/deploy-demo
```

Or undo the bad rollout:

```bash
kubectl rollout undo deploy/deploy-demo
```

## You should now be able to…

- [ ] Explain why Deployments are preferred over bare ReplicaSets for app updates.
- [ ] Create a Deployment and wait for rollout with `kubectl rollout status`.
- [ ] Change an image with `kubectl set image` and roll back with `kubectl rollout undo`.
- [ ] Inspect which image version Pods are running.

---

Previous: [ReplicaSet](02-replicaset.md) · Next: [StatefulSet](04-statefulset.md)
