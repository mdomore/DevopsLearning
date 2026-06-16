# Lab 11 — Ingress

## Explanation

**NodePort** exposes one app per high port — awkward at scale. **Ingress** gives **HTTP routing** by hostname and path (for example `demo.local` → your Service) on a shared entry point.

An **Ingress** resource is only rules; you need an **Ingress Controller** (nginx, traefik, etc.) running in the cluster to enforce them. The controller listens on an external IP or localhost port and forwards to **Services** (usually ClusterIP).

**Problem it solves:** Friendly URLs and path-based routing without opening a NodePort per app.

**Why this lab?** Uses `deploy-demo-clusterip` from [Lab 8](08-clusterip-service.md) as the backend — completes the path Pod → Deployment → Service → Ingress.

**Prerequisites**

- [Phase 0 tooling](../../00-getting-started/00-tooling-setup.md) — kind cluster `kube-lab`.
- [Lab 0 — Setup](00-setup.md) — namespace `kube-lab`.
- [Lab 8 — ClusterIP Service](08-clusterip-service.md) — Service `deploy-demo-clusterip` must exist.
- **Ingress Controller** installed on your cluster (many kind setups install one in Phase 0 or via a addon — verify below).
- Concept recap: [Ingress](../02-core-concepts/ingress.md).

**Shell:** local terminal, `kubectl` → kind cluster, namespace `kube-lab`.

## Key fields (before you apply)

| Field | What it controls |
|---|---|
| `rules[].host: demo.local` | HTTP requests with `Host: demo.local` match this rule. |
| `paths[].path: /` | URL path prefix (`/` = all paths). |
| `pathType: Prefix` | Match path and everything under it. |
| `backend.service.name` | ClusterIP Service to forward to (`deploy-demo-clusterip`). |
| `backend.service.port.number` | Service port (80). |

Hostname `demo.local` is arbitrary for the lab — your client must send that Host header (or map the name in `/etc/hosts`).

## Implementation

Confirm an Ingress Controller is running:

```bash
kubectl get pods -A | grep -i ingress
```

If nothing appears, install a controller for your environment (kind docs often use `ingress-nginx`) before continuing.

Create the Ingress:

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: deploy-demo-ingress
spec:
  rules:
    - host: demo.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: deploy-demo-clusterip
                port:
                  number: 80
EOF
```

Inspect routing status:

```bash
kubectl get ingress deploy-demo-ingress
kubectl describe ingress deploy-demo-ingress
```

From `kubectl get ingress` or `describe`, note `<INGRESS_IP_OR_HOST>` (controller address or localhost port on kind).

Test:

```bash
curl -H "Host: demo.local" http://<INGRESS_IP_OR_HOST>
```

You should see nginx HTML.

## Verification

```bash
kubectl get ingress deploy-demo-ingress
```

- Shows an **ADDRESS** (or port mapping documented by your controller).

```bash
kubectl describe ingress deploy-demo-ingress
```

- Rules list host `demo.local`, path `/`, backend `deploy-demo-clusterip:80`.

```bash
curl -sS -H "Host: demo.local" -o /dev/null -w '%{http_code}\n' http://<INGRESS_IP_OR_HOST>
```

- HTTP **200**.

## Break & repair

**Symptom:** Ingress exists but `curl` returns 404, 503, or connection refused; `ADDRESS` field empty.

**Cause:** No Ingress Controller, wrong Service name/port in backend, or Service has no endpoints.

**Repair:**

```bash
kubectl get pods -A | grep -i ingress
kubectl describe ingress deploy-demo-ingress
kubectl get svc deploy-demo-clusterip
kubectl get endpoints deploy-demo-clusterip
```

- Install or fix Ingress Controller if no controller pods.
- Fix `backend.service.name` / `port.number` to match [Lab 8](08-clusterip-service.md).
- If endpoints empty, fix Deployment/Pods first, then re-test.

## You should now be able to…

- [ ] Explain Ingress vs NodePort (HTTP routing vs raw node port).
- [ ] State that an Ingress Controller must be running for rules to work.
- [ ] Create an Ingress rule mapping host + path to a Service.
- [ ] Test with `curl -H "Host: ..."` when DNS is not set up.

---

Previous: [StorageClass + PVC + PV](10-storage-pvc-pv.md) · Next: [Cleanup](12-cleanup.md)
