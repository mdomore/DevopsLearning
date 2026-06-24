# Package and custom chart

**Prerequisite:** [Chart, values, and templates](03-chart-values-templates.md), [Uninstall](07-uninstall.md) (clean `kube-lab` releases optional).

This lesson covers **`helm package`** (build a distributable **package**) and a minimal **custom chart** you author yourself.

---

## Explanation

### Package (.tgz)

A **package** is a compressed archive of the chart directory:

```text
web-demo-0.1.0.tgz  =  Chart.yaml + values.yaml + templates/ + …
```

Teams **package** charts to:

- Upload to a **repository**
- Pin exact versions in CI/CD
- Share internally without Git checkouts

```bash
helm package ./web-demo-chart
# produces web-demo-0.1.0.tgz (name-version from Chart.yaml)
```

Install from package:

```bash
helm install my-web web-demo-0.1.0.tgz -n kube-lab
```

### Custom chart goal

Build a small chart (Deployment + Service) — same objects as [Deployment lab](../04-hands-on-labs/03-deployment.md), but templated with **values**.

---

## Implementation — Step 1: Create a minimal custom chart

```bash
mkdir -p ~/kube-lab/helm/web-demo-chart/templates
cd ~/kube-lab/helm/web-demo-chart
```

**Chart.yaml:**

```bash
cat <<'EOF' > Chart.yaml
apiVersion: v2
name: web-demo
description: Minimal nginx chart for course lab
type: application
version: 0.1.0
appVersion: "1.27"
EOF
```

**values.yaml:**

```bash
cat <<'EOF' > values.yaml
replicaCount: 1
image:
  repository: nginx
  tag: "1.27"
service:
  type: ClusterIP
  port: 80
labels:
  app: web-demo
EOF
```

**templates/deployment.yaml:**

```bash
cat <<'EOF' > templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-web
  labels:
    app: {{ .Values.labels.app }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.labels.app }}
  template:
    metadata:
      labels:
        app: {{ .Values.labels.app }}
    spec:
      containers:
        - name: web
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 80
EOF
```

**templates/service.yaml:**

```bash
cat <<'EOF' > templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-web
  labels:
    app: {{ .Values.labels.app }}
spec:
  type: {{ .Values.service.type }}
  selector:
    app: {{ .Values.labels.app }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: 80
EOF
```

**Why `{{ .Release.Name }}`?** Prefixes resource names with the release name so two installs do not collide.

---

## Implementation — Step 2: Render and install

```bash
cd ~/kube-lab/helm
helm template test-render ./web-demo-chart -n kube-lab
helm install web-demo ./web-demo-chart -n kube-lab
kubectl get deploy,svc,pods -n kube-lab -l app=web-demo
```

---

## Implementation — Step 3: Package the chart

```bash
cd ~/kube-lab/helm
helm package web-demo-chart
ls -la web-demo-0.1.0.tgz
```

Helm reads `name` and `version` from `Chart.yaml` for the file name.

Install from the **package** (uninstall first if `web-demo` exists):

```bash
helm uninstall web-demo -n kube-lab
helm install web-demo-from-tgz web-demo-0.1.0.tgz -n kube-lab
helm list -n kube-lab
```

Same chart, two ways to reference it: **folder** or **`.tgz` package**.

---

## Implementation — Step 4: Upgrade packaged chart

Bump chart version after a change:

```bash
# Edit values default or templates, then bump Chart.yaml version to 0.2.0
sed -i '' 's/version: 0.1.0/version: 0.2.0/' web-demo-chart/Chart.yaml 2>/dev/null || \
  sed -i 's/version: 0.1.0/version: 0.2.0/' web-demo-chart/Chart.yaml

helm package web-demo-chart
helm upgrade web-demo-from-tgz web-demo-0.2.0.tgz -n kube-lab \
  --set replicaCount=2
kubectl get pods -n kube-lab -l app=web-demo
```

---

## Verification

- [ ] `helm package` creates `web-demo-0.1.0.tgz`
- [ ] `helm install` works from folder **and** from `.tgz`
- [ ] `helm list -n kube-lab` shows release **deployed**
- [ ] Two pods after upgrade with `replicaCount=2`

---

## Break & repair

**`helm package` wrong file name**  
→ Check `name` and `version` in `Chart.yaml`.

**Install from `.tgz` fails: chart incompatible**  
→ Repackage after fixes; bump `version` in `Chart.yaml`.

**Template syntax error**  
→ `helm template test ./web-demo-chart --debug`

---

## Cleanup / revert

```bash
helm uninstall web-demo-from-tgz -n kube-lab 2>/dev/null || true
helm uninstall web-demo -n kube-lab 2>/dev/null || true
rm -f ~/kube-lab/helm/web-demo-*.tgz
```

Optional — remove chart source:

```bash
rm -rf ~/kube-lab/helm/web-demo-chart
```

See also: [00-tooling-setup-cleanup.md](../../00-getting-started/00-tooling-setup-cleanup.md)

---

## You should now be able to…

- Author a minimal chart with **templates** and **values**.
- Run **`helm package`** and explain the `.tgz` **package** format.
- **Install** and **upgrade** from a local package archive.

Previous: [Uninstall](07-uninstall.md) · [Helm chapter index](README.md)
