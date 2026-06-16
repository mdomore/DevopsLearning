# DevOps Learning Course

Comprehensive hands-on course covering **Docker**, **Kubernetes**, **Terraform**, and **AWS**.

**For learners:** Every section explains concepts before commands. Start at [00-getting-started](00-getting-started/learning-path.md). No prior DevOps or programming background required.

## How this course is organized

- Each **chapter** is a directory under `course/` (numbered `00`–`04`).
- Each **section** is a separate Markdown file inside that chapter.
- Every chapter has a `README.md` index listing its sections in order.

## Recommended learning order

| Order | Chapter | Why |
|---|---|---|
| 0 | [00-getting-started](00-getting-started/) | Install tools; create local cluster — no concepts yet |
| 1 | [01-docker](01-docker/) | Containers are the unit Kubernetes runs |
| 2 | [02-kubernetes](02-kubernetes/) | Orchestration, networking, workloads |
| 3 | [04-aws](04-aws/) | Cloud basics before provisioning AWS with Terraform |
| 4 | [03-terraform](03-terraform/) | IaC; AWS labs (VPC, EKS) after AWS chapter |

**Terraform note:** Sections 1–5 (HCL, state, variables, modules) do not require AWS. Sections 6–7 (VPC, EKS labs) require the [AWS chapter](04-aws/) first.

## Chapters

| Chapter | Path | Status |
|---|---|---|
| Getting started | [00-getting-started](00-getting-started/) | Ready (tooling guides) |
| Docker | [01-docker](01-docker/) | Ready |
| Kubernetes | [02-kubernetes](02-kubernetes/) | Ready (foundations + labs) |
| AWS | [04-aws](04-aws/) | Ready (concepts; cloud labs in later phases) |
| Terraform | [03-terraform](03-terraform/) | Ready (HCL + previews; full AWS labs after AWS chapter) |

## Validation rule

Mark a topic as learned only when you can:

- explain it in one sentence,
- implement it hands-on,
- verify it with at least two checks.

## Personal notes

Use the [notes template](02-kubernetes/templates/notes-template.md) after each lab session.
