# ServiceAccount

## Explanation

A **ServiceAccount** is an **identity for Pods** inside the cluster. When a Pod runs, it can use a ServiceAccount to authenticate to the Kubernetes API (if the app needs to list resources, for example).

**Problem it solves:** Workloads that talk to the API server need credentials. ServiceAccount ties “this Pod runs as this identity” so **RBAC** (Role-Based Access Control) can grant least-privilege permissions.

Every namespace has a `default` ServiceAccount; production apps usually get a dedicated ServiceAccount with only the permissions they need.

## Example — add to the cluster

**What's new:** Pods run as a named identity instead of only `default`.

**Before this step:** [Secret](secret.md) — `concept-web` has config + secret env vars.

Create and attach:

```bash
kubectl create serviceaccount concept-sa -n kube-lab
kubectl patch deployment concept-web -n kube-lab -p \
  '{"spec":{"template":{"spec":{"serviceAccountName":"concept-sa"}}}}'
kubectl rollout status deployment/concept-web
```

### Verify

```bash
kubectl get sa concept-sa
POD=$(kubectl get pod -l app=concept-web -o jsonpath='{.items[0].metadata.name}')
kubectl get pod "$POD" -o jsonpath='{.spec.serviceAccountName}{"\n"}'
```

### Break & repair

Remove `serviceAccountName` — Pod falls back to `default`:

```bash
kubectl patch deployment concept-web -p \
  '{"spec":{"template":{"spec":{"serviceAccountName":null}}}}'
```

Next: [ClusterIP Service](clusterip-service.md).

## How it relates to other objects

- **Pod / Deployment** — `serviceAccountName` in the Pod spec selects which ServiceAccount to use.
- **Secret** — ServiceAccounts can mount token Secrets used for API authentication (mechanism varies by Kubernetes version).
- **RBAC (Role, RoleBinding)** — binds permissions to a ServiceAccount; not covered in depth here, but this is the “why” of ServiceAccounts.

See also: [Relationship map](relationship-map.md)

## Hands-on lab

Lab: [../04-hands-on-labs/07-serviceaccount.md](../04-hands-on-labs/07-serviceaccount.md)

## You should now be able to…

- Explain what identity a ServiceAccount gives a Pod inside the cluster.
- Describe why apps that call the Kubernetes API need a ServiceAccount and RBAC.
- Point to where `serviceAccountName` is set in a Pod or Deployment spec.
