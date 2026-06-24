# Repositories

**Prerequisite:** [Chart, values, and templates](03-chart-values-templates.md).

A **repository** (repo) is a HTTP server that hosts **packaged charts** (`.tgz` files) plus an index. Like Docker Hub for images — but for Helm charts.

---

## Explanation

### Why repositories exist

You could copy a chart folder from Git every time. Teams prefer:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-db bitnami/postgresql -n kube-lab
```

Helm downloads the **package** from the repo, unpacks it, renders templates, and installs.

### Repo vs chart vs package

| Term | Meaning |
|---|---|
| **Repository** | Catalog of charts (URL + index) |
| **Chart** | One product in the catalog (`bitnami/nginx`) |
| **Package** | Versioned archive (`nginx-15.0.0.tgz`) stored in the repo |

### Common commands

| Command | Action |
|---|---|
| `helm repo add <name> <url>` | Register a repo locally |
| `helm repo update` | Refresh index from all registered repos |
| `helm search repo <keyword>` | Search chart names and descriptions |
| `helm show chart <repo/chart>` | Metadata without installing |
| `helm show values <repo/chart>` | Default **values** for a chart |

---

## Implementation — Step 1: List and add a repository

See what is already configured:

```bash
helm repo list
```

Add Bitnami (widely used for learning; large catalog):

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

### Verification

```bash
helm repo list
```

Expected: `bitnami` with URL `https://charts.bitnami.com/bitnami`.

---

## Implementation — Step 2: Search and inspect charts

Search nginx charts:

```bash
helm search repo nginx
```

Inspect chart metadata and default values **without installing**:

```bash
helm show chart bitnami/nginx
helm show values bitnami/nginx | head -n 40
```

**Why `helm show values`?** Every chart exposes different knobs. Read defaults before `install` or `--set`.

---

## Implementation — Step 3: Remove a repo (optional)

```bash
helm repo remove bitnami
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

Useful when a repo URL changes or you want a clean list.

---

## Verification

- [ ] `helm repo list` shows at least one repository
- [ ] `helm search repo nginx` returns results
- [ ] `helm show values bitnami/nginx` prints YAML defaults

---

## Break & repair

**`helm repo add` fails (network / 403)**  
→ Check internet; try `helm repo update` again. Corporate proxy may block chart URLs.

**Search returns nothing**  
→ Run `helm repo update` after `add`.

**Wrong chart installed later**  
→ Always use full name `repo/chart` (e.g. `bitnami/nginx`, not just `nginx`).

---

## Cleanup / revert

Removing a repo does **not** uninstall releases that came from it:

```bash
helm repo remove bitnami
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

---

## You should now be able to…

- Explain what a Helm **repository** is.
- Add, update, and search repos.
- Inspect chart metadata and default **values** before install.

Next: [Install and release](05-install-release.md)

Previous: [Chart, values, and templates](03-chart-values-templates.md)
