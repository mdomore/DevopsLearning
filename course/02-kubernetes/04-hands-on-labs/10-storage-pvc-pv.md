# Lab 10 — StorageClass + PVC + PV

## Explanation

Containers are ephemeral — when a Pod is deleted, its filesystem is gone unless you attach **persistent storage**. Kubernetes separates:

- **PersistentVolume (PV)** — a piece of storage in the cluster (like a disk).
- **PersistentVolumeClaim (PVC)** — a Pod’s **request** for storage (“give me 1Gi ReadWriteOnce”).
- **StorageClass** — tells the cluster **how** to provision PVs automatically when someone creates a PVC.

**Problem it solves:** Data survives Pod restarts; apps can write files that persist.

**Why this lab?** You saw PVCs auto-created in [Lab 4 — StatefulSet](04-statefulset.md). Here you claim storage manually and mount it into a simple Pod.

**Prerequisites**

- [Phase 0 tooling](../../00-getting-started/00-tooling-setup.md) — kind cluster `kube-lab`.
- [Lab 0 — Setup](00-setup.md) — namespace `kube-lab`.
- Concept recap: [StorageClass](../02-core-concepts/storageclass.md), [PVC](../02-core-concepts/pvc.md), [PV](../02-core-concepts/pv.md).

**Shell:** local terminal, `kubectl` → kind cluster, namespace `kube-lab`.

## Key fields (before you apply)

**PVC:**

| Field | What it controls |
|---|---|
| `accessModes: ReadWriteOnce` | One node can mount for read/write at a time. |
| `resources.requests.storage: 1Gi` | Size requested; StorageClass provisions a matching PV. |

**Pod volume:**

| Field | What it controls |
|---|---|
| `volumes[].persistentVolumeClaim.claimName` | Which PVC to attach. |
| `volumeMounts[].mountPath` | Path inside container (`/data`). |

## Implementation

See which StorageClasses exist (kind usually has `standard` as default):

```bash
kubectl get storageclass
```

Create PVC and Pod:

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-demo
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: pvc-demo-pod
spec:
  containers:
    - name: box
      image: busybox:1.36
      command: ["sh", "-c", "echo hello > /data/hello.txt; sleep 3600"]
      volumeMounts:
        - name: data
          mountPath: /data
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: pvc-demo
EOF
```

Check binding and read the file:

```bash
kubectl get pvc pvc-demo
kubectl get pv
kubectl exec -it pvc-demo-pod -- sh -c 'cat /data/hello.txt'
```

## Verification

```bash
kubectl get pvc pvc-demo
```

- `STATUS` is **Bound**, `CAPACITY` shows **1Gi**.

```bash
kubectl get pv
```

- At least one PV bound to claim `kube-lab/pvc-demo` (namespace/claim name in output).

```bash
kubectl exec -it pvc-demo-pod -- sh -c 'cat /data/hello.txt'
```

- Prints: `hello`.

## Break & repair

**Symptom:** PVC stays `Pending`; Pod `pvc-demo-pod` stays `Pending`.

**Cause:** No default StorageClass, or storage request exceeds what the provisioner allows.

**Repair:**

```bash
kubectl describe pvc pvc-demo
kubectl get storageclass
```

- If no default class, set one or add `storageClassName: standard` under PVC `spec` (match your cluster’s class name).
- After fixing StorageClass, delete and recreate PVC/Pod if needed:

```bash
kubectl delete pod pvc-demo-pod
kubectl delete pvc pvc-demo
```

Then re-run the apply block.

## You should now be able to…

- [ ] Explain PV vs PVC vs StorageClass in plain terms.
- [ ] Create a PVC and mount it in a Pod with `persistentVolumeClaim`.
- [ ] Confirm `Bound` status on PVC and matching PV.
- [ ] Read a file from a mounted volume with `kubectl exec`.

---

Previous: [NodePort Service](09-nodeport-service.md) · Next: [Ingress](11-ingress.md)
