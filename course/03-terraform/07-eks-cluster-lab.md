# EKS cluster lab

## Prerequisites

- [AWS VPC lab](06-aws-vpc-lab.md) — VPC with public/private subnets (or equivalent network).
- [AWS — EKS overview](../04-aws/06-eks-overview.md) — control plane, node groups, `update-kubeconfig`.
- [AWS — cost control](../04-aws/07-cost-control.md) — teardown checklist before you start.
- [Kubernetes hands-on labs](../02-kubernetes/04-hands-on-labs/) — you will redeploy sample workloads on this cluster.

Terraform foundations [01–05](01-hcl-providers-resources.md) and working profile `lab`.

---

## Explanation

This lab uses Terraform to create:

| Piece | Plain meaning |
|---|---|
| **EKS cluster** | AWS-managed Kubernetes API server |
| **Managed node group** | EC2 instances registered as Kubernetes nodes |
| **IAM roles** | Let EKS and nodes call AWS APIs (no long-lived keys on nodes) |
| **Add-ons** | VPC CNI, CoreDNS, kube-proxy (often enabled by default) |

Alternative: **Fargate profiles** run pods without EC2 — different cost model; this lab uses a **small managed node group** for familiarity.

After apply:

```bash
aws eks update-kubeconfig --name CLUSTER_NAME --region eu-west-1 --profile lab
kubectl get nodes
```

Then run [Deployment](../02-kubernetes/04-hands-on-labs/03-deployment.md) and [Service](../02-kubernetes/04-hands-on-labs/08-clusterip-service.md) labs against EKS instead of kind.

**Cost:** EKS control plane + nodes + NAT (from VPC) + any `LoadBalancer` Services. Schedule destroy the same day.

---

## What you will do

1. Configure Terraform for EKS (cluster + node group).
2. Apply and connect `kubectl`.
3. Deploy a sample app from the Kubernetes chapter.
4. Destroy the cluster and confirm nodes gone.

Full module stack (e.g. `terraform-aws-modules/eks/aws`): **coming in this chapter** — preview below.

---

## Implementation

### Step 1 — Project setup

Use the same VPC from section 06 or a dedicated `~/learning/terraform-eks-lab` root module that references VPC outputs.

Minimal preview `main.tf` pattern:

```hcl
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 20.0"

  cluster_name    = "${var.name_prefix}-eks"
  cluster_version = "1.29"

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  eks_managed_node_groups = {
    lab = {
      instance_types = ["t3.small"]
      min_size       = 1
      max_size       = 2
      desired_size   = 1
    }
  }

  tags = {
    Environment = "lab"
  }
}
```

Variables, VPC module, and IAM details will match the repo lab files when published — init from that directory when available.

### Step 2 — Init and apply

```bash
cd ~/learning/terraform-eks-lab   # or course repo path when added
terraform init
terraform plan    # expect cluster, node group, IAM roles — review carefully
terraform apply
```

Apply may take **15–20 minutes**. Do not interrupt.

### Step 3 — Configure kubectl

```bash
aws eks update-kubeconfig \
  --name course-eks \
  --region eu-west-1 \
  --profile lab

kubectl get nodes
```

Expected: one or more nodes `Ready`.

### Step 4 — Sample workload

From [Pod lab](../02-kubernetes/04-hands-on-labs/01-pod.md) or Deployment lab — example:

```bash
kubectl create deployment web --image=nginx:1.27
kubectl expose deployment web --port=80 --type=ClusterIP
kubectl get pods,svc
```

Optional — external access (creates AWS load balancer — costs money):

```bash
kubectl expose deployment web --port=80 --type=LoadBalancer
kubectl get svc web
```

Delete LoadBalancer Service when done:

```bash
kubectl delete svc web
```

### Step 5 — ECR image (optional)

If you completed [ECR](../04-aws/05-ecr.md), patch Deployment to your ECR URI and verify pull on nodes.

### Step 6 — Destroy

```bash
terraform destroy
```

Confirm:

```bash
aws eks list-clusters --profile lab --region eu-west-1
kubectl get nodes   # should fail or show local context only
```

Follow [Cost control teardown](../04-aws/07-cost-control.md).

---

## Verification checklist

- [ ] `terraform apply` created cluster and node group without error.
- [ ] `kubectl get nodes` shows Ready nodes via EKS context.
- [ ] At least one Deployment runs pods on the cluster.
- [ ] (Optional) Sample Service reachable internally with `kubectl run` curl pod.
- [ ] `terraform destroy` removed cluster; no stray load balancers.

---

## Break & repair

| Symptom | Likely cause | Fix |
|---|---|---|
| Nodes NotReady | Subnet/SG/IAM | Check node group events in EKS console; verify private subnet routes to NAT |
| `Unauthorized` on kubectl | Stale kubeconfig | Re-run `update-kubeconfig` |
| Pods pending | Insufficient CPU | Scale node group `desired_size` |
| LoadBalancer pending | Subnet tags / controller | For advanced setup install AWS Load Balancer Controller; or use NodePort for lab |
| Destroy stuck | LB or ENI remaining | Delete Kubernetes Services type LoadBalancer first |

**Break on purpose:** Switch kubectl context to local cluster, run `get nodes`, switch back — confirm you know which cluster you target.

---

## You should now be able to…

- Provision EKS with Terraform and connect `kubectl`.
- Run Kubernetes chapter labs on a real AWS cluster.
- Destroy EKS and related billable resources safely.

Previous: [AWS VPC lab](06-aws-vpc-lab.md) · Next: course complete — revisit [Cost control](../04-aws/07-cost-control.md)

See also: [Kubernetes hands-on labs](../02-kubernetes/04-hands-on-labs/)
