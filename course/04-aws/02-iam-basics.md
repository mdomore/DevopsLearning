# IAM basics

## Prerequisites

- [AWS account and CLI setup](01-account-cli-setup.md) — CLI working (`aws sts get-caller-identity` with your default profile, or `--profile lab` if you created one).
- One-sentence recap: **IAM** controls who can do what in AWS; the CLI uses an IAM user's keys via a profile.

---

## Explanation

**IAM** (Identity and Access Management) is AWS's permission system.

Main pieces:

| Term | Plain meaning |
|---|---|
| **IAM user** | A long-lived identity for a person or script (CLI keys belong here) |
| **IAM role** | A temporary identity that a service or person *assumes* (no permanent password) |
| **Policy** | A JSON document listing allowed or denied **actions** on **resources** |
| **Permission policy** | Attached to a user/role/group — defines what they can do |
| **Trust policy** | Attached to a role — defines *who* is allowed to assume that role |

**Least privilege** means granting only the permissions needed for the task — not `AdministratorAccess` for a hello-world lab unless you accept the risk on a personal sandbox account.

For this course you will see IAM again when:

- Terraform runs as your lab user ([Terraform chapter](../03-terraform/)).
- **EKS** (Elastic Kubernetes Service — AWS-managed Kubernetes) uses roles so nodes and the control plane can call AWS APIs.
- Pods use **IRSA** (IAM Roles for Service Accounts) in advanced setups — out of scope for now, but the pattern is: role + trust policy, not long-lived keys inside the cluster.

---

## What you will do

1. Open IAM in the console and find your lab user.
2. Read an AWS managed policy document.
3. Attach a **read-only** policy and confirm limited access via CLI.
4. Explain user vs role in one sentence.

---

## Implementation

### Step 1 — Inspect your lab user

1. AWS Console → search **IAM** → **Users** → open `devops-course-lab` (or your lab user).
2. Open the **Permissions** tab. Note attached **policies** (permission documents).

### Step 2 — Read a policy

1. IAM → **Policies** → open e.g. `ReadOnlyAccess` (AWS managed).
2. Click **JSON**. You will see `"Effect": "Allow"` and `"Action"` lists like `"ec2:Describe*"`.

You do not need to memorize JSON — notice that policies are explicit lists of API actions.

### Step 3 — Attach read-only access (sandbox account only)

On a **personal lab account** where you accept broader read access:

1. Users → your lab user → **Add permissions** → **Attach policies directly**.
2. Attach **`ReadOnlyAccess`** (in addition to whatever lab policy you already have).
3. Save.

Verify — this should succeed (read):

```bash
aws ec2 describe-vpcs --profile lab --region eu-west-1
```

Verify — this should **fail** with `AccessDenied` if you only have read-only (no create):

```bash
aws ec2 create-vpc --cidr-block 10.99.0.0/16 --profile lab --region eu-west-1
```

If create succeeds, your user still has write permissions from another policy — note which policies are attached.

### Step 4 — User vs role (concept)

**User:** "Alice's CLI keys — always Alice."

**Role:** "The EKS node may temporarily become `NodeRole` to pull images from ECR."

Roles use **trust policies** so AWS knows which service or account may assume them.

---

## Verification checklist

- [ ] You can find your lab user and its attached policies in the console.
- [ ] You can open a policy JSON and point to one `Action`.
- [ ] `aws ec2 describe-vpcs --profile lab` works with read-only attached.
- [ ] You can explain **IAM user vs IAM role** in one sentence (see Step 4).

---

## Break & repair

| Symptom | Likely cause | Fix |
|---|---|---|
| Everything denied | No policies attached | Attach a lab or read-only policy |
| Cannot create resources in later labs | Read-only only | Add a controlled write policy for lab work (not on production accounts) |
| `AccessDenied` on `sts:AssumeRole` | Missing trust on role | Fix trust policy on the role (EKS labs later) |

**Break on purpose:** Detach all policies from a *test* user (not your only admin), try `describe-vpcs`, see `AccessDenied`, reattach `ReadOnlyAccess`.

---

## You should now be able to…

- Name the four IAM ideas: user, role, permission policy, trust policy.
- Apply **least privilege** when choosing lab policies.
- Test permissions with one allowed and one denied CLI call.

Previous: [Account and CLI setup](01-account-cli-setup.md) · Next: [VPC networking](03-vpc-networking.md)
