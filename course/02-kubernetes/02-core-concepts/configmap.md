# ConfigMap

## Explanation

A **ConfigMap** stores **non-sensitive** configuration — settings, feature flags, file contents — outside your container image.

**Problem it solves:** Hard-coding config in an image means rebuilding for every small change. ConfigMap lets you change behavior (environment variables or mounted files) by updating a Kubernetes object, then restarting or rolling Pods as needed.

Examples: `LOG_LEVEL=debug`, a `nginx.conf` snippet, or a JSON config file mounted at `/etc/app/config.json`.

## How it relates to other objects

- **Pod / Deployment / StatefulSet** — reference ConfigMaps in `env`, `envFrom`, or `volumeMounts`.
- **Secret** — same usage patterns, but for sensitive data (passwords, tokens).
- **Service / Ingress** — not directly related; ConfigMap feeds app behavior inside Pods.

See also: [Relationship map](relationship-map.md)

## Hands-on lab

Lab: [../04-hands-on-labs/05-configmap.md](../04-hands-on-labs/05-configmap.md)

## You should now be able to…

- Explain why configuration belongs in ConfigMaps instead of only in container images.
- Name two ways a Pod can consume a ConfigMap (environment vs mounted file).
- Distinguish ConfigMap from Secret at a high level.
