# Optional — Create an AWS lab IAM user (Path B)

Use this guide only if you chose **Path B**: a separate `lab` CLI profile so course work stays isolated from your other AWS usage.

If you already have a working default profile, you can skip this file and use [Path A in Step 7](00-tooling-setup-macos.md#step-7--install-and-configure-aws-cli).

---

## Why create a separate IAM user?

**IAM** (Identity and Access Management) controls *who* can do *what* in AWS.

| Approach | Risk |
|---|---|
| Use your main personal/prod credentials for labs | One wrong `terraform destroy` or typo can affect real resources |
| Separate **lab IAM user** + `lab` profile | Course commands use different keys; your default profile stays untouched |

Think of the lab user as a **dedicated login for learning**, not your main account keys.

---

## What you will create

1. An IAM user named e.g. `devops-course-lab`
2. A permissions policy (what this user is allowed to do)
3. **Access keys** (ID + secret) for the CLI
4. A local CLI profile `lab` that stores those keys

---

## Step 1 — Open IAM in the AWS Console

1. Sign in to https://console.aws.amazon.com/
2. Search the top bar for **IAM**
3. Open **IAM** → **Users**

You need permission to create IAM users. If you get “access denied”, ask your account admin or use Path A with your existing profile.

---

## Step 2 — Create the IAM user

1. Click **Create user**
2. **User name:** `devops-course-lab` (any clear name is fine)
3. **Do not** enable “Provide user access to the AWS Management Console” for this lab user — we only need CLI access keys.
4. Click **Next**

---

## Step 3 — Attach permissions

The user needs enough rights for later course labs (VPC, EKS, ECR). For a **personal learning account**, attach:

**Option 1 (simplest for learning):**  
- Policy: **PowerUserAccess** (AWS managed)  
- Allows almost everything except IAM/user management. Good for solo lab accounts.

**Option 2 (stricter, if your org requires it):**  
- Ask your admin for a custom “DevOps lab” policy, or start with **ReadOnlyAccess** for Phase 0 only and expand later.

1. Select **Attach policies directly**
2. Search for `PowerUserAccess`
3. Check the box → **Next** → **Create user**

### Why PowerUserAccess for learning?

Later chapters create VPCs, EKS clusters, and load balancers. A read-only user cannot run those labs. PowerUserAccess is a common learning compromise: powerful enough for labs, but cannot create new IAM admin users.

**Never use the root account access keys for CLI work.**

---

## Step 4 — Create access keys

1. Open the user you just created (`devops-course-lab`)
2. Tab **Security credentials**
3. Scroll to **Access keys** → **Create access key**
4. Use case: **Command Line Interface (CLI)** → confirm the warning → **Next**
5. Optional description: `devops course lab mac/linux`
6. **Create access key**

You will see:

- **Access key ID** (example: `AKIA...`)
- **Secret access key** (shown **once**)

Copy both to a password manager or download the `.csv` file. You cannot retrieve the secret again later.

---

## Step 5 — Configure the `lab` profile on your machine

In your terminal:

```bash
aws configure --profile lab
```

| Prompt | What to enter |
|---|---|
| AWS Access Key ID | The Access key ID from Step 4 |
| AWS Secret Access Key | The Secret access key from Step 4 |
| Default region name | Your lab region, e.g. `eu-west-1` |
| Default output format | `json` |

This writes a **new** profile only. It does not modify `[default]`.

---

## Verification

Confirm the lab profile points at the new user:

```bash
aws sts get-caller-identity --profile lab
```

Expected: `Arn` contains `devops-course-lab` (or the name you chose).

Confirm your original profile still works:

```bash
aws sts get-caller-identity
```

Expected: still shows your original user — unchanged.

List all profiles:

```bash
aws configure list-profiles
```

---

## How to switch (reminder)

```bash
aws s3 ls --profile lab           # one command
export AWS_PROFILE=lab            # whole shell session
unset AWS_PROFILE                 # back to default
```

---

## Break & repair

**Access denied when creating IAM user**  
→ Your current login lacks IAM admin rights. Use Path A, or ask an admin to create the lab user for you.

**Lost the secret access key**  
→ IAM → Users → your lab user → Security credentials → **Create access key** (new pair). Delete the old key.

**Lab user can read but not create VPC/EKS**  
→ Policy too restrictive. Attach `PowerUserAccess` (personal lab) or ask admin for a broader lab policy.

**Accidentally used lab keys in default profile**  
→ Run `aws configure` (no `--profile`) and restore your original default keys.

---

## Cleanup when you finish the course

1. Delete access keys for `devops-course-lab`
2. Delete the IAM user (if no longer needed)
3. Remove local profile (optional): edit `~/.aws/credentials` and remove the `[lab]` section

This avoids leaving unused keys on your account.

Previous: [Tooling setup index](00-tooling-setup.md)
