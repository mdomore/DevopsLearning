# Install and release

**Prerequisite:** [Repositories](04-repositories.md), namespace `kube-lab`.

---

## Explanation

### What a release is

When you run **`helm install`**, Helm:

1. Loads a **chart** (from repo or local path)
2. Merges **values** (defaults + your `--set` / `-f`)
3. Renders **templates** to Kubernetes manifests
4. Creates resources in the cluster
5. Stores a **release** record (name + revision history)

A **release** = one named installation of a chart in a **namespace**.  
Same chart, two names → two releases (e.g. `web-dev` and `web-prod`).

```text
helm install RELEASE_NAME CHART [flags]
         │              │
         │              └── bitnami/nginx  OR  ./hello-chart
         └── unique per namespace (e.g. lab-nginx)
```

### Install vs template

| Command | Cluster changed? | Release created? |
|---|---|---|
| `helm template` | No | No |
| `helm install` | Yes | Yes (revision 1) |

---

## Implementation — Step 1: Install from a repository

Use a small footprint for the lab:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami 2>/dev/null || true
helm repo update

helm install lab-nginx bitnami/nginx \
  -n kube-lab \
  --set replicaCount=1 \
  --set service.type=ClusterIP \
  --set resources.requests.memory=64Mi
```

**Key flags:**

| Flag | Meaning |
|---|---|
| `-n kube-lab` | Target namespace |
| `--set key=val` | Override a **value** |
| `-f myvalues.yaml` | Values file (alternative to many `--set`) |

Wait until resources exist:

```bash
kubectl get pods,svc -n kube-lab -l app.kubernetes.io/instance=lab-nginx
```

---

## Implementation — Step 2: Inspect the release

```bash
helm list -n kube-lab
helm status lab-nginx -n kube-lab
helm get manifest lab-nginx -n kube-lab | head -n 50
helm get values lab-nginx -n kube-lab
```

| Command | Shows |
|---|---|
| `helm list` | Release names, chart version, status |
| `helm status` | Summary + related resources |
| `helm get manifest` | Rendered YAML applied to cluster |
| `helm get values` | Values used for this release (merged) |

---

## Implementation — Step 3: Install from a local chart (optional)

If you still have `~/kube-lab/helm/hello-chart`:

```bash
helm install hello-local ~/kube-lab/helm/hello-chart \
  -n kube-lab \
  --set replicaCount=1 \
  --set service.type=ClusterIP
```

Two releases can coexist in `kube-lab` if names differ (`lab-nginx` and `hello-local`).

---

## Verification

- [ ] `helm list -n kube-lab` shows `lab-nginx` with status **deployed**
- [ ] `kubectl get pods -n kube-lab` shows nginx pods **Running**
- [ ] `helm get manifest lab-nginx -n kube-lab` contains Deployment and Service

Test connectivity inside cluster:

```bash
kubectl run curl-test -n kube-lab --rm -it --restart=Never \
  --image=curlimages/curl:8.5.0 -- \
  curl -s -o /dev/null -w "%{http_code}\n" http://lab-nginx.kube-lab.svc.cluster.local
```

Expected: `200` (Service DNS name may vary — check `kubectl get svc -n kube-lab`).

---

## Break & repair

**`Error: INSTALLATION FAILED: cannot re-use a name`**  
→ Release name already exists. Use another name or `helm uninstall lab-nginx -n kube-lab` first.

**Pods Pending / insufficient memory**  
→ Reduce resources: `--set resources.requests.memory=64Mi,replicaCount=1`

**Wrong namespace**  
→ Always pass `-n kube-lab` or set default: `export HELM_NAMESPACE=kube-lab`

---

## Cleanup / revert

Skip full cleanup here — you will practice **upgrade** and **rollback** next. To remove only this release now:

```bash
helm uninstall lab-nginx -n kube-lab
```

---

## You should now be able to…

- Define **release** and how it differs from **chart**.
- Run **`helm install`** from a repository with `--set`.
- Use **`helm list`**, **`helm status`**, **`helm get manifest`**, and **`helm get values`**.

Next: [Upgrade and rollback](06-upgrade-rollback.md)

Previous: [Repositories](04-repositories.md)
