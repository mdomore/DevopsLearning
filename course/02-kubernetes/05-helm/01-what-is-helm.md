# What is Helm?

**Prerequisite:** You have applied Kubernetes YAML by hand ([`kubectl` basics](../01-foundations/03-kubectl-basics.md)) and know what a Deployment and Service are ([labs 03](../04-hands-on-labs/03-deployment.md) and [08](../04-hands-on-labs/08-clusterip-service.md)).

This page is **conceptual** — no install yet. Install comes in [02-install-helm-cli.md](02-install-helm-cli.md).

---

## Explanation

### The mess Helm fixes

Imagine deploying a web app that needs:

- a Deployment (3 replicas)
- a Service (ClusterIP)
- an Ingress (HTTPS route)
- a ConfigMap (app config)
- a Secret (API key)

You could maintain five YAML files and run:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
# … three more times
```

That works for learning — you did exactly that in the labs. In a team, problems appear quickly:

- **Duplication** — staging and prod need almost the same files with small differences (image tag, replica count).
- **No single “version”** — hard to answer “what is running in prod right now?”
- **Risky upgrades** — easy to apply files in the wrong order or forget one file.
- **No one-click undo** — rollback means finding old YAML in Git and re-applying.

**Helm** is a **package manager for Kubernetes**. It bundles manifests into a **chart**, installs them as a **release**, and tracks **revisions** for **upgrade** and **rollback**.

---

## Full vocabulary — every topic in this chapter

| Term | One-line meaning | Where you practice |
|---|---|---|
| **CLI** | The `helm` command on your laptop | [02-install-helm-cli.md](02-install-helm-cli.md) |
| **Chart** | Folder: `Chart.yaml` + `values.yaml` + `templates/` | [03-chart-values-templates.md](03-chart-values-templates.md) |
| **Values** | Configuration inputs (`values.yaml`, `--set`, `-f`) | [03](03-chart-values-templates.md), [05](05-install-release.md), [06](06-upgrade-rollback.md) |
| **Templates** | YAML with `{{ .Values… }}` placeholders | [03-chart-values-templates.md](03-chart-values-templates.md) |
| **Repository** | Online catalog of packaged charts | [04-repositories.md](04-repositories.md) |
| **Package** | Chart archive `.tgz` (`helm package`) | [08-package-custom-chart.md](08-package-custom-chart.md) |
| **Release** | One installed chart instance (name + namespace) | [05-install-release.md](05-install-release.md) |
| **Revision** | History entry after each install/upgrade/rollback | [06-upgrade-rollback.md](06-upgrade-rollback.md) |
| **install** | Create release (revision 1) | [05-install-release.md](05-install-release.md) |
| **upgrade** | Change release → new revision | [06-upgrade-rollback.md](06-upgrade-rollback.md) |
| **rollback** | Restore an older revision | [06-upgrade-rollback.md](06-upgrade-rollback.md) |
| **uninstall** | Delete release and its resources | [07-uninstall.md](07-uninstall.md) |

### How they connect

```text
Repository  ──►  chart (.tgz package or folder)
                      │
                      │  values.yaml + templates/
                      ▼
                 helm install  ──►  RELEASE (rev 1)
                      │
                      ├── helm upgrade  ──►  rev 2, 3, …
                      ├── helm rollback   ──►  rev N (copy of old config)
                      └── helm uninstall  ──►  gone
```

Helm does **not** replace the Kubernetes API. It **generates** Deployment, Service, etc. and sends them to the cluster — you still debug with `kubectl` when needed ([debugging containers](../../01-docker/00-debugging-containers.md) mindset applies).

---

### Helm 3 (what this course uses)

- **No Tiller** — the **CLI** talks to the Kubernetes API directly.
- Release metadata stored in the namespace (Secrets).
- One **release name** per namespace.

---

### When teams use Helm

- Install standard apps from a **repository** (ingress, monitoring, databases).
- **Package** internal apps as **charts** with **values** per environment.
- **Upgrade** and **rollback** as one operation on EKS and other clusters.

### When you still use plain `kubectl apply`

- Learning a new resource type for the first time.
- One-off debug objects.
- Kubernetes labs **before** this Helm section.

---

## Comparison — same app, two ways

**Plain YAML (labs):**

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

**Helm (this chapter):**

```bash
helm install my-web ./web-demo-chart -n kube-lab
helm upgrade my-web ./web-demo-chart -n kube-lab -f values-prod.yaml
helm rollback my-web 1 -n kube-lab
helm uninstall my-web -n kube-lab
```

---

## Verification (self-check, no commands)

- What is the difference between **chart**, **package**, and **release**?
- What role do **values** play vs **templates**?
- Name four **CLI** commands besides `install`.

---

## Break & repair (conceptual)

**“Helm replaces Kubernetes”** → False. It **packages** Kubernetes resources.

**“Learn Helm before `kubectl`”** → Wrong order. Do [labs](../04-hands-on-labs/) first.

---

## You should now be able to…

- Map every Helm term in this chapter to its lesson.
- Explain why teams add Helm after raw YAML.

Next: [Install Helm and the CLI](02-install-helm-cli.md)

Previous: [Helm chapter index](README.md)
