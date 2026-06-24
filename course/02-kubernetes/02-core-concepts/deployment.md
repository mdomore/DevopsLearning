# Deployment

## Explanation

A **Deployment** manages **ReplicaSets** for **stateless** applications — apps where any copy can handle traffic and you do not rely on a fixed Pod name or local disk.

**Problem it solves:** You need to run multiple copies of an app, **update** them safely (rolling update), **roll back** a bad release, and **scale** up or down — without editing Pods one by one.

When you change the Pod template (for example a new image tag), the Deployment creates a new ReplicaSet and gradually shifts traffic to new Pods while retiring old ones.

## Example — add to the cluster

**What's new:** a **Deployment** becomes the main app (`concept-web`); rolling updates and rollbacks later.

**Before this step:** [ReplicaSet](replicaset.md). Optional cleanup so labels do not overlap:

```bash
kubectl delete pod concept-pod --ignore-not-found
kubectl delete replicaset concept-rs --ignore-not-found
```

Save as `~/kube-lab/manifests/concepts/03-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: concept-web
  namespace: kube-lab
spec:
  replicas: 2
  selector:
    matchLabels:
      app: concept-web
  template:
    metadata:
      labels:
        app: concept-web
    spec:
      containers:
      - name: web
        image: nginx:1.27
        ports:
        - containerPort: 80
```

Apply:

```bash
kubectl apply -f ~/kube-lab/manifests/concepts/03-deployment.yaml
```

### Verify

```bash
kubectl get deploy concept-web
kubectl get pods -l app=concept-web
kubectl get rs -l app=concept-web
```

### Break & repair

Change the image tag and watch a rolling update:

```bash
kubectl set image deployment/concept-web web=nginx:1.27-alpine
kubectl rollout status deployment/concept-web
kubectl rollout undo deployment/concept-web   # rollback
```

Later steps patch this same Deployment. Next: [ConfigMap](configmap.md).

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
