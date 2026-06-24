# Learning Path — Full Course

Use this as your roadmap and progress tracker across Docker, Kubernetes, Terraform, and AWS.

**How to use this file:** Work through phases in order. Each phase links to sections that explain *why* before *how*. If a term is new, read the linked page — do not jump straight to config + `apply`.

**No prior experience assumed.** Phase 0 = install only. Phase 1 = Docker from scratch. Later phases build step by step.

## Phase 0 — Tooling setup only (no concepts yet)

Goal of this phase: prepare your machine so all later labs run smoothly.
You are **not** expected to learn Docker or Kubernetes theory here.

Why Docker appears in Phase 0:
- Local Kubernetes options like `kind` and Docker Desktop Kubernetes rely on container runtime tooling.
- So Docker is installed first as an execution prerequisite for local Kubernetes labs.

**Install guides:**

- [00-tooling-setup.md](00-tooling-setup.md) — index + checklist
- [macOS](00-tooling-setup-macos.md)
- [Linux — Ubuntu LTS](00-tooling-setup-linux.md)

- [ ] Install Docker (Engine on Linux, Desktop or Colima on macOS)
- [ ] Install `kubectl`
- [ ] Local cluster + namespace `kube-lab` (OS guide Steps 4–5)
- [ ] Install `terraform` CLI
- [ ] Install AWS CLI + configure a lab profile

Optional reading (concepts, not setup): [where to run your cluster](../02-kubernetes/01-foundations/01-where-to-run-cluster.md).

---

## Phase 1 — Docker

Chapter: [01-docker](../01-docker/)

- [ ] Docker basics (images, containers, cleanup)
- [ ] Dockerfile basics
- [ ] Build, run, logs, exec
- [ ] Volumes and port mapping
- [ ] Docker Compose (multi-container apps)
- [ ] **Capstone** — mini production stack (proxy + API + Redis)

---

## Phase 2 — Kubernetes foundations

Chapter: [02-kubernetes/01-foundations](../02-kubernetes/01-foundations/)

- [ ] Cluster architecture (control plane vs worker nodes)
- [ ] `kubectl` basics (`get`, `describe`, `apply`, `logs`, `exec`)
- [ ] YAML structure (`apiVersion`, `kind`, `metadata`, `spec`)

---

## Phase 3 — Kubernetes core workloads and exposure

Concepts: [02-kubernetes/02-core-concepts](../02-kubernetes/02-core-concepts/)  
Labs: [02-kubernetes/04-hands-on-labs](../02-kubernetes/04-hands-on-labs/)

- [ ] Pod
- [ ] ReplicaSet
- [ ] Deployment
- [ ] StatefulSet
- [ ] ClusterIP Service
- [ ] NodePort Service
- [ ] Ingress

---

## Phase 4 — Kubernetes configuration, identity, data

- [ ] ConfigMap
- [ ] Secret
- [ ] ServiceAccount
- [ ] StorageClass
- [ ] PersistentVolumeClaim (PVC)
- [ ] PersistentVolume (PV)

---

## Phase 5 — Helm (Kubernetes packaging)

Chapter: [02-kubernetes/05-helm](../02-kubernetes/05-helm/)

**When:** After Phase 3–4 labs.

- [ ] Vocabulary: chart, values, templates, repository, release, package
- [ ] **CLI** — install `helm`, command map
- [ ] **Chart** structure — `helm create`, `helm template`
- [ ] **Repository** — `helm repo add`, search, show values
- [ ] **install** + **release** — `helm install`, list, status
- [ ] **upgrade** + **rollback** — revisions, `helm history`
- [ ] **uninstall**
- [ ] **package** — `helm package`, custom chart + `.tgz`

---

## Phase 6 — AWS

Chapter: [04-aws](../04-aws/)

- [ ] IAM users, roles, policies
- [ ] VPC, subnets, security groups, routing
- [ ] EC2, EBS, ALB/NLB
- [ ] EKS control plane + node groups
- [ ] ECR for container images

---

## Phase 7 — Terraform

Chapter: [03-terraform](../03-terraform/)

- [ ] HCL syntax, state, providers (no AWS account required)
- [ ] Variables, outputs, modules
- [ ] Remote state (S3 + DynamoDB lock)
- [ ] Provision AWS VPC + EKS (after Phase 5 — AWS basics)

---

## Validation rule

Mark a concept as learned only when you can:

- explain it in one sentence,
- create it with `kubectl apply -f` / `terraform apply` / AWS CLI,
- verify it with at least two checks.
