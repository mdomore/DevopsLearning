# Modules

## Prerequisites

- [Variables and outputs](03-variables-outputs.md) — inputs and exported values.
- Comfortable with one Terraform project folder from earlier sections.

---

## Explanation

A **module** is a reusable bundle of Terraform configuration — like a function you call with arguments.

Why modules:

- **Reuse** — same VPC pattern for dev and staging with different CIDR inputs.
- **Clarity** — root `main.tf` stays short; details live in `./modules/...`.
- **Sharing** — public registry modules (e.g. `terraform-aws-modules/vpc/aws`) or your own git repo.

Standard layout inside a module:

```text
modules/my-module/
  main.tf       # resources
  variables.tf  # inputs
  outputs.tf    # exports
```

**Root module** — the directory where you run `terraform apply` (calls child modules).

**Child module** — invoked with a `module "name" { source = "..." }` block.

---

## What you will do

1. Extract the local file logic into `modules/file-writer`.
2. Call the module from the root with inputs.
3. Plan and apply; read module output at root level.

AWS VPC module extraction preview: coming in [AWS VPC lab](06-aws-vpc-lab.md).

---

## Implementation

### Step 1 — Directory layout

```bash
mkdir -p ~/learning/terraform-04-modules/modules/file-writer
cd ~/learning/terraform-04-modules
```

### Step 2 — Child module `modules/file-writer/variables.tf`

```hcl
variable "greeting" {
  type = string
}

variable "name_prefix" {
  type = string
}
```

### Step 3 — Child module `modules/file-writer/main.tf`

```hcl
locals {
  filename = "${var.name_prefix}-from-module.txt"
}

resource "local_file" "hello" {
  content  = var.greeting
  filename = "${path.module}/${local.filename}"
}
```

Note: `path.module` here is the **module** directory, not root.

### Step 4 — Child module `modules/file-writer/outputs.tf`

```hcl
output "file_path" {
  value = local_file.hello.filename
}
```

### Step 5 — Root `main.tf`

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

module "writer" {
  source = "./modules/file-writer"

  greeting    = "Hello from a module\n"
  name_prefix = "course"
}
```

### Step 6 — Root `outputs.tf`

```hcl
output "generated_file" {
  value = module.writer.file_path
}
```

### Step 7 — Apply

```bash
terraform init
terraform plan
terraform apply
```

Expected output includes `generated_file` path under `modules/file-writer/`.

### Preview — VPC module call (later)

```hcl
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"

  name = "course-vpc"
  cidr = "10.0.0.0/16"
  azs  = ["eu-west-1a", "eu-west-1b"]
  # public_subnets, private_subnets — full lab in section 06
}
```

Full configuration: **coming in this chapter** — [06-aws-vpc-lab.md](06-aws-vpc-lab.md).

---

## Verification checklist

- [ ] `terraform init` downloads providers and initializes the local module.
- [ ] Plan shows resources under `module.writer`.
- [ ] Root output `generated_file` matches module output.
- [ ] You can explain root vs child module in one sentence.

---

## Break & repair

| Symptom | Likely cause | Fix |
|---|---|---|
| `Module not found` | Wrong `source` path | Use `./modules/file-writer` relative to root |
| Duplicate resource names | Two modules same type/name | Each module instance is separate; names differ by module path |
| Output undefined | Wrong output name | Check `outputs.tf` in module; reference `module.NAME.OUTPUT` |
| Init after module change | Source updated | Re-run `terraform init -upgrade` when switching registry versions |

**Break on purpose:** Change `name_prefix` input at root only — see plan update file in module without editing module code.

---

## You should now be able to…

- Structure `main.tf`, `variables.tf`, `outputs.tf` inside a module.
- Call a module with inputs and expose its outputs at root.
- Describe why VPC logic belongs in a module for reuse.

Previous: [Variables and outputs](03-variables-outputs.md) · Next: [Remote state](05-remote-state.md)
