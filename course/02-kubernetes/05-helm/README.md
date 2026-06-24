# Helm — package and release on Kubernetes

**Prerequisite:** Complete [hands-on labs](../04-hands-on-labs/) through at least Deployment + Service (labs 03 and 08). Comfortable with `kubectl apply`, YAML, and namespace `kube-lab`.

**When in the course:** After Phase 4 (ConfigMap, Secret, PVC…) and **before** AWS / Terraform.

Previous: [Hands-on labs](../04-hands-on-labs/) · Next: [AWS](../../04-aws/)

---

## Explanation — why this section exists

Real apps need many YAML files. Helm bundles them into a **chart**, installs a **release**, and manages **upgrade**, **rollback**, and **uninstall** — with **values** and **templates** so dev/staging/prod differ without copy-paste.

This chapter covers **all** core Helm topics:

**chart · values · templates · repository · CLI · release · install · uninstall · upgrade · rollback · package**

---

## Sections (in order)

| # | Topic | File | Covers |
|---|---|---|---|
| 1 | What is Helm? | [01-what-is-helm.md](01-what-is-helm.md) | Vocabulary map, mental model |
| 2 | Install Helm + CLI | [02-install-helm-cli.md](02-install-helm-cli.md) | **CLI**, command overview |
| 3 | Chart, values, templates | [03-chart-values-templates.md](03-chart-values-templates.md) | **Chart**, **values**, **templates**, `helm create`, `helm template` |
| 4 | Repositories | [04-repositories.md](04-repositories.md) | **Repository**, `repo add/search` |
| 5 | Install and release | [05-install-release.md](05-install-release.md) | **install**, **release**, `list`, `status` |
| 6 | Upgrade and rollback | [06-upgrade-rollback.md](06-upgrade-rollback.md) | **upgrade**, **rollback**, **revision**, `history` |
| 7 | Uninstall | [07-uninstall.md](07-uninstall.md) | **uninstall**, cleanup |
| 8 | Package + custom chart | [08-package-custom-chart.md](08-package-custom-chart.md) | **package** (`.tgz`), custom chart capstone |

Each lesson: Explanation → Implementation → Verification → Break & repair → Cleanup → You should now be able to…

---

## Suggested flow

1. Finish core [Kubernetes labs](../04-hands-on-labs/) (minimum: Deployment + Service; ideally through Ingress).
2. Work through sections **01 → 08** on cluster `kube-lab` (context `kind-kube-lab` or `docker-desktop`).
3. Keep [glossary](../03-glossary/glossary.md) open for quick lookup.

**Install:** `helm` is installed in lesson 02, not Phase 0.

---

## Chapter checklist

- [ ] **CLI** — `helm version`, help, same kubeconfig as `kubectl`
- [ ] **Chart** — `Chart.yaml`, `values.yaml`, `templates/`
- [ ] **Values** — defaults, `--set`, `-f`
- [ ] **Templates** — `helm template`, `{{ .Values }}`
- [ ] **Repository** — add, search, show values
- [ ] **Release** — install, list, status, get manifest/values
- [ ] **install** / **upgrade** / **rollback** / **uninstall**
- [ ] **Package** — `helm package`, install from `.tgz`
- [ ] Custom chart deployed in `kube-lab`

---

## Link to later chapters

- **AWS EKS:** Helm deploys add-ons and apps on production clusters.
- **Terraform:** Helm provider installs **releases** from CI — learn CLI first.

---

## You should now be able to… (after lesson 08)

- Define every term in the chapter checklist and use the matching **CLI** command.
- Install from a **repository** and from a local **package**.
- Author, **package**, **install**, **upgrade**, **rollback**, and **uninstall** a custom chart.
