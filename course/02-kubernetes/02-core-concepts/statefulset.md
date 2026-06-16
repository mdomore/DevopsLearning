# StatefulSet

## Explanation

A **StatefulSet** runs **stateful** workloads: apps that need **stable network identity** (predictable names like `app-0`, `app-1`) and often **stable storage per replica**.

**Problem it solves:** Deployments treat all Pods as interchangeable. Databases, queues, and clustered software often need “this replica is always `db-0`” and “this replica keeps its own disk.” StatefulSet provides ordered startup, stable hostnames, and optional persistent volumes tied to each replica.

## How it relates to other objects

- **Pod** — StatefulSet creates Pods with stable names and ordinal indexes.
- **Deployment** — use for stateless apps; StatefulSet when identity or per-replica storage matters.
- **Service** — usually a **headless** Service (`clusterIP: None`) gives each Pod a stable DNS name.
- **PVC** — often created per replica via `volumeClaimTemplates` so storage follows the Pod identity.

See also: [Relationship map](relationship-map.md)

## Hands-on lab

Lab: [../04-hands-on-labs/04-statefulset.md](../04-hands-on-labs/04-statefulset.md)

## You should now be able to…

- Explain when to choose StatefulSet instead of Deployment.
- Describe stable Pod naming and why it matters for clustered software.
- Explain how per-replica storage often connects to PVCs in a StatefulSet.
