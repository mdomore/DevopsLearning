# AWS VPC lab

## Prerequisites

Complete the [AWS chapter](../04-aws/) first — at minimum:

- [Account and CLI setup](../04-aws/01-account-cli-setup.md)
- [IAM basics](../04-aws/02-iam-basics.md)
- [VPC networking](../04-aws/03-vpc-networking.md) — concepts recap: VPC, subnets, IGW, NAT, route tables, security groups

Terraform sections [01–05](01-hcl-providers-resources.md) — HCL, state, variables, modules, remote state.

---

## Explanation

This lab provisions a **VPC** (Virtual Private Cloud) suitable for **EKS** later: public subnets for load balancers, private subnets for worker nodes, internet egress via **NAT Gateway**.

You will use:

- **AWS provider** — Terraform plugin for AWS
- **Variables** — region, CIDR, environment name
- **Module** — community `terraform-aws-modules/vpc/aws` (battle-tested) or equivalent custom module

**Cost warning:** NAT Gateways bill hourly. Run `terraform destroy` when done ([AWS cost control](../04-aws/07-cost-control.md)).

---

## What you will do

1. Create a Terraform root module for VPC.
2. `terraform init` → `plan` → `apply` a VPC with **2 Availability Zones**.
3. Verify subnets, route tables, and internet gateway in AWS.
4. `terraform destroy` when the lab session ends.

Full copy-paste stack: **coming in this chapter** — repository lab folder will contain `main.tf`, `variables.tf`, and `outputs.tf`. Below is the preview you will apply.

---

## Implementation

### Step 1 — Project directory

```bash
mkdir -p ~/learning/terraform-vpc-lab && cd ~/learning/terraform-vpc-lab
```

### Step 2 — Core configuration (preview)

`providers.tf`:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region  = var.aws_region
  profile = "lab"
}
```

`variables.tf`:

```hcl
variable "aws_region" {
  type    = string
  default = "eu-west-1"
}

variable "name_prefix" {
  type    = string
  default = "course"
}

variable "vpc_cidr" {
  type    = string
  default = "10.0.0.0/16"
}
```

`main.tf`:

```hcl
data "aws_availability_zones" "available" {
  state = "available"
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "${var.name_prefix}-vpc"
  cidr = var.vpc_cidr

  azs             = slice(data.aws_availability_zones.available.names, 0, 2)
  public_subnets  = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnets = ["10.0.11.0/24", "10.0.12.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = true # cheaper for labs; not HA

  tags = {
    Environment = "lab"
    ManagedBy   = "terraform"
  }
}
```

`outputs.tf`:

```hcl
output "vpc_id" {
  value = module.vpc.vpc_id
}

output "private_subnets" {
  value = module.vpc.private_subnets
}

output "public_subnets" {
  value = module.vpc.public_subnets
}
```

### Step 3 — Initialize and plan

```bash
terraform init
terraform plan
```

Review: ~1 VPC, subnets, IGW, NAT, route tables, security groups (module defaults).

### Step 4 — Apply

```bash
terraform apply
```

Type `yes`. Note outputs `vpc_id` and subnet IDs.

### Step 5 — Verify (CLI)

```bash
export AWS_PROFILE=lab
terraform output vpc_id

aws ec2 describe-subnets --region eu-west-1 \
  --filters "Name=vpc-id,Values=$(terraform output -raw vpc_id)" \
  --query 'Subnets[*].[SubnetId,AvailabilityZone,CidrBlock,Tags[?Key==`Name`].Value|[0]]' \
  --output table
```

Console: **VPC** → **Your VPCs** → select VPC → **Resource map**.

Confirm:

- Public subnets → route to **Internet Gateway**
- Private subnets → route to **NAT Gateway**

### Step 6 — Security groups (awareness)

The VPC module creates default SG; EKS lab adds node and cluster SG rules. For this lab, note default SG exists:

```bash
aws ec2 describe-security-groups --region eu-west-1 \
  --filters "Name=vpc-id,Values=$(terraform output -raw vpc_id)" \
  --query 'SecurityGroups[*].[GroupId,GroupName]' --output table
```

### Step 7 — Destroy

```bash
terraform destroy
```

Wait until NAT and VPC are gone — check Cost Explorer if unsure.

---

## Verification checklist

- [ ] `terraform apply` completed without error.
- [ ] Two public and two private subnets in distinct AZs.
- [ ] Internet gateway attached; NAT gateway present (if enabled).
- [ ] `terraform output` shows VPC and subnet IDs.
- [ ] `terraform destroy` removes billable NAT/VPC resources.

---

## Break & repair

| Symptom | Likely cause | Fix |
|---|---|---|
| `AccessDenied` on create | IAM too narrow | Attach VPC/NAT/EC2 create permissions on lab user |
| NAT still billing | Destroy incomplete | Re-run destroy; delete ENI/NAT manually in console |
| Wrong subnet count | AZ slice | Check `azs` length matches subnet CIDR lists |
| Module download fails | Network/registry | Retry `terraform init -upgrade` |

**Break on purpose:** Change `single_nat_gateway` to `false`, plan — see second NAT (cost jump) — revert for labs.

---

## You should now be able to…

- Provision a multi-AZ VPC with Terraform modules.
- Verify subnets and routing in console or CLI.
- Tear down VPC lab resources to control cost.

Previous: [Remote state](05-remote-state.md) · Next: [EKS cluster lab](07-eks-cluster-lab.md)
