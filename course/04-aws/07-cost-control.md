# Cost control for labs

## Prerequisites

- [Kubernetes — where to run cluster](../02-kubernetes/01-foundations/01-where-to-run-cluster.md) — local vs cloud cost tradeoffs.
- [EKS overview](06-eks-overview.md) — what costs money even when you are not deploying apps.
- [AWS account and CLI setup](01-account-cli-setup.md) — you can use the console and CLI.

---

## Explanation

AWS charges for **what is running**, not just what you use actively. Lab resources that commonly surprise learners:

| Resource | Why it costs |
|---|---|
| **EKS control plane** | Hourly fee per cluster |
| **EC2 worker nodes** | Hourly per instance |
| **NAT Gateway** | Hourly + data processed (expensive if left on) |
| **Load Balancer** | Hourly while it exists |
| **EBS volumes** | Storage attached to stopped instances still bills |

**Budget** — a planned spending limit you set in AWS Budgets.

**Billing alert** — email (or SNS notification) when spend crosses a threshold.

**Spot instances** — spare AWS capacity at lower price; can be interrupted — OK for non-prod labs with tolerance for restarts.

This course prefers **short, intentional test windows**: create → verify → destroy.

---

## What you will do

1. Create a monthly cost budget with an email alert.
2. Learn a teardown checklist before leaving a lab session.
3. Apply right-sizing habits (smallest nodes, one node when possible).
4. Find current spend in the billing console.

---

## Implementation

### Step 1 — Create a budget (console)

1. AWS Console → search **Billing** → **Budgets**.
2. **Create budget** → **Cost budget** → **Monthly**.
3. Amount: e.g. **$20** (adjust to your comfort).
4. Alert threshold: e.g. **80%** and **100%** → your email.
5. Create.

You should receive a confirmation email from AWS.

### Step 2 — Teardown checklist (run after each lab)

Before closing your laptop after AWS labs:

- [ ] Delete Kubernetes `LoadBalancer` Services (or Ingress controllers that create LBs).
- [ ] `terraform destroy` for lab stacks ([Terraform VPC](../03-terraform/06-aws-vpc-lab.md) / [EKS](../03-terraform/07-eks-cluster-lab.md)) when finished.
- [ ] Confirm **NAT gateways** removed (VPC lab).
- [ ] Confirm **EKS cluster** deleted (control plane fee stops after delete completes).
- [ ] Terminate stray **EC2** instances (EC2 → Instances → filter running).
- [ ] Delete unused **EBS volumes** (Volumes → available/unattached).

Quick check — load balancers:

```bash
aws elbv2 describe-load-balancers --profile lab --region eu-west-1 \
  --query 'LoadBalancers[*].LoadBalancerName' --output table
```

Empty table is good after cleanup.

### Step 3 — Right-sizing habits

- Use **one** small node (`t3.small` or similar) for learning phases.
- Prefer **local kind/minikube** until you need real cloud networking ([Kubernetes foundations](../02-kubernetes/01-foundations/01-where-to-run-cluster.md)).
- For EKS experiments, schedule **destroy the same day**.
- Consider **Spot** node groups only after you understand interruptions (optional advanced).

### Step 4 — Review current costs

Billing → **Bills** or **Cost Explorer** → filter by service (EKS, EC2, VPC/NAT, ELB).

No CLI required; console is enough for labs.

---

## Verification checklist

- [ ] A budget exists with at least one email alert.
- [ ] You can recite three resources that must be deleted to avoid overnight charges.
- [ ] You know where to look in the console for month-to-date spend.
- [ ] You have a personal rule: no idle EKS + NAT over weekends.

---

## Break & repair

| Symptom | Likely cause | Fix |
|---|---|---|
| Bill higher than expected | Forgotten LB or NAT | Run teardown checklist; Cost Explorer by service |
| Budget alert but no detail | Budget is account-wide | Use Cost Explorer filters; tag resources in Terraform later |
| Cluster deleted but still charged | Delete still in progress | Wait; check CloudFormation/Terraform destroy logs |
| Many `available` EBS volumes | Instances terminated without volume delete | Delete volumes manually |

**Break on purpose:** Create a budget at $1 on a sandbox account — understand you may get alerts quickly if any NAT/EKS runs.

---

## You should now be able to…

- Set up a billing budget and alert on day one.
- Tear down EKS, NAT gateways, and load balancers systematically.
- Choose local clusters until cloud features are truly needed.

Previous: [EKS overview](06-eks-overview.md) · Next: [Terraform](../03-terraform/)

Content aligned with [Kubernetes foundations — cost control](../02-kubernetes/01-foundations/01-where-to-run-cluster.md).
