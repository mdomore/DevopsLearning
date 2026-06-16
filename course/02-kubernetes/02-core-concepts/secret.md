# Secret

## Explanation

A **Secret** stores **sensitive** data: passwords, API tokens, TLS certificates, registry credentials.

**Problem it solves:** Secrets must not live in images, Git repos, or plain ConfigMaps. Kubernetes Secrets keep sensitive values in a dedicated object type (with access control and encoding expectations — though you still treat cluster RBAC and etcd encryption as part of real security).

Pods consume Secrets the same ways as ConfigMaps: environment variables or mounted files (often as files under `/etc/secrets/...`).

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
