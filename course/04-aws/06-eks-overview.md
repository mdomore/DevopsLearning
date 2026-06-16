# EKS overview

## Prerequisites

- [VPC networking](03-vpc-networking.md) — subnets and security groups for nodes and load balancers.
- [EC2 and load balancers](04-ec2-and-load-balancers.md) — what a node and ALB are.
- [Kubernetes — cluster architecture](../02-kubernetes/01-foundations/02-cluster-architecture.md) — control plane vs workers.
- [Kubernetes — where to run cluster](../02-kubernetes/01-foundations/01-where-to-run-cluster.md) — when cloud/EKS is worth the cost.

---

## Explanation

**EKS** (Elastic Kubernetes Service) is AWS-managed **Kubernetes**: AWS runs the **control plane** (API server, scheduler, etc.); you run **worker** capacity where pods execute.

| Option | Plain meaning |
|---|---|
| **Managed control plane** | AWS patches and hosts API server; you pay a hourly cluster fee |
| **Managed node group** | EC2 instances join the cluster automatically as nodes |
| **Fargate** | Run pods without managing EC2 (serverless workers; different pricing) |
| **Add-ons** | Extra cluster software: **VPC CNI** (pod IPs), **CoreDNS** (DNS), **kube-proxy** (Service networking) |

**Self-managed** Kubernetes on EC2 means you install and upgrade the control plane yourself — more control, more work. This course focuses on **EKS**.

Connect `kubectl` to EKS:

```bash
aws eks update-kubeconfig --name CLUSTER_NAME --region eu-west-1 --profile lab
```

That merges cluster credentials into `~/.kube/config` (same file you used for local kind/minikube — **context** switches which cluster `kubectl` talks to).

**Responsibilities split:**

| You manage | AWS manages |
|---|---|
| Node groups / Fargate profiles | Control plane availability |
| Workloads (Deployments, etc.) | Control plane upgrades (with your window) |
| VPC, subnets, many add-on versions | etcd backing store for control plane |

Provisioning EKS with Terraform: [Terraform — EKS cluster lab](../03-terraform/07-eks-cluster-lab.md) (after this chapter).

---

## What you will do

1. Understand managed vs self-managed and node group vs Fargate.
2. List EKS clusters (if any) with the CLI.
3. Configure `kubectl` for an EKS cluster (when you have one).
4. Run basic `kubectl` checks against the cluster.

If you do not have a cluster yet, complete Steps 1–2 now; Steps 3–4 are **coming in this chapter** when you run the [Terraform EKS lab](../03-terraform/07-eks-cluster-lab.md).

---

## Implementation

### Step 1 — List clusters

```bash
aws eks list-clusters --profile lab --region eu-west-1
```

Empty list is normal before the lab.

### Step 2 — Inspect cluster details (when it exists)

```bash
aws eks describe-cluster --name CLUSTER_NAME --profile lab --region eu-west-1 \
  --query 'cluster.{Status:status,Endpoint:endpoint,Version:version}'
```

### Step 3 — Configure kubectl (when cluster exists)

```bash
aws eks update-kubeconfig --name CLUSTER_NAME --region eu-west-1 --profile lab
kubectl config current-context
kubectl get nodes
```

Expected: nodes in `Ready` state.

### Step 4 — Identify add-ons (console)

EKS → **Clusters** → your cluster → **Add-ons** tab. Note **Amazon VPC CNI**, **CoreDNS**, **kube-proxy**.

### Step 5 — Run a sample workload (preview)

After the cluster works, reuse labs from [Kubernetes hands-on](../02-kubernetes/04-hands-on-labs/):

```bash
kubectl apply -f deployment.yaml
kubectl get pods -o wide
```

LoadBalancer Services create AWS load balancers — remember [Cost control](07-cost-control.md).

---

## Verification checklist

- [ ] You can explain control plane vs worker in one sentence each.
- [ ] You know what a managed node group is.
- [ ] `aws eks list-clusters --profile lab` runs without error.
- [ ] (When cluster exists) `kubectl get nodes` shows Ready nodes.
- [ ] (When cluster exists) You can name the three default add-ons: VPC CNI, CoreDNS, kube-proxy.

---

## Break & repair

| Symptom | Likely cause | Fix |
|---|---|---|
| `kubectl` connection refused | Wrong context or cluster down | `update-kubeconfig`; check `aws eks describe-cluster` status |
| Nodes NotReady | Node role / subnet / SG issue | Check node group events in console; VPC lab prerequisites |
| Pods pending | Insufficient capacity | Scale node group; check instance limits |
| Unauthorized | IAM user lacks `eks:DescribeCluster` | Extend lab IAM policy |

**Break on purpose:** Switch to your local kind context with `kubectl config use-context ...`, run `get nodes`, switch back to EKS — notice different node names.

---

## You should now be able to…

- Describe what EKS manages vs what you still manage.
- Run `aws eks update-kubeconfig` and use `kubectl` against EKS.
- Point to the Terraform lab as the next step to create a cluster.

Previous: [ECR](05-ecr.md) · Next: [Cost control for labs](07-cost-control.md)

See also: [Kubernetes — where to run cluster](../02-kubernetes/01-foundations/01-where-to-run-cluster.md)
