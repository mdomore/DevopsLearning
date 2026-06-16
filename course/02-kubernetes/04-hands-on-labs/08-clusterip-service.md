# Lab 8 — ClusterIP Service

## Explanation

Pods come and go — names and IPs change when they restart. Clients need a **stable address** that always reaches **one of the healthy backend Pods**. A **Service** is that stable front door inside the cluster.

A **ClusterIP** Service gets a virtual IP **only reachable from inside the cluster** (other Pods, `kubectl` debug pods). It is the default Service type for internal traffic.

**Problem it solves:** Load-balance traffic across Pod replicas without hard-coding Pod IPs.

**Why this lab?** You created `deploy-demo` in [Lab 3](03-deployment.md). Here you expose it on port 80 inside the cluster — foundation for NodePort and Ingress later.

**Prerequisites**

- [Phase 0 tooling](../../00-getting-started/00-tooling-setup.md) — kind cluster `kube-lab`.
- [Lab 0 — Setup](00-setup.md) — namespace `kube-lab`.
- [Lab 3 — Deployment](03-deployment.md) — `deploy-demo` must exist with label `app: deploy-demo`.
- Concept recap: [ClusterIP Service](../02-core-concepts/clusterip-service.md).

**Shell:** local terminal, `kubectl` → kind cluster, namespace `kube-lab`.

## Key fields (before you apply)

`kubectl expose` builds a Service from an existing Deployment:

| Flag | What it controls |
|---|---|
| `deploy deploy-demo` | Source workload — Service selector matches Deployment Pod labels. |
| `--name=deploy-demo-clusterip` | Service object name. |
| `--port=80` | Port clients use on the Service IP / DNS name. |
| `--target-port=80` | Port on the Pod container (`containerPort`). |
| `--type=ClusterIP` | Internal-only virtual IP (default type). |

**Endpoints** — the set of Pod IPs behind the Service; empty endpoints mean no matching Pods.

## Implementation

Expose the Deployment:

```bash
kubectl expose deploy deploy-demo --name=deploy-demo-clusterip --port=80 --target-port=80 --type=ClusterIP
```

Inspect Service and backends:

```bash
kubectl get svc deploy-demo-clusterip
kubectl get endpoints deploy-demo-clusterip
```

Test from a temporary curl Pod inside the cluster (`--rm` deletes the test Pod when curl finishes):

```bash
kubectl run curl-client --image=curlimages/curl:8.8.0 --restart=Never -it --rm -- curl -sS http://deploy-demo-clusterip
```

You should see nginx HTML (starts with `<!DOCTYPE html>`).

## Verification

```bash
kubectl get svc deploy-demo-clusterip
```

- `TYPE` is **ClusterIP**, `PORT(S)` shows **80**.

```bash
kubectl get endpoints deploy-demo-clusterip
```

- `ENDPOINTS` lists two Pod IPs (matches `deploy-demo` replica count), not empty.

```bash
kubectl run curl-client --image=curlimages/curl:8.8.0 --restart=Never -it --rm -- curl -sS -o /dev/null -w '%{http_code}\n' http://deploy-demo-clusterip
```

- HTTP status **200**.

## Break & repair

**Symptom:** `curl` fails with connection errors; `kubectl get endpoints deploy-demo-clusterip` shows **no addresses**.

**Cause:** Service selector does not match Pod labels (for example Deployment recreated with different `app` label), or no Pods are Ready.

**Repair:**

```bash
kubectl describe svc deploy-demo-clusterip
kubectl get pods -l app=deploy-demo
kubectl get endpoints deploy-demo-clusterip
```

- Confirm selector `app=deploy-demo` matches Pod labels.
- Fix Deployment labels or delete and re-expose:

```bash
kubectl delete svc deploy-demo-clusterip
kubectl expose deploy deploy-demo --name=deploy-demo-clusterip --port=80 --target-port=80 --type=ClusterIP
```

## You should now be able to…

- [ ] Explain what a ClusterIP Service provides (stable internal IP/DNS + load balancing).
- [ ] Create a Service with `kubectl expose` from a Deployment.
- [ ] Read `endpoints` to see which Pods receive traffic.
- [ ] Curl a Service by name from another Pod in the cluster.

---

Previous: [ServiceAccount](07-serviceaccount.md) · Next: [NodePort Service](09-nodeport-service.md)
