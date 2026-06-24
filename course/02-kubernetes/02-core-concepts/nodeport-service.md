# NodePort Service

## Explanation

A **NodePort** Service exposes your app on a **fixed port on every node** (typically `30000–32767`), in addition to the internal ClusterIP.

**Problem it solves:** For labs and simple setups, you need to reach a Service from your laptop without cloud LoadBalancers or a full Ingress setup. NodePort opens the same port on each node’s IP so `http://<node-ip>:<nodePort>` reaches the Service.

Trade-off: quick external access, but not ideal for production (wide exposure, non-standard ports, no L7 routing by host/path).

## Example — add to the cluster

**What's new:** the same Service is reachable from your laptop via a high port on the node.

**Before this step:** [ClusterIP Service](clusterip-service.md) — Service `concept-web` exists.

Change type (keeps ClusterIP behavior inside the cluster):

```bash
kubectl patch svc concept-web -n kube-lab -p '{"spec":{"type":"NodePort"}}'
NODE_PORT=$(kubectl get svc concept-web -o jsonpath='{.spec.ports[0].nodePort}')
echo "NodePort: $NODE_PORT"
```

### Verify

Docker Desktop — node IP is usually `127.0.0.1`:

```bash
curl -s "http://127.0.0.1:${NODE_PORT}"
```

kind — use the node’s internal IP from `kubectl get nodes -o wide`.

### Break & repair

NodePort in range 30000–32767; if you set an invalid port in YAML, `kubectl apply` fails — remove `nodePort:` to let Kubernetes assign one.

Next: [StorageClass](storageclass.md) (storage track).

## How it relates to other objects

- **ClusterIP** — NodePort extends ClusterIP; internal cluster access still works via the ClusterIP.
- **Deployment / Pod** — same label selector model as any Service.
- **Ingress** — preferred for HTTP(S) routing in production; NodePort is a simpler debug/lab path.
- **LoadBalancer** — cloud option that provisions an external IP; NodePort is the “manual” alternative on bare local clusters.

See also: [Relationship map](relationship-map.md)

## Hands-on lab

Lab: [../04-hands-on-labs/09-nodeport-service.md](../04-hands-on-labs/09-nodeport-service.md)

## You should now be able to…

- Explain how NodePort differs from ClusterIP for reaching an app from outside the cluster.
- Name the typical NodePort range and how you construct a test URL.
- Describe when NodePort is enough vs when Ingress or LoadBalancer is a better fit.
