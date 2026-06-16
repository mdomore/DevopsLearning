# Lab 4 — StatefulSet

## Explanation

A **Deployment** treats Pods as interchangeable — any copy can serve traffic. Some apps need **stable identity** and **stable storage per Pod** (for example databases or clustered software where Pod-0 and Pod-1 are not the same).

A **StatefulSet** gives each Pod a **predictable name** (`sts-demo-0`, `sts-demo-1`, …), ordered start/stop, and **per-Pod persistent volumes** via volume claim templates.

**Problem it solves:** “Each replica needs its own disk and a stable hostname,” not “any random Pod from a pool.”

**Why this lab?** You have seen stateless nginx on Deployments. StatefulSets show the other common workload pattern plus how storage binds to a specific Pod.

**Prerequisites**

- [Phase 0 tooling](../../00-getting-started/00-tooling-setup.md) — kind cluster `kube-lab`.
- [Lab 0 — Setup](00-setup.md) — namespace `kube-lab`.
- [Lab 3 — Deployment](03-deployment.md) — Pod templates and labels.
- Concept recap: [StatefulSet](../02-core-concepts/statefulset.md), [PVC](../02-core-concepts/pvc.md).

**Shell:** local terminal, `kubectl` → kind cluster, namespace `kube-lab`.

## Key fields (before you apply)

**Headless Service** (`clusterIP: None`):

| Field | What it controls |
|---|---|
| `kind: Service` | Network identity for StatefulSet Pods. |
| `clusterIP: None` | “Headless” — no single virtual IP; DNS returns individual Pod IPs. |
| `selector.app` | Routes to Pods with matching label. |

**StatefulSet:**

| Field | What it controls |
|---|---|
| `serviceName` | Must match the headless Service name — links stable network identity. |
| `replicas: 2` | Number of Pods (named `sts-demo-0`, `sts-demo-1`). |
| `volumeClaimTemplates` | Creates one **PVC** per Pod automatically (`data-sts-demo-0`, …). |
| `volumeMounts` | Mounts that PVC into the container at `/usr/share/nginx/html`. |
| `accessModes: ReadWriteOnce` | Volume can attach to one node at a time (typical for block storage). |

## Implementation

Apply the headless Service and StatefulSet:

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: sts-demo
spec:
  clusterIP: None
  selector:
    app: sts-demo
  ports:
    - port: 80
      targetPort: 80
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sts-demo
spec:
  serviceName: sts-demo
  replicas: 2
  selector:
    matchLabels:
      app: sts-demo
  template:
    metadata:
      labels:
        app: sts-demo
    spec:
      containers:
        - name: web
          image: nginx:1.27
          volumeMounts:
            - name: data
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi
EOF
```

Inspect StatefulSet, Pods, and PVCs:

```bash
kubectl get sts sts-demo
kubectl get pods -l app=sts-demo
kubectl get pvc | grep data-sts-demo
```

(`rg` is a fast search tool; if you do not have it, use `kubectl get pvc` and look for names starting with `data-sts-demo`.)

## Verification

```bash
kubectl get sts sts-demo
```

- `READY` is **2/2**.

```bash
kubectl get pods -l app=sts-demo
```

- Pods named `sts-demo-0` and `sts-demo-1`, both `Running`.

```bash
kubectl get pvc
```

- Two claims such as `data-sts-demo-0` and `data-sts-demo-1`, status **Bound**.

## Break & repair

**Symptom:** StatefulSet Pods stay `Pending`; `kubectl describe pod sts-demo-0` mentions PVC or scheduling issues.

**Cause:** `serviceName` does not match the headless Service name, or the cluster has no default **StorageClass** to provision volumes.

**Repair:**

```bash
kubectl get storageclass
kubectl describe pod sts-demo-0
```

- Confirm a default StorageClass exists (kind usually provides `standard`).
- Confirm `serviceName: sts-demo` matches `metadata.name` of the Service above.
- Fix YAML and re-apply, or delete the StatefulSet and PVCs if you need a clean retry:

```bash
kubectl delete sts sts-demo
kubectl delete pvc data-sts-demo-0 data-sts-demo-1
```

Then apply the YAML block again.

## You should now be able to…

- [ ] Explain when StatefulSet beats Deployment (stable identity + per-replica storage).
- [ ] Describe what a headless Service (`clusterIP: None`) is for.
- [ ] Read `volumeClaimTemplates` and predict PVC names (`data-sts-demo-0`, …).
- [ ] List StatefulSet Pods and their bound PVCs.

---

Previous: [Deployment](03-deployment.md) · Next: [ConfigMap](05-configmap.md)
