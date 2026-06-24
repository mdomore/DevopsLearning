# 04 — YAML manifests (create/update with `kubectl apply`)

**Before this section:** [kubectl basics](03-kubectl-basics.md) — you can run `apply`, `get`, and `describe`.

Commands run in your **local terminal**, namespace **`kube-lab`**.

---

## Explanation

A **manifest** is a YAML file that describes one Kubernetes object (here: a **Pod**).

Every Pod manifest has the same four blocks:

```text
apiVersion + kind     → what type of object this is
metadata              → name (and optional labels)
spec                  → what should run (containers, image, …)
```

| Field | Role |
|---|---|
| `apiVersion` | API version for this object type (`v1` for a Pod) |
| `kind` | Resource type — `Pod` |
| `metadata.name` | Name shown in `kubectl get pod` |
| `spec.containers` | List of containers in the Pod (at least one) |

**`kubectl apply -f file.yaml`** sends the file to the cluster. Run it again with the same file → no duplicate object (idempotent).

---

## Implementation — Step 1: Create a folder for manifests

```bash
mkdir -p ~/kube-lab/manifests
cd ~/kube-lab/manifests
```

---

## Implementation — Step 2: Create the file — block by block

Create **`yaml-pod.yaml`**. You can use an editor (`nano yaml-pod.yaml`) or the command below.

We build the file in **four blocks**. Read each block, then add it to your file.

### Block 1 — Object type

```yaml
apiVersion: v1
kind: Pod
```

| Line | Meaning |
|---|---|
| `apiVersion: v1` | Core Kubernetes API for Pod objects |
| `kind: Pod` | This file describes a **Pod** (not a Deployment or Service) |

---

### Block 2 — Name

```yaml
metadata:
  name: yaml-pod
```

| Line | Meaning |
|---|---|
| `metadata.name` | Object name in the current namespace (`kube-lab`) |

---

### Block 3 — Container list

```yaml
spec:
  containers:
    - name: box
      image: busybox:1.36
```

| Line | Meaning |
|---|---|
| `spec` | Desired state — what the cluster should run |
| `containers` | One Pod can have one or more containers; here we use **one** |
| `name: box` | Name of this container inside the Pod |
| `image: busybox:1.36` | Container image to pull (small image with `sh`) |

---

### Block 4 — Command (keep the Pod running)

```yaml
      command: ["sh", "-c", "echo READY && sleep 3600"]
```

| Line | Meaning |
|---|---|
| `command` | Overrides the image default; run a shell |
| `echo READY` | Prints a line you can see with `kubectl logs` |
| `sleep 3600` | Keeps the process alive so STATUS stays **Running** |

Without `sleep`, busybox would exit and the Pod would fail.

---

### Full file (all blocks together)

Your **`yaml-pod.yaml`** should look exactly like this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: yaml-pod
spec:
  containers:
    - name: box
      image: busybox:1.36
      command: ["sh", "-c", "echo READY && sleep 3600"]
```

One-shot create (optional):

```bash
cat <<'EOF' > yaml-pod.yaml
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

Check the file on disk:

```bash
cat yaml-pod.yaml
```

---

## Implementation — Step 3: Apply the manifest

```bash
kubectl apply -f yaml-pod.yaml
```

Expected output contains: `pod/yaml-pod created` (or `configured` if it already existed).

Apply the **same file again**:

```bash
kubectl apply -f yaml-pod.yaml
```

Expected: `unchanged` — same Pod, no duplicate.

---

## Implementation — Step 4: Verify

```bash
kubectl get pod yaml-pod
```

| Column | Expected |
|---|---|
| **NAME** | `yaml-pod` |
| **READY** | `1/1` |
| **STATUS** | `Running` |

```bash
kubectl logs yaml-pod
```

Expected: `READY`

```bash
kubectl describe pod yaml-pod
```

Scroll to **Events** — container should **Start** without `Failed` or `ImagePullBackOff`.

---

## Verification

- [ ] File `~/kube-lab/manifests/yaml-pod.yaml` exists with four blocks (`apiVersion`, `metadata`, `spec`, `command`)
- [ ] `kubectl apply -f yaml-pod.yaml` succeeds
- [ ] `kubectl get pod yaml-pod` → **Running**, **1/1**
- [ ] `kubectl logs yaml-pod` → **READY**
- [ ] Second `apply` does not create a second Pod

---

## Break & repair

**Missing `spec`**

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: yaml-broken
EOF
```

→ Validation error. Fix: add a `spec:` block with at least one container.

**Wrong `apiVersion`**

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: v2
kind: Pod
metadata:
  name: yaml-broken2
spec:
  containers:
    - name: box
      image: busybox:1.36
      command: ["sh", "-c", "sleep 3600"]
EOF
```

→ API rejects the file. Fix: use `apiVersion: v1` for Pod.

Cleanup lab Pods when done:

```bash
kubectl delete pod yaml-pod yaml-broken yaml-broken2 --ignore-not-found
```

---

## Cleanup / revert

```bash
kubectl delete pod yaml-pod --ignore-not-found
```

Keep `~/kube-lab/manifests/yaml-pod.yaml` for reference or delete the folder if you prefer a clean disk.

---

## You should now be able to…

- Write a minimal Pod YAML with `apiVersion`, `kind`, `metadata`, and `spec.containers`
- Explain each line in `yaml-pod.yaml`
- Create the Pod with `kubectl apply -f` and verify with `get`, `logs`, and `describe`

Previous: [kubectl basics](03-kubectl-basics.md) · Next: [Completion checklist](05-completion-checklist.md)
