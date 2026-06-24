# StorageClass

## Explanation

A **StorageClass** describes **how** storage is provisioned: which driver, performance tier, reclaim policy, and other parameters. When a PVC requests a StorageClass, the cluster can **create a PV automatically** (dynamic provisioning).

**Problem it solves:** Without StorageClass, someone must create each PV by hand. StorageClass lets the cluster say “claims for `fast-ssd` get NVMe volumes; claims for `standard` get cheaper disks” — matching cloud or local storage backends.

On local lab clusters (`kind`, `minikube`), a default StorageClass often exists so simple PVCs “just work.”

## Example — add to the cluster

**What's new:** you discover how the cluster **provisions disks** — usually no YAML to apply on Docker Desktop / kind.

**Before this step:** [NodePort](nodeport-service.md) — networking track for `concept-web` is complete.

Inspect what already exists:

```bash
kubectl get storageclass
kubectl describe storageclass "$(kubectl get sc -o jsonpath='{.items[0].metadata.name}')"
```

Note the **default** class (annotation `storageclass.kubernetes.io/is-default-class: "true"`). Later PVCs with no `storageClassName` use it.

### Verify

```bash
kubectl get sc -o custom-columns=NAME:.metadata.name,PROVISIONER:.provisioner,DEFAULT:.metadata.annotations.storageclass\.kubernetes\.io/is-default-class
```

### Break & repair

Create a PVC with a **fake** StorageClass name — it stays `Pending`:

```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: concept-bad-pvc
  namespace: kube-lab
spec:
  storageClassName: does-not-exist
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 1Gi
EOF
kubectl get pvc concept-bad-pvc   # Pending
kubectl delete pvc concept-bad-pvc
```

Next: [PVC](pvc.md).

## How it relates to other objects

- **PVC** — `storageClassName` selects which StorageClass to use.
- **PV** — dynamically created PVs reference the StorageClass that provisioned them.
- **StatefulSet** — `volumeClaimTemplates` create PVCs per replica, usually tied to a StorageClass.
- **Pod** — mounts storage through PVC, not StorageClass directly.

See also: [Relationship map](relationship-map.md)

## Hands-on lab

Lab: [../04-hands-on-labs/10-storage-pvc-pv.md](../04-hands-on-labs/10-storage-pvc-pv.md)

## You should now be able to…

- Explain dynamic provisioning vs manually creating PVs.
- Describe what happens when a PVC names a StorageClass.
- List the storage chain: StorageClass → PV ← PVC → Pod mount.
