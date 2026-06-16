# Remote state

## Prerequisites

- [State and plan/apply](02-state-plan-apply.md) — local `terraform.tfstate` behavior.
- [AWS account and CLI setup](../04-aws/01-account-cli-setup.md) — profile `lab` (needed for hands-on backend).
- [IAM basics](../04-aws/02-iam-basics.md) — your user can create S3 and DynamoDB resources for the lab.

---

## Explanation

**Local state** (`terraform.tfstate` on your laptop) breaks down when:

- Two people run Terraform on the same infrastructure.
- You switch computers — state does not follow you.
- CI/CD pipelines need a shared record of what exists.

**Remote state** stores state in a shared backend — commonly:

| Piece | Role |
|---|---|
| **S3 bucket** | Stores the `terraform.tfstate` object |
| **DynamoDB table** | **State locking** — only one `apply` at a time |
| **Encryption** | Protect state at rest (state may contain sensitive values) |

**State locking** — if Alice is mid-apply, Bob's apply waits or fails fast instead of corrupting state.

**Backend** — Terraform setting that says where state lives (`backend "s3" { ... }`).

Teams also use **Terraform Cloud** or other backends; this course uses AWS S3 + DynamoDB to match the AWS chapter.

---

## What you will do

1. Understand why local-only state is risky for teams.
2. Preview S3 + DynamoDB backend configuration.
3. (Hands-on) Create backend resources and migrate state — **coming in this chapter** as a capstone after you confirm S3/DynamoDB permissions.

If you are not ready for AWS resources yet, complete Steps 1–2 conceptually and return after [AWS IAM](../04-aws/02-iam-basics.md).

---

## Implementation

### Step 1 — Concept check (local)

In any prior lab folder with state:

```bash
ls -la terraform.tfstate
```

Imagine committing this to Git — teammates would overwrite each other. Remote state fixes that.

### Step 2 — Backend block (preview)

`backend.tf` (example — do not apply until bucket/table exist):

```hcl
terraform {
  backend "s3" {
    bucket         = "devops-course-tfstate-UNIQUE_SUFFIX"
    key            = "terraform-03/vpc/terraform.tfstate"
    region         = "eu-west-1"
    profile        = "lab"
    dynamodb_table = "devops-course-tf-lock"
    encrypt        = true
  }
}
```

Replace `UNIQUE_SUFFIX` with something globally unique (S3 bucket names are global).

### Step 3 — One-time bootstrap (hands-on)

Create supporting resources (console or CLI). CLI example:

```bash
export AWS_PROFILE=lab
export AWS_REGION=eu-west-1
export BUCKET=devops-course-tfstate-$(aws sts get-caller-identity --query Account --output text)

aws s3api create-bucket --bucket "$BUCKET" --region "$AWS_REGION" \
  --create-bucket-configuration LocationConstraint="$AWS_REGION"

aws dynamodb create-table --table-name devops-course-tf-lock \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST --region "$AWS_REGION"
```

Enable versioning on the bucket (console: S3 → bucket → **Versioning** → Enable) — helps recover from bad state writes.

### Step 4 — Migrate local state to remote

1. Add the `backend "s3"` block with your real bucket name.
2. Run:

```bash
terraform init -migrate-state
```

Answer `yes` to copy existing state to S3.

3. Verify object in S3:

```bash
aws s3 ls "s3://$BUCKET/terraform-03/" --profile lab
```

4. Run plan — should show no changes if infrastructure unchanged:

```bash
terraform plan
```

### Step 5 — Lock test (optional)

Run two `terraform apply` in parallel on the same project — second should wait or error on lock. Do not force-unlock unless you are sure no apply is running.

### Teardown note

Keep the state bucket if other stacks use it. Empty bucket before delete; delete DynamoDB table when course complete ([Cost control](../04-aws/07-cost-control.md)).

---

## Verification checklist

- [ ] You can explain why teams use remote state instead of emailing `tfstate`.
- [ ] You can name S3 (storage) and DynamoDB (locking) roles.
- [ ] (Hands-on) `terraform init -migrate-state` succeeded.
- [ ] (Hands-on) `plan` runs without recreating all resources unexpectedly.
- [ ] State object visible in S3.

---

## Break & repair

| Symptom | Likely cause | Fix |
|---|---|---|
| `Error loading state` | Wrong bucket/key/region | Fix backend block; re-init |
| `AccessDenied` on S3 | IAM policy | Add s3:GetObject/PutObject on bucket |
| Lock never releases | Crashed apply | `terraform force-unlock LOCK_ID` only when safe |
| Plan wants full recreate | Wrong state key | Stop; fix key before apply |

**Break on purpose:** Change `key` in backend to a new path, re-init — understand Terraform sees empty state (dangerous) — revert key.

---

## You should now be able to…

- Describe remote state and state locking.
- Configure an S3 backend with DynamoDB locking (preview + hands-on).
- Migrate local state with `terraform init -migrate-state`.

Previous: [Modules](04-modules.md) · Next: [AWS VPC lab](06-aws-vpc-lab.md)
