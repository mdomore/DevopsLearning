# Ingress

## Explanation

**Ingress** defines **HTTP/HTTPS routing rules** — by hostname, path, or TLS — and sends traffic to **Services** behind the scenes.

**Problem it solves:** NodePort exposes raw ports on nodes; LoadBalancers cost money and one Service per app does not scale. Ingress gives you “`https://api.example.com/v1` → this Service, `https://app.example.com` → that Service” on a single entry point.

Ingress rules alone do nothing until an **Ingress Controller** (for example NGINX Ingress, Traefik) runs in the cluster and implements them.

## Example — add to the cluster

**What's new:** HTTP routing by hostname to Service `concept-web` (L7 entry point).

**Before this step:** [ClusterIP](clusterip-service.md) or [NodePort](nodeport-service.md) — Service `concept-web` must exist. An **Ingress Controller** must be running:

```bash
kubectl get pods -A | grep -i ingress
```

If nothing shows, install a controller for your cluster before continuing (see [Lab 11](../04-hands-on-labs/11-ingress.md)).

Save as `~/kube-lab/manifests/concepts/13-ingress.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: concept-ing
  namespace: kube-lab
spec:
  rules:
  - host: concept.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: concept-web
            port:
              number: 80
```

Apply:

```bash
kubectl apply -f ~/kube-lab/manifests/concepts/13-ingress.yaml
```

### Verify

```bash
kubectl get ingress concept-ing
kubectl describe ingress concept-ing
```

Test (replace address from `kubectl get ingress` — Docker Desktop often uses localhost):

```bash
curl -H "Host: concept.local" http://127.0.0.1:<INGRESS_PORT>
```

Optional `/etc/hosts`: `127.0.0.1 concept.local`

### Break & repair

Wrong `backend.service.name` — 502 from controller; fix Service name and re-apply.

Stack complete: [progressive-stack.md](progressive-stack.md)

## How it relates to other objects

- **Service (ClusterIP)** — Ingress backends point to Services, not Pod IPs directly.
- **Deployment / Pod** — Services still select Pods; Ingress sits in front of Services.
- **Secret** — often stores TLS certificates referenced by Ingress for HTTPS.
- **NodePort / LoadBalancer** — alternative external access patterns; Ingress is L7 (HTTP) routing.

See also: [Relationship map](relationship-map.md)

## Hands-on lab

Lab: [../04-hands-on-labs/11-ingress.md](../04-hands-on-labs/11-ingress.md)

## You should now be able to…

- Explain what problem Ingress solves compared to NodePort.
- State that an Ingress Controller must be installed for Ingress resources to work.
- Describe the traffic path: client → Ingress → Service → Pod.
