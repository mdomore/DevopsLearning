# PV (PersistentVolume)

## Explanation

A **PersistentVolume (PV)** represents a **piece of storage** in the cluster — like a disk volume — that exists independently of any single Pod.

**Problem it solves:** Pod storage is ephemeral by default. When a Pod dies, its local files are gone. PV is the cluster-level “actual storage” that can outlive a Pod and be reused when a new Pod mounts the same claim.

A PV can be created by an admin (static) or automatically by a **StorageClass** (dynamic provisioning).

## Example — add to the cluster

**What's new:** you see the **actual volume** bound to `concept-pvc` (dynamic provisioning — you did not create the PV by hand).

**Before this step:** [PVC](pvc.md) — `concept-pvc` status `Bound`.

Observe the binding:

```bash
kubectl get pvc concept-pvc
kubectl get pv
kubectl describe pvc concept-pvc | grep -E 'Volume:|StorageClass'
```

The PV name is auto-generated. `concept-pvc` ↔ one PV is a 1:1 bind while in use.

### Verify

```bash
PV=$(kubectl get pvc concept-pvc -o jsonpath='{.spec.volumeName}')
kubectl get pv "$PV" -o wide
```

### Break & repair

PVC stuck `Pending` — usually no StorageClass or no capacity. Fix the PVC spec or install a default StorageClass, then re-check events:

```bash
kubectl describe pvc concept-pvc
```

Next: [StatefulSet](statefulset.md).

## How it relates to other objects

- **PVC** — a workload **claims** storage; PV is bound to a matching PVC.
- **StorageClass** — defines how new PVs are provisioned dynamically.
- **Pod / StatefulSet** — mount storage via PVC; they do not reference PV directly in most app YAML.
- **Deployment** — usually stateless; StatefulSet + PVC is the common persistent pattern.

See also: [Relationship map](relationship-map.md)

## Hands-on lab

Lab: [../04-hands-on-labs/10-storage-pvc-pv.md](../04-hands-on-labs/10-storage-pvc-pv.md)

## You should now be able to…

- Explain that a PV is cluster storage and survives individual Pod restarts.
- Describe the binding relationship between PV and PVC.
- Explain when an admin creates PVs vs when StorageClass creates them automatically.
