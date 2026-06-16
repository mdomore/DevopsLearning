# HCL, providers, resources

## Prerequisites

- [Getting started — tooling setup](../00-getting-started/00-tooling-setup.md) — `terraform` CLI installed (`terraform version` works).
- A working directory on your machine; basic terminal use from the same chapter.

**AWS not required** for this section — we use the built-in **local** provider to create a file on disk.

---

## Explanation

**Terraform** is a tool for **Infrastructure as Code (IaC)**: you describe servers, networks, and settings in text files, then Terraform creates or updates real infrastructure to match.

**HCL** (HashiCorp Configuration Language) is the syntax in `.tf` files — human-readable blocks with `key = value` pairs.

Core ideas:

| Term | Plain meaning |
|---|---|
| **Provider** | Plugin that talks to a platform (AWS, local filesystem, etc.) |
| **Resource** | Something Terraform creates and manages (e.g. a file, S3 bucket, VPC) |
| **Data source** | Read-only lookup of existing objects (no create/destroy) |
| **Block** | A section like `provider "aws" { ... }` or `resource "..." "name" { ... }` |

**Resource vs data source:**

- **Resource** — "Make this exist" (`terraform apply` creates it).
- **Data source** — "Tell me about something that already exists" (reference only).

Workflow preview (each step has its own section later):

```text
terraform init → terraform plan → terraform apply → (terraform destroy)
```

- **init** — download providers
- **plan** — show proposed changes
- **apply** — make changes
- **destroy** — remove managed resources

The **AWS provider** appears in later labs; here we use **`hashicorp/local`** so you learn HCL without a cloud bill.

---

## What you will do

1. Create a small Terraform project directory.
2. Write a provider block and one resource.
3. Run `terraform init` and `terraform plan`.
4. (Optional) Run `terraform apply` to create a local file.

---

## Implementation

### Step 1 — Project directory

```bash
mkdir -p ~/learning/terraform-01-intro && cd ~/learning/terraform-01-intro
```

### Step 2 — `main.tf`

Create `main.tf`:

```hcl
terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }
  }
}

provider "local" {}

resource "local_file" "hello" {
  content  = "Hello from Terraform\n"
  filename = "${path.module}/hello-from-terraform.txt"
}
```

**Key lines:**

- `terraform { required_providers ... }` — pins which provider plugins to use.
- `provider "local"` — configures the local provider (empty block = defaults).
- `resource "local_file" "hello"` — type `local_file`, logical name `hello`.

### Step 3 — Initialize

Downloads the local provider plugin:

```bash
terraform init
```

Expected: `Terraform has been successfully initialized!`

### Step 4 — Plan

Shows what would change without applying:

```bash
terraform plan
```

Expected: plan to **create** 1 resource (`local_file.hello`).

### Step 5 — Apply (optional)

```bash
terraform apply
```

Type `yes` when prompted. Confirm file exists:

```bash
cat hello-from-terraform.txt
```

### Preview — AWS provider (later labs)

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

# resource "aws_s3_bucket" "example" { ... }  # coming in AWS labs
```

Full AWS resources: [AWS VPC lab](06-aws-vpc-lab.md) after the [AWS chapter](../04-aws/).

---

## Verification checklist

- [ ] `terraform init` completes without errors.
- [ ] `terraform plan` shows 1 resource to add.
- [ ] You can explain **provider**, **resource**, and **data source** in plain language.
- [ ] (If applied) `hello-from-terraform.txt` exists with expected content.

---

## Break & repair

| Symptom | Likely cause | Fix |
|---|---|---|
| `terraform: command not found` | CLI not installed | [Tooling setup](../00-getting-started/00-tooling-setup.md) |
| Provider install fails | Network/registry block | Retry; check proxy/firewall |
| Plan wants to destroy unexpectedly | Changed resource name | Names are addresses; renaming replaces resource — read plan carefully |
| Syntax error | Invalid HCL | Check braces and `=`; run `terraform validate` |

**Break on purpose:** Change `content` in `main.tf`, run `plan` again — see an **update** in the diff.

---

## You should now be able to…

- Read a minimal `.tf` file and identify provider and resource blocks.
- Run `terraform init` and `terraform plan` successfully.
- State the difference between a resource and a data source.

Previous: [AWS](../04-aws/) (for later labs) · Next: [State and plan/apply](02-state-plan-apply.md)
