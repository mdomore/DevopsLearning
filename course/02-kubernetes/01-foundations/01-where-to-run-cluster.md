# Where to run your cluster

**Prerequisite:** [Phase 0 — Tooling setup](../../00-getting-started/00-tooling-setup.md) (install Docker, `kubectl`, and a local cluster tool such as `kind` or `minikube` for your OS).

Previous: [Prerequisites](00-prerequisites.md)

---

## Explanation

Kubernetes runs on a **cluster** — one or more machines (physical, virtual, or local) managed by Kubernetes control-plane software. Before you write YAML, you choose **where** that cluster lives.

**Problem this section solves:** There are many ways to run Kubernetes (laptop, cloud, managed service). Picking the wrong option early wastes money or adds friction. This page gives a simple decision path for **learning** this course.

You can run a full cluster on your own computer for most of Phase 1–2. Move to a small **managed cloud** cluster only when you need real external networking, cloud LoadBalancers, or IAM integrations.

---

## Local cluster (recommended to start)

A **local cluster** runs on your machine inside VMs or containers. Cost is usually **zero**; you can delete and recreate it anytime.

Good fits for this course:

| Option | Notes |
|---|---|
| **[kind](https://kind.sigs.k8s.io/)** (Kubernetes in Docker) | Reproducible; matches many course labs; needs Docker |
| **[minikube](https://minikube.sigs.k8s.io/)** | Beginner-friendly single-node (or multi-node) cluster |
| **Docker Desktop Kubernetes** | Easiest GUI toggle; fine for early exploration |

Use local when learning: `kubectl`, Pods, Deployments, Services, ConfigMaps, debugging with `describe` and `logs`.

Install and verify via your OS guide under [Phase 0 — Tooling setup](../../00-getting-started/00-tooling-setup.md):

- macOS → [00-tooling-setup-macos.md](../../00-getting-started/00-tooling-setup-macos.md)
- Linux → [00-tooling-setup-linux.md](../../00-getting-started/00-tooling-setup-linux.md)

Quick sanity check after Phase 0:

```bash
kubectl get nodes
kubectl config current-context
```

---

## Cloud cluster (when you need “real” infrastructure)

A **managed** cluster (cloud provider runs the control plane) is useful when labs need:

- Public **LoadBalancer** IPs
- Integration with cloud **IAM** / identity
- Multi-user access or long-lived shared environments
- Behavior closer to production networking

Examples (not required for early chapters):

| Provider | Typical use in learning |
|---|---|
| **DigitalOcean Kubernetes (DOKS)** | Often simpler/cheaper for personal labs |
| **Hetzner** (where available) | Lower-cost option in supported regions |
| **AWS EKS** | Best when your goal is AWS-native production patterns |

**Short recommendation:** Start **local** for Phase 1 and most of Phase 2. Add a **small managed cluster** only for specific cloud networking or IAM exercises.

---

## Implementation (choose your path)

### Path A — Local (default for this course)

Follow Phase 0 for your OS until `kubectl get nodes` shows Ready.

If you use `kind` with cluster name `kube-lab` (as in Phase 0):

```bash
kind get clusters
kubectl get nodes
```

### Path B — Small managed cluster (optional, later)

When you outgrow local-only labs:

1. Create the smallest single-node (or minimal) cluster your provider allows.
2. Install/configure `kubectl` with the provider’s kubeconfig instructions.
3. Run the same checks as [Prerequisites](00-prerequisites.md) (`kubectl get nodes`, create `kube-lab` namespace).

Do **not** leave idle cloud clusters running — see cost control below.

---

## Verification

Local path success looks like:

- `kubectl get nodes` → at least one node **Ready**
- `kubectl cluster-info` → API server URL responds without connection errors
- Phase 0 checklist in [00-tooling-setup.md](../../00-getting-started/00-tooling-setup.md) is complete

---

## Break & repair

**Break:** Cluster stopped or Docker not running — `kubectl get nodes` hangs or fails.

**Repair:** Start Docker (or your VM backend), then start/recreate the local cluster using the commands in your Phase 0 OS guide. Re-run:

```bash
kubectl get nodes
```

**Break:** Wrong kubectl **context** (pointing at an old or empty cluster):

```bash
kubectl config get-contexts
kubectl config use-context <your-local-context>
kubectl get nodes
```

**Break (cloud):** Forgotten cluster still billing.

**Repair:** Delete the cluster and any LoadBalancers/volumes in the cloud console; set **budget alerts** before the next experiment.

---

## Cost-control checklist (cloud labs)

- Use the smallest worker size and **one node** when possible.
- Delete LoadBalancers and clusters when not testing.
- Set budget alerts on day one.
- Prefer short, intentional test windows over always-on clusters.

---

## You should now be able to…

- Explain why a local cluster is enough for most early course labs.
- Name two local options (`kind`, `minikube`, Docker Desktop) and link to Phase 0 for install steps.
- Describe when a managed cloud cluster is worth the extra cost and complexity.

Next: [Cluster architecture](02-cluster-architecture.md)
