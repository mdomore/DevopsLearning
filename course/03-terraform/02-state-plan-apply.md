# State and plan/apply

## Prerequisites

- [HCL, providers, resources](01-hcl-providers-resources.md) — you have a working `local_file` example and ran `init` / `plan`.

One-sentence recap: **Terraform** compares your `.tf` files to **state** to decide what to create, change, or destroy.

---

## Explanation

**State** is Terraform's memory of what it manages. By default it lives in a file **`terraform.tfstate`** in the project directory (JSON — do not edit by hand).

| Command | Plain meaning |
|---|---|
| `terraform plan` | Diff: desired config vs state vs real world |
| `terraform apply` | Execute the plan (with confirmation) |
| `terraform destroy` | Remove all resources in state |
| `terraform refresh` | Update state from real infrastructure (legacy debugging; plan usually enough) |

**Drift** — someone changed infrastructure outside Terraform (console, CLI). Next `plan` shows differences; apply may revert or fail depending on settings.

**Safe apply practices:**

1. Always run **plan** before **apply** in shared or production environments.
2. Read the plan: `+` create, `~` change, `-` destroy.
3. Use version control (Git) for `.tf` files — not for secrets or state (later: remote state).
4. Never commit `terraform.tfstate` with secrets inside (our local file example is fine for learning).

---

## What you will do

1. Apply the intro project and inspect `terraform.tfstate`.
2. Change a resource attribute and observe the plan diff.
3. Apply again and confirm reality matches state.
4. Destroy resources when finished.

Continue using `~/learning/terraform-01-intro` from section 01 (or recreate it).

---

## Implementation

### Step 1 — Apply if not already done

```bash
cd ~/learning/terraform-01-intro
terraform apply
```

### Step 2 — Inspect state file

```bash
terraform show
```

Or open `terraform.tfstate` — find `"type": "local_file"` and attributes like `content` and `filename`.

### Step 3 — Introduce drift (change config)

Edit `main.tf` — change the greeting:

```hcl
resource "local_file" "hello" {
  content  = "Updated by Terraform plan/apply lab\n"
  filename = "${path.module}/hello-from-terraform.txt"
}
```

Plan:

```bash
terraform plan
```

Expected: `~` update in place on `local_file.hello` (content change).

### Step 4 — Apply the change

```bash
terraform apply
```

Verify:

```bash
cat hello-from-terraform.txt
```

### Step 5 — Manual drift (break on purpose)

Outside Terraform, edit `hello-from-terraform.txt` by hand. Run:

```bash
terraform plan
```

Terraform wants to change the file back to match `main.tf` — that is drift detection.

### Step 6 — Destroy (cleanup)

```bash
terraform destroy
```

Confirm file removal (resource gone; you may keep the empty directory).

---

## Verification checklist

- [ ] `terraform show` lists your `local_file` resource.
- [ ] Changing `content` produces a visible plan diff.
- [ ] After apply, file content matches `main.tf`.
- [ ] After manual edit, `plan` shows drift.
- [ ] `terraform destroy` removes the managed file.

---

## Break & repair

| Symptom | Likely cause | Fix |
|---|---|---|
| Plan wants unexpected destroy/recreate | Renamed resource block | Use `moved` blocks (advanced) or accept replace in lab |
| State out of sync | Deleted resource manually | `terraform refresh` or fix resource; worst case remove from state (advanced) |
| `Error acquiring state lock` | Stale lock (remote state later) | Follow [Remote state](05-remote-state.md) lock guidance |
| Applied wrong project | Wrong directory | `pwd`; use separate folders per stack |

**Break on purpose:** Delete `terraform.tfstate` while the file still exists — run `plan`; understand Terraform may try to create duplicate or show import need.

---

## You should now be able to…

- Explain what `terraform.tfstate` is for.
- Use `plan` to preview changes and detect drift.
- Apply and destroy safely in a local lab project.

Previous: [HCL, providers, resources](01-hcl-providers-resources.md) · Next: [Variables and outputs](03-variables-outputs.md)
