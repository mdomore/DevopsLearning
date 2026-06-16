# NodePort Service

## Explanation

A **NodePort** Service exposes your app on a **fixed port on every node** (typically `30000–32767`), in addition to the internal ClusterIP.

**Problem it solves:** For labs and simple setups, you need to reach a Service from your laptop without cloud LoadBalancers or a full Ingress setup. NodePort opens the same port on each node’s IP so `http://<node-ip>:<nodePort>` reaches the Service.

Trade-off: quick external access, but not ideal for production (wide exposure, non-standard ports, no L7 routing by host/path).

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
