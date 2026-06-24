# ConfigMap

## Explanation

A **ConfigMap** stores **non-sensitive** configuration — settings, feature flags, file contents — outside your container image.

**Problem it solves:** Hard-coding config in an image means rebuilding for every small change. ConfigMap lets you change behavior (environment variables or mounted files) by updating a Kubernetes object, then restarting or rolling Pods as needed.

Examples: `LOG_LEVEL=debug`, a `nginx.conf` snippet, or a JSON config file mounted at `/etc/app/config.json`.

## Example — add to the cluster

**What's new:** configuration stored outside the image; Deployment reads it via env var.

**Before this step:** [Deployment](deployment.md) — `concept-web` running.

Create the ConfigMap:

```bash
kubectl create configmap concept-config -n kube-lab \
  --from-literal=GREETING="Hello from ConfigMap"
```

Wire it into the Deployment (adds one env var to the existing container):

```bash
kubectl set env deployment/concept-web GREETING= \
  --from=configmap/concept-config --keys=GREETING
```

Or add to `03-deployment.yaml` under the container:

```yaml
        env:
        - name: GREETING
          valueFrom:
            configMapKeyRef:
              name: concept-config
              key: GREETING
```

Then `kubectl apply -f ~/kube-lab/manifests/concepts/03-deployment.yaml`.

### Verify

```bash
kubectl get configmap concept-config -o yaml
kubectl exec deploy/concept-web -- printenv GREETING
```

### Break & repair

Change the value — Pods still show the old env until restarted:

```bash
kubectl patch configmap concept-config -p '{"data":{"GREETING":"Updated"}}'
kubectl exec deploy/concept-web -- printenv GREETING   # still old
kubectl rollout restart deployment/concept-web
kubectl exec deploy/concept-web -- printenv GREETING   # Updated
```

Next: [Secret](secret.md).

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
