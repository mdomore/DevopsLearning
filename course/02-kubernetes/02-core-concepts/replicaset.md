# ReplicaSet

## Explanation

A **ReplicaSet** keeps a fixed number of **identical Pods** running. If a Pod crashes or is deleted, the ReplicaSet starts a replacement so the count stays at the number you asked for.

**Problem it solves:** A single Pod is fragile. You want “always run N copies of this app” without manually watching and recreating Pods.

You pick a **replica count** and a **Pod template** (image, labels, ports). The ReplicaSet continuously compares “how many Pods match my selector?” with “how many I want?” and creates or deletes Pods to match.

## Example — add to the cluster

**What's new:** Kubernetes keeps **2 Pods** running — if one dies, a replacement appears.

**Before this step:** [Pod](pod.md) — optional: keep `concept-pod` to compare manual vs managed.

Save as `~/kube-lab/manifests/concepts/02-replicaset.yaml`:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: concept-rs
  namespace: kube-lab
spec:
  replicas: 2
  selector:
    matchLabels:
      app: concept-rs
  template:
    metadata:
      labels:
        app: concept-rs
    spec:
      containers:
      - name: web
        image: nginx:1.27
        ports:
        - containerPort: 80
```

Apply:

```bash
kubectl apply -f ~/kube-lab/manifests/concepts/02-replicaset.yaml
```

### Verify

```bash
kubectl get rs concept-rs
kubectl get pods -l app=concept-rs
```

### Break & repair

Delete one Pod — the ReplicaSet recreates it:

```bash
POD=$(kubectl get pod -l app=concept-rs -o jsonpath='{.items[0].metadata.name}')
kubectl delete pod "$POD"
kubectl get pods -l app=concept-rs   # back to 2
```

Next: [Deployment](deployment.md) replaces this pattern for day-to-day use.

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
