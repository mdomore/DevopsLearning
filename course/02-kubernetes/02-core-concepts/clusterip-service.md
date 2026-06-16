# ClusterIP Service

## Explanation

A **Service** gives Pods a **stable network address** even though Pods come and go. **ClusterIP** is the default Service type: a virtual IP and DNS name **reachable only inside the cluster**.

**Problem it solves:** Pod IPs change when Pods restart. Other Pods (or Ingress) need a steady name like `my-api.kube-lab.svc.cluster.local` that load-balances across healthy backend Pods selected by **labels**.

ClusterIP is what you use for east-west traffic: frontend → backend, app → database sidecar, microservice to microservice.

## How it relates to other objects

- **Pod / Deployment** — Service `selector` matches Pod labels; endpoints update as Pods change.
- **NodePort / LoadBalancer** — build on ClusterIP to expose traffic outside the cluster.
- **Ingress** — routes external HTTP(S) traffic to a Service (often ClusterIP).
- **kube-proxy / CNI** — cluster components implement Service forwarding (conceptual; you observe behavior via DNS and connectivity).

See also: [Relationship map](relationship-map.md)

## Hands-on lab

Lab: [../04-hands-on-labs/08-clusterip-service.md](../04-hands-on-labs/08-clusterip-service.md)

## You should now be able to…

- Explain why Services exist given that Pod IPs are not stable.
- Describe how label selectors connect a Service to Pods.
- Explain that ClusterIP is internal-only and is the default Service type.
