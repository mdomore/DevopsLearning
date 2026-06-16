# AWS account and CLI setup

## Prerequisites

- Tools installed from [Getting started — tooling setup](../00-getting-started/00-tooling-setup.md) (Terraform and AWS CLI).
- A terminal you can copy commands into (covered in the same chapter).

---

## Explanation

**AWS** (Amazon Web Services) is a cloud platform: you rent computers, storage, and networking over the internet instead of buying physical servers.

To work from your laptop, you use the **AWS CLI** — a command-line program that talks to AWS APIs (Application Programming Interfaces — the way programs request services over the network).

**Authentication** means proving who you are. AWS uses:

- **Access key ID** — like a username for the CLI
- **Secret access key** — like a password (never share it or commit it to Git)

A **profile** is a named set of credentials stored on your machine (in `~/.aws/credentials`). Using a separate **`lab` profile** keeps course work isolated from any other AWS account you use.

**Do not use the root account** (the email you signed up with) for daily CLI work. Root has unlimited power; if keys leak, your whole account is at risk. Create a normal **IAM user** instead (next section goes deeper).

**Region** is the geographic AWS data center (for example `eu-west-1` in Ireland). Many resources are tied to one region. Pick one near you and stay consistent in this course.

---

## What you will do

1. Confirm you can sign in to the AWS web console.
2. Create or reuse a **lab IAM user** with access keys (optional detailed steps: [Optional — AWS lab IAM user](../00-getting-started/00-aws-lab-iam-user.md)).
3. Configure the CLI with profile `lab`.
4. Verify the CLI sees your account.

---

## Implementation

### Step 1 — Sign in to the AWS console

1. Open https://console.aws.amazon.com/ and sign in.
2. Note your **Account ID** (12 digits): click your account name (top right) → **Account**.

If you do not have an account yet, create one at https://aws.amazon.com/ — you will need a credit card for verification; we use cost controls later in this chapter.

### Step 2 — Create lab access keys (if needed)

Follow [Optional — AWS lab IAM user](../00-getting-started/00-aws-lab-iam-user.md) to create user `devops-course-lab` and download **Access key ID** + **Secret access key**.

If you already have keys for a dedicated lab user, skip to Step 3.

### Step 3 — Configure the AWS CLI

Run on your **local machine** (not inside a cluster):

```bash
aws configure --profile lab
```

Answer the prompts:

| Prompt | Example | Notes |
|---|---|---|
| AWS Access Key ID | `AKIA...` | From IAM user |
| AWS Secret Access Key | (hidden) | Paste once; store safely |
| Default region name | `eu-west-1` | Or your chosen region |
| Default output format | `json` | Easy to read in scripts |

**Important:** Do not run `aws configure` without `--profile` unless you intend to overwrite your default profile.

### Step 4 — Verify identity

```bash
aws sts get-caller-identity --profile lab
```

Expected: JSON with `Account`, `UserId`, and `Arn` (Amazon Resource Name — a unique ID string for the user).

Optional — list profiles:

```bash
aws configure list-profiles
```

---

## Verification checklist

- [ ] You can sign in to the AWS console.
- [ ] Profile `lab` exists (`aws configure list-profiles` shows `lab`).
- [ ] `aws sts get-caller-identity --profile lab` returns your account ID (12 digits).
- [ ] You are **not** using root access keys for the CLI.

---

## Break & repair

| Symptom | Likely cause | Fix |
|---|---|---|
| `Unable to locate credentials` | Profile not configured | Run `aws configure --profile lab` again |
| `InvalidClientTokenId` | Wrong access key | Create new keys in IAM → Users → Security credentials |
| `AccessDenied` on later commands | User lacks permissions | Attach a broader lab policy (see [IAM basics](02-iam-basics.md)) |
| Wrong account ID | Wrong profile active | Always pass `--profile lab` or set `export AWS_PROFILE=lab` for the session |

**Break on purpose:** Temporarily set a wrong secret key in `~/.aws/credentials` under `[lab]`, run `get-caller-identity`, observe the error, then restore the correct secret.

---

## You should now be able to…

- Explain what the AWS CLI and a **profile** are.
- Configure profile `lab` without touching root credentials.
- Run `aws sts get-caller-identity --profile lab` and read the account ID.

Previous: [Kubernetes](../02-kubernetes/) · Next: [IAM basics](02-iam-basics.md)
