# ReplicaSet

## Explanation

A **ReplicaSet** keeps a fixed number of **identical Pods** running. If a Pod crashes or is deleted, the ReplicaSet starts a replacement so the count stays at the number you asked for.

**Problem it solves:** A single Pod is fragile. You want “always run N copies of this app” without manually watching and recreating Pods.

You pick a **replica count** and a **Pod template** (image, labels, ports). The ReplicaSet continuously compares “how many Pods match my selector?” with “how many I want?” and creates or deletes Pods to match.

## How it relates to other objects

- **Pod** — ReplicaSet creates Pods from its template; each Pod gets labels the ReplicaSet selects.
- **Deployment** — the usual way to manage ReplicaSets (rolling updates, rollbacks). In practice you use Deployment, not ReplicaSet directly.
- **Service** — selects Pods by label; works the same whether Pods came from a ReplicaSet or elsewhere.

See also: [Relationship map](relationship-map.md)

## Hands-on lab

Lab: [../04-hands-on-labs/02-replicaset.md](../04-hands-on-labs/02-replicaset.md)

## You should now be able to…

- Explain what a ReplicaSet guarantees (steady Pod count).
- Describe what happens when one Pod in a ReplicaSet is deleted.
- Explain why Deployments are preferred over managing ReplicaSets by hand.
