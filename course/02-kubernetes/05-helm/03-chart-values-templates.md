# Chart, values, and templates

**Prerequisite:** [Install Helm CLI](02-install-helm-cli.md).

---

## Explanation

A **chart** is Helm’s **package** format — a directory (or a `.tgz` file) that describes a Kubernetes app.

Three pieces work together:

| Piece | File / folder | Role |
|---|---|---|
| **Chart metadata** | `Chart.yaml` | Name, version, description of the package |
| **Values** | `values.yaml` | Default configuration (inputs) |
| **Templates** | `templates/*.yaml` | Kubernetes YAML with placeholders — become real manifests when rendered |

```text
values.yaml  ──►  {{ .Values.replicaCount }}  in templates  ──►  Deployment spec
```

**Template** = YAML + [Go template](https://pkg.go.dev/text/template) syntax. Helm replaces `{{ .Values.foo }}` with content from **values** (defaults in `values.yaml`, or overrides from CLI).

**Chart vs release vs package:**

| Term | Meaning |
|---|---|
| **Chart** | Source package on disk (folder) |
| **Package** | Compressed chart archive `mychart-1.0.0.tgz` (`helm package`) |
| **Release** | Chart **installed** in the cluster with a name (lesson 05) |

---

## Implementation — Step 1: Scaffold a chart with `helm create`

```bash
mkdir -p ~/kube-lab/helm
cd ~/kube-lab/helm
helm create hello-chart
tree hello-chart -L 2
```

**What `helm create` gives you:**

```text
hello-chart/
├── Chart.yaml          # chart name + version
├── values.yaml         # defaults
├── charts/             # dependent charts (subcharts) — empty at first
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    ├── NOTES.txt
    └── _helpers.tpl    # reusable template snippets
```

You will simplify this chart in [08-package-custom-chart.md](08-package-custom-chart.md). For now, use it to learn structure.

---

## Implementation — Step 2: Read `Chart.yaml`

```bash
cat hello-chart/Chart.yaml
```

Key fields:

| Field | Meaning |
|---|---|
| `apiVersion: v2` | Helm 3 chart format |
| `name` | Chart name (used in package file name) |
| `version` | Chart version (semver — bump when chart changes) |
| `appVersion` | Version of the app inside (informational) |

---

## Implementation — Step 3: Read `values.yaml`

```bash
head -n 30 hello-chart/values.yaml
```

Typical keys: `replicaCount`, `image.repository`, `image.tag`, `service.type`.

**Values** are the knobs installers turn without editing templates. Example override (preview):

```bash
helm template demo ./hello-chart --set replicaCount=3
```

`--set` changes a value at render time.

---

## Implementation — Step 4: Inspect a template

```bash
grep -n "replicaCount\|\.Values" hello-chart/templates/deployment.yaml | head -n 10
```

You will see lines like:

```yaml
replicas: {{ .Values.replicaCount }}
image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
```

- `.Values.*` — from `values.yaml` or `--set` / `-f`
- `.Chart.*` — from `Chart.yaml`
- `.Release.*` — set at **install** time (release name, namespace)

---

## Implementation — Step 5: Render locally with `helm template`

**Why:** See final YAML **before** touching the cluster — same idea as `kubectl apply --dry-run=client`.

Default values:

```bash
cd ~/kube-lab/helm
helm template hello-local ./hello-chart -n kube-lab | head -n 40
```

Override values:

```bash
helm template hello-local ./hello-chart -n kube-lab \
  --set replicaCount=1 \
  --set service.type=ClusterIP | grep -A2 "replicas:"
```

Custom values file:

```bash
cat <<'EOF' > hello-chart/values-lab.yaml
replicaCount: 1
service:
  type: ClusterIP
EOF

helm template hello-local ./hello-chart -n kube-lab -f hello-chart/values-lab.yaml
```

No release is created — this is **dry render only**.

---

## Verification

- [ ] `hello-chart/Chart.yaml`, `values.yaml`, and `templates/` exist
- [ ] `helm template` prints valid Kubernetes YAML (apiVersion, kind, metadata)
- [ ] `--set replicaCount=3` changes `replicas:` in rendered output

---

## Break & repair

**`helm template` error: parse error in template**  
→ Syntax error in `templates/*.yaml`. Check `{{ }}` pairs and indentation.

**Empty or wrong output**  
→ Add `--debug` to see which templates failed: `helm template demo ./hello-chart --debug`

**Confused chart vs release name**  
→ `hello-local` in `helm template` is a **fake release name** for rendering only. Real releases come from `helm install` (next lessons).

---

## Cleanup / revert

```bash
rm -rf ~/kube-lab/helm/hello-chart
```

Or keep it for later lessons.

---

## You should now be able to…

- Describe the role of `Chart.yaml`, `values.yaml`, and `templates/`.
- Explain how **values** feed **templates** to produce manifests.
- Use `helm create` and `helm template` to inspect a chart without installing.

Next: [Repositories](04-repositories.md)

Previous: [Install Helm CLI](02-install-helm-cli.md)
