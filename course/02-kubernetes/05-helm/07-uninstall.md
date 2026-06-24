# Uninstall

**Prerequisite:** [Upgrade and rollback](06-upgrade-rollback.md).

---

## Explanation

### What uninstall does

**`helm uninstall`** removes a **release** and, by default, **all Kubernetes resources** that Helm created for that release (Deployments, Services, etc.).

```bash
helm uninstall RELEASE -n NAMESPACE
```

Helm also removes the release **history** stored in the namespace (Secrets labeled by Helm).

### Uninstall vs rollback

| Action | Release | Resources | History |
|---|---|---|---|
| **rollback** | Kept | Reverted to old revision | New revision added |
| **uninstall** | Removed | Deleted (default) | Removed |

### When to uninstall

- Lab cleanup
- Remove an app completely before reinstalling under a new name
- Free cluster resources

---

## Implementation — Step 1: List releases before delete

```bash
helm list -n kube-lab
kubectl get all -n kube-lab -l app.kubernetes.io/instance=lab-nginx
```

---

## Implementation — Step 2: Uninstall a release

```bash
helm uninstall lab-nginx -n kube-lab
```

Verify release gone:

```bash
helm list -n kube-lab
helm status lab-nginx -n kube-lab
```

Expected: `helm list` empty (or without `lab-nginx`); `helm status` → release not found.

Verify resources gone:

```bash
kubectl get pods,svc,deploy -n kube-lab -l app.kubernetes.io/instance=lab-nginx
```

Expected: `No resources found`.

---

## Implementation — Step 3: Uninstall other releases (if any)

If you installed `hello-local` earlier:

```bash
helm list -n kube-lab
helm uninstall hello-local -n kube-lab
```

---

## Implementation — Step 4: Keep history on uninstall (optional flag)

Some teams audit after delete. Helm can keep release records:

```bash
# Re-install for demo only:
helm install lab-nginx bitnami/nginx -n kube-lab \
  --set replicaCount=1 --set service.type=ClusterIP --set resources.requests.memory=64Mi

helm uninstall lab-nginx -n kube-lab --keep-history
helm list -n kube-lab
helm list -n kube-lab --uninstalled
```

`--keep-history` allows `helm rollback` on an uninstalled release in some workflows; for labs, plain **`helm uninstall`** is enough.

Final cleanup:

```bash
helm uninstall lab-nginx -n kube-lab 2>/dev/null || true
```

---

## Verification

- [ ] `helm list -n kube-lab` shows no unwanted releases
- [ ] Pods and Services from `lab-nginx` are gone
- [ ] Namespace `kube-lab` still exists (only the release was removed)

---

## Break & repair

**Resources left after uninstall**  
→ Chart may have resources with `helm.sh/resource-policy: keep` (uncommon in lab charts). Delete manually:

```bash
kubectl delete deploy,svc -n kube-lab -l app.kubernetes.io/instance=lab-nginx
```

**`helm uninstall` on wrong release**  
→ `helm list -A` — check all namespaces. Re-install from [05-install-release.md](05-install-release.md) if needed.

---

## Cleanup / revert

Reinstall anytime for practice:

```bash
helm repo update
helm install lab-nginx bitnami/nginx -n kube-lab \
  --set replicaCount=1 --set service.type=ClusterIP --set resources.requests.memory=64Mi
```

Full Helm lab reset: uninstall all releases in `kube-lab`, then continue to [Package and custom chart](08-package-custom-chart.md).

---

## You should now be able to…

- Run **`helm uninstall`** and confirm resources are removed.
- Explain the difference between **uninstall** and **rollback**.
- Use **`helm list`** to verify no stray releases remain.

Next: [Package and custom chart](08-package-custom-chart.md)

Previous: [Upgrade and rollback](06-upgrade-rollback.md)
