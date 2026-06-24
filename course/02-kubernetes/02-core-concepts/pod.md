# Pod

## Explanation

A **Pod** is the smallest thing Kubernetes runs. It is usually one container (sometimes a few that must share storage or network tightly).

**Problem it solves:** You need a standard way to tell the cluster “run this container image with these settings.” The Pod is that unit — image, ports, environment, volumes, and restart behavior in one object.

Pods are **ephemeral** (temporary): if a node fails or a Pod is deleted, it is gone. Kubernetes creates a new Pod; it does not “repair” the old one in place. That is why we rarely manage Pods by hand — higher-level objects (ReplicaSet, Deployment) create and replace them for us.

## Example — add to the cluster

**What's new:** your first runnable workload in `kube-lab`.

**Before this step:** [Namespace](namespace.md) — `kube-lab` exists.

Save as `~/kube-lab/manifests/concepts/01-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: concept-pod
  namespace: kube-lab
  labels:
    app: concept-demo
spec:
  containers:
  - name: web
    image: nginx:1.27
    ports:
    - containerPort: 80
```

Apply:

```bash
mkdir -p ~/kube-lab/manifests/concepts
kubectl apply -f ~/kube-lab/manifests/concepts/01-pod.yaml
```

### Verify

```bash
kubectl get pod concept-pod
kubectl describe pod concept-pod
```

### Break & repair

Delete the Pod — it does **not** come back (no controller yet):

```bash
kubectl delete pod concept-pod
kubectl get pod concept-pod   # NotFound — expected
kubectl apply -f ~/kube-lab/manifests/concepts/01-pod.yaml   # recreate manually
```

Next: [ReplicaSet](replicaset.md) · Stack order: [progressive-stack.md](progressive-stack.md)

## How it relates to other objects

- **ReplicaSet / Deployment / StatefulSet** create and manage Pods (you declare how many; they keep Pods running).
- **Service** sends traffic to Pods selected by **labels** on the Pod.
- **ConfigMap / Secret** inject configuration into Pods.
- **ServiceAccount** gives a Pod an identity inside the cluster.
- **PVC** can mount persistent storage into a Pod.

See also: [Relationship map](relationship-map.md)

## Hands-on lab

Lab: [../04-hands-on-labs/01-pod.md](../04-hands-on-labs/01-pod.md)

## You should now be able to…

- Explain what a Pod is and why it is the basic runnable unit in Kubernetes.
- Describe why Pods are replaced rather than “fixed” when something goes wrong.
- Name at least two other Kubernetes objects that create or connect to Pods.
