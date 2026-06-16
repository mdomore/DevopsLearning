# PVC (PersistentVolumeClaim)

## Explanation

A **PersistentVolumeClaim (PVC)** is a **request for storage** from an app’s point of view: “I need 5 GiB, read-write-once, preferably fast SSD.”

**Problem it solves:** Developers should not hard-code disk IDs or cloud volume names. They declare what they need in a PVC; Kubernetes binds it to a suitable **PV** (or creates one via **StorageClass**).

Pods reference the PVC by name in a `volume`; the mounted path looks like a normal directory inside the container.

## How it relates to other objects

- **PV** — PVC binds to exactly one PV when storage is allocated (in the common static/dynamic models).
- **StorageClass** — PVC can name a class so the right type of volume is provisioned.
- **Pod / StatefulSet** — consume PVCs through `volumes` and `volumeMounts`.
- **Deployment** — rarely uses PVCs for app data; StatefulSet + PVC is typical for databases.

See also: [Relationship map](relationship-map.md)

## Hands-on lab

Lab: [../04-hands-on-labs/10-storage-pvc-pv.md](../04-hands-on-labs/10-storage-pvc-pv.md)

## You should now be able to…

- Explain a PVC in plain language (“app asks for storage”).
- Name the main fields you declare: size, access mode, optional StorageClass.
- Trace the chain: Pod → PVC → PV.
