# Kubernetes Glossary (Quick Lookup)

Use this page when you see an unfamiliar word. Each term has a longer explanation in [Core concepts](../02-core-concepts/) and hands-on practice in [Labs](../04-hands-on-labs/).

| Term | Meaning |
|---|---|
| Pod | Smallest runnable unit; one or more containers together. |
| ReplicaSet | Ensures a target number of Pod replicas. |
| Deployment | Declarative manager for ReplicaSets and rolling updates. |
| StatefulSet | Workload for stable identity and persistent state. |
| ConfigMap | Non-secret configuration data for Pods. |
| Secret | Sensitive configuration data for Pods. |
| ServiceAccount | Pod identity for Kubernetes API and RBAC permissions. |
| Service | Stable endpoint in front of dynamic Pods. |
| ClusterIP | Internal-only Service type. |
| NodePort | Service type exposed on each node IP + static port. |
| Ingress | HTTP/HTTPS routing layer in front of Services. |
| Ingress Controller | Component that enforces Ingress rules. |
| StorageClass | Template/policy for dynamic volume provisioning. |
| PVC | Claim request for persistent storage. |
| PV | Backing persistent storage resource. |
| Label | Key/value tag for selecting/grouping resources. |
| Selector | Query on labels used by Services/controllers. |
| Namespace | Logical resource isolation boundary. |
| RBAC | Permission model (who can do what). |
| kubelet | Node agent that runs Pod workloads. |
| kube-proxy | Node network component for Service traffic forwarding. |

See also: [../02-core-concepts/](../02-core-concepts/)
