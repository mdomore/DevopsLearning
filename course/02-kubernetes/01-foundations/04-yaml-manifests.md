# Topic C — YAML manifests (create/update with `kubectl apply`)

**Before this section:** [`kubectl` basics](03-kubectl-basics.md) — you can run `apply`, `get`, and `describe`.

---

## Explanation

Kubernetes objects (Pods, Services, etc.) are usually stored as **YAML** files called **manifests**. YAML is a human-readable text format using indentation (like an outline).

Every manifest shares the same top-level shape:

| Field | Meaning |
|---|---|
| `apiVersion` | Which API version of Kubernetes understands this object (e.g. `v1` for a Pod) |
| `kind` | What type of object this is (`Pod`, `Service`, …) |
| `metadata` | Name, labels, namespace — how the object is identified |
| `spec` | **Desired state** — what you want running (image, replicas, ports, …) |

Two key ideas:

- **Idempotency:** `kubectl apply` can be run many times; Kubernetes reconciles toward the same desired state instead of duplicating objects.
- **Server-side validation:** Wrong `apiVersion`, missing `spec`, or invalid fields → the API rejects the file with an error message (better than silent failure).

## Implementation (minimal Pod YAML)

Create a Pod using a `command`:

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: yaml-pod
spec:
  containers:
    - name: box
      image: busybox:1.36
      command: ["sh", "-c", "echo READY && sleep 3600"]
EOF
```

Verify:

```bash
kubectl get pod yaml-pod -o wide
kubectl logs yaml-pod
```

Now update it (change a label and the command output):

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: yaml-pod
  labels:
    stage: v2
spec:
  containers:
    - name: box
      image: busybox:1.36
      command: ["sh", "-c", "echo UPDATED && sleep 3600"]
EOF
```

Verification:

```bash
kubectl get pod yaml-pod --show-labels
kubectl describe pod yaml-pod | grep -n "UPDATED\|stage"
```

## Break & repair

**Break idea 1: Missing required field**

Try to apply invalid YAML (missing `spec`):

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: yaml-broken
EOF
```

Expected: `kubectl` prints a validation error.

Repair:

```bash
kubectl delete pod yaml-broken --ignore-not-found
```

Re-apply a correct Pod:

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: yaml-broken
spec:
  containers:
    - name: box
      image: busybox:1.36
      command: ["sh", "-c", "sleep 3600"]
EOF
```

**Break idea 2: Wrong apiVersion/kind**

Apply a Pod but with a wrong `apiVersion`:

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: v2
kind: Pod
metadata:
  name: yaml-wrong-apiversion
spec:
  containers:
    - name: box
      image: busybox:1.36
      command: ["sh", "-c", "sleep 3600"]
EOF
```

Repair:

```bash
kubectl delete pod yaml-wrong-apiversion --ignore-not-found
```

Re-apply correctly:

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: yaml-wrong-apiversion
spec:
  containers:
    - name: box
      image: busybox:1.36
      command: ["sh", "-c", "sleep 3600"]
EOF
```

---

## You should now be able to…

- Name the four top-level YAML fields (`apiVersion`, `kind`, `metadata`, `spec`) and what each means
- Create and update a Pod with `kubectl apply` twice without creating duplicates
- Read a validation error when `spec` or `apiVersion` is wrong

Previous: [`kubectl` basics](03-kubectl-basics.md) · Next: [Completion checklist](05-completion-checklist.md)
