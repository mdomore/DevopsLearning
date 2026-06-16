# Lab 9 — NodePort Service

## Explanation

A **ClusterIP** Service is only reachable **inside** the cluster. Sometimes you need traffic from **outside** — your laptop, another machine on the network — without an Ingress controller yet.

A **NodePort** Service opens a **static port on every node’s IP** (high port range, typically 30000–32767) and forwards to your Pods. You reach the app via `http://<NODE_IP>:<NODE_PORT>`.

**Problem it solves:** Quick external access for labs and debugging (not usually the production pattern — Ingress or cloud load balancers are common there).

**Why this lab?** Extends [Lab 8 — ClusterIP](08-clusterip-service.md) with one more Service type so you can compare internal vs node-level access.

**Prerequisites**

- [Phase 0 tooling](../../00-getting-started/00-tooling-setup.md) — kind cluster `kube-lab`.
- [Lab 0 — Setup](00-setup.md) — namespace `kube-lab`.
- [Lab 3 — Deployment](03-deployment.md) — `deploy-demo` running.
- Concept recap: [NodePort Service](../02-core-concepts/nodeport-service.md).

**Shell:** local terminal, `kubectl` → kind cluster, namespace `kube-lab`.

## Key fields (before you apply)

| Flag | What it controls |
|---|---|
| `--type=NodePort` | Allocates a node port in addition to ClusterIP behavior. |
| `--port=80` | Service port (also used inside cluster). |
| `--target-port=80` | Container port on Pods. |
| `spec.ports[0].nodePort` | High port on each node — what you use from outside. |

On **kind**, the “node IP” is often the Docker host IP or you use `kind` port mappings from Phase 0 — if `curl` to the node IP fails, check your kind setup docs.

## Implementation

Create the NodePort Service:

```bash
kubectl expose deploy deploy-demo --name=deploy-demo-nodeport --port=80 --target-port=80 --type=NodePort
```

Discover the assigned node port and node IP:

```bash
kubectl get svc deploy-demo-nodeport
kubectl get svc deploy-demo-nodeport -o jsonpath='{.spec.ports[0].nodePort}{"\n"}'
kubectl get nodes -o wide
```

Note `<NODE_PORT>` from jsonpath and `<NODE_IP>` from the `INTERNAL-IP` (or external IP if shown) column.

Then from your machine (same terminal, outside the cluster network path):

```bash
curl http://<NODE_IP>:<NODE_PORT>
```

Replace placeholders with your values. You should see nginx welcome HTML.

## Verification

```bash
kubectl get svc deploy-demo-nodeport
```

- `TYPE` is **NodePort**.
- `PORT(S)` shows something like `80:3xxxx/TCP` — the `3xxxx` part is the node port.

```bash
curl -sS -o /dev/null -w '%{http_code}\n' http://<NODE_IP>:<NODE_PORT>
```

- Returns **200**.

## Break & repair

**Symptom:** `curl http://<NODE_IP>:<NODE_PORT>` times out or connection refused from your laptop.

**Cause:** Wrong node IP (kind nodes are Docker containers — you may need the host IP and kind’s published port), or firewall blocking the port.

**Repair:**

```bash
kubectl get svc deploy-demo-nodeport -o wide
kubectl get endpoints deploy-demo-nodeport
docker ps | grep kube-lab
```

- Confirm endpoints are not empty (same as ClusterIP lab).
- On kind, try the control-plane node `INTERNAL-IP` or follow [Phase 0 tooling](../../00-getting-started/00-tooling-setup.md) notes for reaching node ports.
- As a cluster-internal sanity check:

```bash
kubectl run curl-client --image=curlimages/curl:8.8.0 --restart=Never -it --rm -- curl -sS -o /dev/null -w '%{http_code}\n' http://deploy-demo-nodeport
```

If that returns 200 but host `curl` fails, the issue is host→node routing (kind networking), not the Service itself.

## You should now be able to…

- [ ] Explain ClusterIP vs NodePort (internal only vs port on every node).
- [ ] Find the auto-assigned `nodePort` with kubectl.
- [ ] Map a NodePort Service to node IP + port for external access.
- [ ] Use endpoints to confirm backends are registered.

---

Previous: [ClusterIP Service](08-clusterip-service.md) · Next: [StorageClass + PVC + PV](10-storage-pvc-pv.md)
