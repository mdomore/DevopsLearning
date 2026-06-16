# Deployment

## Explanation

A **Deployment** manages **ReplicaSets** for **stateless** applications — apps where any copy can handle traffic and you do not rely on a fixed Pod name or local disk.

**Problem it solves:** You need to run multiple copies of an app, **update** them safely (rolling update), **roll back** a bad release, and **scale** up or down — without editing Pods one by one.

When you change the Pod template (for example a new image tag), the Deployment creates a new ReplicaSet and gradually shifts traffic to new Pods while retiring old ones.

## How it relates to other objects

- **ReplicaSet** — Deployment owns ReplicaSets; each template change can create a new ReplicaSet.
- **Pod** — ReplicaSets create Pods; you interact with Deployments, not individual Pods, for day-to-day ops.
- **Service** — stable network front-end to Deployment Pods (ClusterIP, NodePort, or via Ingress).
- **ConfigMap / Secret** — referenced in the Deployment’s Pod template for configuration.

See also: [Relationship map](relationship-map.md)

## Hands-on lab

Lab: [../04-hands-on-labs/03-deployment.md](../04-hands-on-labs/03-deployment.md)

## You should now be able to…

- Explain why Deployments are the default choice for stateless web APIs and similar apps.
- Describe rolling updates at a high level (new ReplicaSet, old Pods phased out).
- Name the objects below Deployment in the chain: Deployment → ? → Pod.
