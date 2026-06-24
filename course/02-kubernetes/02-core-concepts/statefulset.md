# StatefulSet

## Explanation

A **StatefulSet** runs **stateful** workloads: apps that need **stable network identity** (predictable names like `app-0`, `app-1`) and often **stable storage per replica**.

**Problem it solves:** Deployments treat all Pods as interchangeable. Databases, queues, and clustered software often need “this replica is always `db-0`” and “this replica keeps its own disk.” StatefulSet provides ordered startup, stable hostnames, and optional persistent volumes tied to each replica.

## Example — add to the cluster

**What's new:** Pods named `concept-sts-0`, `concept-sts-1`, each with its **own PVC** (via `volumeClaimTemplates`).

**Before this step:** [PV](pv.md) — storage chain understood.

Save as `~/kube-lab/manifests/concepts/12-statefulset.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: concept-sts-headless
  namespace: kube-lab
spec:
  clusterIP: None
  selector:
    app: concept-sts
  ports:
  - port: 80
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: concept-sts
  namespace: kube-lab
spec:
  serviceName: concept-sts-headless
  replicas: 2
  selector:
    matchLabels:
      app: concept-sts
  template:
    metadata:
      labels:
        app: concept-sts
    spec:
      containers:
      - name: web
        image: nginx:1.27
        ports:
        - containerPort: 80
        volumeMounts:
        - name: data
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

Apply:

```bash
kubectl apply -f ~/kube-lab/manifests/concepts/12-statefulset.yaml
```

### Verify

```bash
kubectl get sts concept-sts
kubectl get pods -l app=concept-sts
kubectl get pvc | grep concept-sts
```

Stable DNS (from another Pod): `concept-sts-0.concept-sts-headless.kube-lab.svc.cluster.local`

### Break & repair

Scale down then up — **same ordinal names** return:

```bash
kubectl scale sts concept-sts --replicas=1
kubectl scale sts concept-sts --replicas=2
kubectl get pods -l app=concept-sts   # concept-sts-0, concept-sts-1 again
```

Next: [Ingress](ingress.md).

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
