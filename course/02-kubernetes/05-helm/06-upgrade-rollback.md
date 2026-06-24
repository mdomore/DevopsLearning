# Upgrade and rollback

**Prerequisite:** [Install and release](05-install-release.md) — release `lab-nginx` in namespace `kube-lab`.

If you uninstalled it, reinstall first:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami 2>/dev/null || true
helm repo update
helm install lab-nginx bitnami/nginx -n kube-lab \
  --set replicaCount=1 --set service.type=ClusterIP --set resources.requests.memory=64Mi
```

---

## Explanation

### Upgrade

**`helm upgrade`** applies a new version of a chart or new **values** to an **existing release**. Each successful upgrade creates a new **revision** (1, 2, 3…).

```bash
helm upgrade RELEASE CHART [flags]
```

Same release name — cluster objects are updated in place (like `kubectl apply` on many files, with history).

### Rollback

**`helm rollback`** restores a previous **revision** when an upgrade breaks the app.

```bash
helm rollback RELEASE REVISION
```

Helm re-applies the manifests from that revision. This is release-level rollback (all resources in the chart together), not only a Deployment rollout.

### History

```bash
helm history RELEASE -n kube-lab
```

Shows revision number, status, chart version, and description.

---

## Implementation — Step 1: Upgrade with new values

Change replica count and a label via values:

```bash
helm upgrade lab-nginx bitnami/nginx -n kube-lab \
  --set replicaCount=2 \
  --set service.type=ClusterIP \
  --set resources.requests.memory=64Mi \
  --set commonLabels.course=helm-lab
```

Check revision:

```bash
helm history lab-nginx -n kube-lab
kubectl get pods -n kube-lab -l app.kubernetes.io/instance=lab-nginx
```

Expected: **revision 2**, two pods Running.

---

## Implementation — Step 2: Upgrade with a values file

**Why:** Production teams store overrides in Git instead of long `--set` chains.

```bash
cat <<'EOF' > ~/kube-lab/helm/nginx-lab-values.yaml
replicaCount: 1
service:
  type: ClusterIP
resources:
  requests:
    memory: 64Mi
commonLabels:
  course: helm-lab
  env: lab
EOF

helm upgrade lab-nginx bitnami/nginx -n kube-lab -f ~/kube-lab/helm/nginx-lab-values.yaml
helm history lab-nginx -n kube-lab
```

Revision **3** — back to one replica, plus `env: lab` label.

Verify values stored on the release:

```bash
helm get values lab-nginx -n kube-lab
```

---

## Implementation — Step 3: Rollback

Roll back to revision **2** (two replicas):

```bash
helm rollback lab-nginx 2 -n kube-lab
helm history lab-nginx -n kube-lab
kubectl get pods -n kube-lab -l app.kubernetes.io/instance=lab-nginx
```

Expected: new revision **4** whose content matches revision 2 (rollback creates a **new** revision — it does not delete history).

---

## Implementation — Step 4: Compare revisions

```bash
helm get manifest lab-nginx -n kube-lab --revision 2 | grep "replicas:"
helm get manifest lab-nginx -n kube-lab --revision 3 | grep "replicas:"
```

See how **values** changed rendered manifests between revisions.

---

## Verification

- [ ] `helm history lab-nginx -n kube-lab` shows multiple revisions
- [ ] After upgrade to 2 replicas, two pods Running
- [ ] After rollback to revision 2, pod count matches that revision
- [ ] `helm get values` reflects your last upgrade

---

## Break & repair

**Upgrade hangs / pods CrashLoopBackOff**  
→ `kubectl get pods -n kube-lab` and `kubectl logs …` — fix values, then upgrade again or rollback:

```bash
helm rollback lab-nginx 1 -n kube-lab
```

**`rollback: release has no revision 9`**  
→ `helm history lab-nginx -n kube-lab` — use a revision number that exists.

**Changed chart version by mistake**  
→ Pin chart version on upgrade: `helm upgrade lab-nginx bitnami/nginx --version 15.0.0 -n kube-lab …`

---

## Cleanup / revert

Keep `lab-nginx` for the next lesson. No action required.

---

## You should now be able to…

- Run **`helm upgrade`** with `--set` and `-f` values files.
- Read **`helm history`** and explain **revision** numbers.
- Run **`helm rollback`** to recover from a bad upgrade.

Next: [Uninstall](07-uninstall.md)

Previous: [Install and release](05-install-release.md)
