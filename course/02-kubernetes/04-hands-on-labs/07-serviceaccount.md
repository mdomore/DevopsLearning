# Lab 7 — ServiceAccount

## Explanation

When a Pod talks to the Kubernetes API (or when the cluster identifies “who” a Pod is), it uses a **ServiceAccount** — an identity for workloads inside the cluster. Every namespace has a **default** ServiceAccount; you can create custom ones for finer control.

**Problem it solves:** “Which identity should this Pod use?” — needed for API access, some security policies, and cloud IAM bindings in later chapters.

**Why this lab?** Before [Services](08-clusterip-service.md) (network access), you see **workload identity** — a different axis from networking.

**Prerequisites**

- [Phase 0 tooling](../../00-getting-started/00-tooling-setup.md) — kind cluster `kube-lab`.
- [Lab 0 — Setup](00-setup.md) — namespace `kube-lab`.
- [Lab 1 — Pod](01-pod.md) — Pod spec basics.
- Concept recap: [ServiceAccount](../02-core-concepts/serviceaccount.md).

**Shell:** local terminal, `kubectl` → kind cluster, namespace `kube-lab`.

## Key fields (before you apply)

| Field / command | What it controls |
|---|---|
| `kubectl create serviceaccount sa-demo` | Creates identity object `sa-demo` in current namespace. |
| `spec.serviceAccountName` | Pod field — which ServiceAccount this Pod runs as (not the default). |
| Default behavior | If omitted, Pod uses `default` ServiceAccount in the namespace. |

ServiceAccounts are often paired with **Roles** and **RoleBindings** (RBAC) — not in this short lab, but that is how permissions attach to an identity.

## Implementation

Create a ServiceAccount:

```bash
kubectl create serviceaccount sa-demo
```

Create a Pod that uses it:

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: sa-demo-pod
spec:
  serviceAccountName: sa-demo
  containers:
    - name: box
      image: busybox:1.36
      command: ["sh", "-c", "sleep 3600"]
EOF
```

Verify:

```bash
kubectl get sa sa-demo
kubectl get pod sa-demo-pod -o jsonpath='{.spec.serviceAccountName}{"\n"}'
```

## Verification

```bash
kubectl get sa sa-demo
```

- ServiceAccount `sa-demo` exists in namespace `kube-lab`.

```bash
kubectl get pod sa-demo-pod -o jsonpath='{.spec.serviceAccountName}{"\n"}'
```

- Prints: `sa-demo` (not `default`).

## Break & repair

**Symptom:** Pod runs but `serviceAccountName` shows `default`, or Pod events mention ServiceAccount not found.

**Cause:** Wrong name in `serviceAccountName`, or ServiceAccount created after Pod (Pod already bound to default).

**Repair:**

```bash
kubectl get sa sa-demo
kubectl describe pod sa-demo-pod
```

- Fix YAML to match exact ServiceAccount name.
- Delete and recreate Pod after ServiceAccount exists:

```bash
kubectl delete pod sa-demo-pod
```

Then re-apply the Pod YAML with `serviceAccountName: sa-demo`.

## You should now be able to…

- [ ] Explain what a ServiceAccount is (in-cluster workload identity).
- [ ] Create a ServiceAccount and assign it to a Pod.
- [ ] Read `serviceAccountName` from a live Pod spec.
- [ ] Contrast ServiceAccount (identity) with Service (networking).

---

Previous: [Secret](06-secret.md) · Next: [ClusterIP Service](08-clusterip-service.md)
