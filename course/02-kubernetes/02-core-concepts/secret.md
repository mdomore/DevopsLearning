# Secret

## Explanation

A **Secret** stores **sensitive** data: passwords, API tokens, TLS certificates, registry credentials.

**Problem it solves:** Secrets must not live in images, Git repos, or plain ConfigMaps. Kubernetes Secrets keep sensitive values in a dedicated object type (with access control and encoding expectations — though you still treat cluster RBAC and etcd encryption as part of real security).

Pods consume Secrets the same ways as ConfigMaps: environment variables or mounted files (often as files under `/etc/secrets/...`).

## Example — add to the cluster

**What's new:** a sensitive value separate from ConfigMap.

**Before this step:** [ConfigMap](configmap.md) — `concept-config` on `concept-web`.

Create the Secret:

```bash
kubectl create secret generic concept-secret -n kube-lab \
  --from-literal=API_TOKEN=dev-token-not-for-production
```

Add to the Deployment:

```bash
kubectl set env deployment/concept-web API_TOKEN= \
  --from=secret/concept-secret --keys=API_TOKEN
```

### Verify

```bash
kubectl get secret concept-secret
kubectl exec deploy/concept-web -- printenv API_TOKEN
```

(`kubectl get secret -o yaml` shows base64-encoded data — not encryption.)

### Break & repair

Put a password in a ConfigMap by mistake — anyone with `kubectl get configmap -o yaml` sees plain text. Move it to a Secret and remove it from the ConfigMap.

Next: [ServiceAccount](serviceaccount.md).

## How it relates to other objects

- **ConfigMap** — same injection patterns; ConfigMap for non-sensitive, Secret for sensitive.
- **Pod / Deployment / StatefulSet** — reference Secrets in the Pod spec.
- **ServiceAccount** — can mount token Secrets automatically for API access.
- **Ingress** — may reference TLS Secrets for HTTPS certificates.

See also: [Relationship map](relationship-map.md)

## Hands-on lab

Lab: [../04-hands-on-labs/06-secret.md](../04-hands-on-labs/06-secret.md)

## You should now be able to…

- Explain why passwords and tokens should use Secret instead of ConfigMap.
- Describe how a Pod reads a Secret (env var or volume mount).
- State that Secrets still require proper RBAC and cluster security practices.
