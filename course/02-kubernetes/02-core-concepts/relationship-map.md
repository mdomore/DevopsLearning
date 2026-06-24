# Relationship map

## Explanation

Kubernetes objects work together. This page is a **mental map** — not something you apply with `kubectl`, but a guide for reading YAML and debugging (“which object owns which?”).

**Problem it solves:** Beginners see many resource kinds (Pod, Deployment, Service, Ingress, PVC…) and lose track of dependencies. The map shows the usual direction of control and traffic.

## Progressive stack (this course)

Follow [progressive-stack.md](progressive-stack.md) — each concept adds one object to `kube-lab`:

```
kube-lab (namespace)
  ├─ concept-pod → concept-rs → concept-web (Deployment)
  │     + concept-config, concept-secret, concept-sa
  │     + Service concept-web (ClusterIP → NodePort)
  │     + Ingress concept-ing
  ├─ concept-pvc → concept-storage (Pod) → PV (auto)
  └─ concept-sts + concept-sts-headless + PVCs per replica
```

## How objects connect

**Workloads (what runs)**

```
Deployment  →  ReplicaSet  →  Pod
StatefulSet  →  Pod  (stable names + optional PVCs per replica)
```

**Configuration & identity (what Pods use)**

```
Pod  →  ConfigMap / Secret  (env or files)
Pod  →  ServiceAccount  (cluster identity)
Pod  →  PVC  →  PV  (often via StorageClass)
```

**Networking (how traffic reaches Pods)**

```
Client  →  Ingress  →  Service  →  Pod  (selected by labels)
Client  →  NodePort / LoadBalancer  →  Service  →  Pod
In-cluster  →  ClusterIP Service  →  Pod
```

**Storage (how data persists)**

```
StorageClass  →  PV  ←  PVC  ←  Pod volumeMount
```

## Hands-on lab

There is no single lab for this page — each concept has its own lab under [../04-hands-on-labs/](../04-hands-on-labs/). Start with [Pod lab](../04-hands-on-labs/01-pod.md) and follow the numbered sequence.

## You should now be able to…

- Draw or recite the chain Deployment → ReplicaSet → Pod.
- Explain how a Service finds Pods (labels) and how Ingress relates to Services.
- Trace storage from a Pod mount through PVC to PV and StorageClass.
