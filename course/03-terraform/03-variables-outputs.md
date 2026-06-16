# Variables and outputs

## Prerequisites

- [State and plan/apply](02-state-plan-apply.md) — you understand plan, apply, and local state.
- [HCL, providers, resources](01-hcl-providers-resources.md) — resource blocks and providers.

---

## Explanation

Hard-coding values (region names, project prefixes) in every resource makes reuse hard. Terraform provides:

| Construct | Plain meaning |
|---|---|
| **`variable`** | Input parameter to the module/root module |
| **`locals`** | Internal computed values (not set from outside) |
| **`output`** | Values exported after apply (for humans or other Terraform) |
| **`.tfvars` file** | Key/value file passed at apply time for environments |

Example mental model:

- **Variable** — "Caller chooses `environment = dev`."
- **Local** — "`name_prefix = dev-app` built from variables."
- **Output** — "After apply, print the file path or bucket ARN."

**`.tfvars`** keeps secrets and environment-specific values out of `main.tf` (still never commit secrets to Git).

---

## What you will do

1. Add variables for greeting text and file name prefix.
2. Use a `locals` block for a full file name.
3. Add an output for the file path.
4. Pass values via `terraform.tfvars` and verify with `plan` / `apply`.

---

## Implementation

### Step 1 — New or reset project

```bash
mkdir -p ~/learning/terraform-03-vars && cd ~/learning/terraform-03-vars
```

### Step 2 — `variables.tf`

```hcl
variable "greeting" {
  type        = string
  description = "Text written into the file"
  default     = "Hello from variables\n"
}

variable "name_prefix" {
  type        = string
  description = "Prefix for the output file name"
}
```

### Step 3 — `main.tf`

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

locals {
  filename = "${var.name_prefix}-output.txt"
}

resource "local_file" "hello" {
  content  = var.greeting
  filename = "${path.module}/${local.filename}"
}
```

### Step 4 — `outputs.tf`

```hcl
output "file_path" {
  description = "Absolute path to the generated file"
  value       = local_file.hello.filename
}
```

### Step 5 — `terraform.tfvars`

```hcl
name_prefix = "course-lab"
greeting    = "Parameterized greeting\n"
```

### Step 6 — Run workflow

```bash
terraform init
terraform plan
terraform apply
```

Note outputs at the end:

```text
file_path = "/Users/you/learning/terraform-03-vars/course-lab-output.txt"
```

Verify:

```bash
cat course-lab-output.txt
```

Override on CLI without editing file:

```bash
terraform plan -var="name_prefix=cli-override"
```

Preview for AWS labs — same pattern for region:

```hcl
variable "aws_region" {
  type    = string
  default = "eu-west-1"
}
```

Used in [AWS VPC lab](06-aws-vpc-lab.md).

---

## Verification checklist

- [ ] `terraform plan` fails or prompts if required `name_prefix` is missing **without** `.tfvars`.
- [ ] With `terraform.tfvars`, plan succeeds.
- [ ] `apply` prints `file_path` output.
- [ ] Changing `greeting` in `.tfvars` changes the plan diff.

---

## Break & repair

| Symptom | Likely cause | Fix |
|---|---|---|
| `No value for required variable` | Missing tfvars / `-var` | Add `terraform.tfvars` or default in variable |
| Output not shown | Old apply | Re-run `apply` or `terraform output` |
| Wrong file name | Typo in local | Fix `locals` block; plan again |
| Sensitive data in output | Marked wrong | Use `sensitive = true` on outputs for secrets (AWS keys never in outputs) |

**Break on purpose:** Remove `default` from a variable, delete tfvars entry, run `plan` — see clear error message.

---

## You should now be able to…

- Declare `variable`, `locals`, and `output` blocks.
- Parameterize a project with `terraform.tfvars`.
- Read output values after apply.

Previous: [State and plan/apply](02-state-plan-apply.md) · Next: [Modules](04-modules.md)
