# VPC networking

## Prerequisites

- [IAM basics](02-iam-basics.md) — you can use the CLI with profile `lab`.
- [Kubernetes — cluster architecture](../02-kubernetes/01-foundations/02-cluster-architecture.md) — recap: nodes run pods; traffic enters via Services/Ingress.

---

## Explanation

A **VPC** (Virtual Private Cloud) is your private network inside AWS — like a fenced campus with its own address plan.

Key terms:

| Term | Plain meaning |
|---|---|
| **CIDR** | IP address range notation, e.g. `10.0.0.0/16` |
| **Subnet** | A slice of the VPC in one **Availability Zone (AZ)** — an isolated data center within a region |
| **Route table** | Rules: "traffic to destination X goes via gateway Y" |
| **Internet Gateway (IGW)** | Lets **public** subnets reach the public internet |
| **NAT Gateway** | Lets **private** subnets initiate outbound internet without inbound exposure |
| **Security group (SG)** | Stateful firewall on a network interface (allow/deny ports) |
| **NACL** (Network ACL) | Stateless subnet-level firewall (less common for app rules) |

Typical layout for Kubernetes on AWS:

- **Public subnets** — load balancers, NAT gateways
- **Private subnets** — EKS worker nodes (pods get IPs via the **VPC CNI** add-on)
- **Control plane** — managed by AWS (you do not SSH to it)

Traffic path you will use later:

```text
Internet → ALB (public subnet) → Node/Ingress (private or public) → Pod
```

**Security group vs NACL:** Use **security groups** for almost all app and node rules (return traffic allowed automatically). NACLs are coarse subnet gates — know they exist, not your first debugging stop.

---

## What you will do

1. Inspect default or existing VPC objects in the console (no cost).
2. Map subnets, route tables, and an IGW conceptually.
3. Describe how this layout supports an EKS cluster later.
4. (Optional) Draw the traffic flow from internet to pod.

Hands-on **building** a full VPC is in [Terraform — AWS VPC lab](../03-terraform/06-aws-vpc-lab.md).

---

## Implementation

### Step 1 — List VPCs (CLI)

```bash
aws ec2 describe-vpcs --profile lab --region eu-west-1 \
  --query 'Vpcs[*].[VpcId,CidrBlock,IsDefault]' --output table
```

Note one `VpcId` (e.g. `vpc-0abc...`). Many accounts have a **default VPC** per region.

### Step 2 — Subnets and AZs

```bash
aws ec2 describe-subnets --profile lab --region eu-west-1 \
  --filters "Name=vpc-id,Values=<YOUR_VPC_ID>" \
  --query 'Subnets[*].[SubnetId,AvailabilityZone,CidrBlock,MapPublicIpOnLaunch]' --output table
```

`MapPublicIpOnLaunch=true` often indicates a **public** subnet (simplified check).

### Step 3 — Route tables and internet gateway

Console path: **VPC** → **Your VPCs** → select VPC → **Resource map** (or **Route tables** / **Internet gateways**).

Look for:

- A route `0.0.0.0/0` → `igw-...` on a public route table
- Private subnets using `0.0.0.0/0` → `nat-...` (when NAT is deployed)

### Step 4 — Security group example

```bash
aws ec2 describe-security-groups --profile lab --region eu-west-1 \
  --filters "Name=vpc-id,Values=<YOUR_VPC_ID>" \
  --query 'SecurityGroups[*].[GroupId,GroupName]' --output table
```

Open one group in the console → **Inbound rules**. Example for a web server: allow TCP `443` from `0.0.0.0/0`.

For EKS nodes, rules are often minimal on the node SG; the load balancer SG allows user traffic.

### Step 5 — Map to Kubernetes (preview)

| AWS piece | Kubernetes use |
|---|---|
| Public subnet | External LoadBalancer / Ingress controller |
| Private subnet | Worker nodes |
| Security groups | Node ↔ control plane ↔ LB traffic |
| Pod IPs in VPC CIDR | VPC CNI assigns real VPC IPs to pods |

Full build: coming in [Terraform VPC lab](../03-terraform/06-aws-vpc-lab.md).

---

## Verification checklist

- [ ] You can name your VPC ID and one subnet ID.
- [ ] You can explain public vs private subnet in plain language.
- [ ] You can point to an IGW or NAT route in a route table (console or CLI).
- [ ] You can describe traffic: Internet → ALB → Service/Ingress → Pod (even as a sketch).

---

## Break & repair

| Symptom | Likely cause | Fix |
|---|---|---|
| Subnet has no internet | Missing IGW or wrong route table | Associate subnet with route table that has `0.0.0.0/0` → IGW |
| Private nodes cannot pull images | No NAT in private subnet route | Add NAT gateway + route (costly — tear down when idle; see [Cost control](07-cost-control.md)) |
| Connection timeout to pod | SG blocks port | Add inbound rule on node or LB security group |
| Wrong AZ for LB | Subnet in single AZ only | Use subnets in at least two AZs for production patterns |

**Break on purpose:** In the console, view a security group with no inbound rules and explain why SSH or HTTP would fail.

---

## You should now be able to…

- Define VPC, subnet, route table, IGW, NAT, and security group.
- Contrast security groups with NACLs.
- Relate VPC layout to where EKS nodes and load balancers live.

Previous: [IAM basics](02-iam-basics.md) · Next: [EC2 and load balancers](04-ec2-and-load-balancers.md)
