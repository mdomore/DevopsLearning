# PV (PersistentVolume)

## Explanation

A **PersistentVolume (PV)** represents a **piece of storage** in the cluster — like a disk volume — that exists independently of any single Pod.

**Problem it solves:** Pod storage is ephemeral by default. When a Pod dies, its local files are gone. PV is the cluster-level “actual storage” that can outlive a Pod and be reused when a new Pod mounts the same claim.

A PV can be created by an admin (static) or automatically by a **StorageClass** (dynamic provisioning).

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
