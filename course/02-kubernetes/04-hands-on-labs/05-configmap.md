# Lab 5 — ConfigMap

## Explanation

Apps need **configuration** (log level, feature flags, file paths) that should not be baked into the container image. You want to change settings without rebuilding the image.

A **ConfigMap** stores non-sensitive config as key–value pairs or small files. Pods can read ConfigMap data as **environment variables** or **mounted files**.

**Problem it solves:** Separate config from code/image; reuse the same image with different settings per environment.

**Why this lab?** Before [Secrets](06-secret.md) (sensitive data), ConfigMaps show the standard pattern for injecting config into Pods.

**Prerequisites**

- [Phase 0 tooling](../../00-getting-started/00-tooling-setup.md) — kind cluster `kube-lab`.
- [Lab 0 — Setup](00-setup.md) — namespace `kube-lab`.
- [Lab 1 — Pod](01-pod.md) — basic Pod structure.
- Concept recap: [ConfigMap](../02-core-concepts/configmap.md).

**Shell:** local terminal, `kubectl` → kind cluster, namespace `kube-lab`.

## Key fields (before you apply)

**ConfigMap** (created via imperative command here):

| Piece | What it controls |
|---|---|
| `kubectl create configmap ... --from-literal` | Key=value pairs stored in the cluster (`APP_MODE=lab`, `LOG_LEVEL=debug`). |

**Pod env injection:**

| Field | What it controls |
|---|---|
| `envFrom.configMapRef.name` | Loads **all** keys from ConfigMap `app-config` as env vars in the container. |
| `command` | Shell one-liner prints env vars so you can verify injection in logs. |

ConfigMaps are **not encrypted** — do not put passwords here (use Secrets in Lab 6).

## Implementation

Create the ConfigMap:

```bash
kubectl create configmap app-config --from-literal=APP_MODE=lab --from-literal=LOG_LEVEL=debug
```

Create a Pod that reads it:

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo
spec:
  containers:
    - name: box
      image: busybox:1.36
      command: ["sh", "-c", "echo MODE=$APP_MODE LEVEL=$LOG_LEVEL; sleep 3600"]
      envFrom:
        - configMapRef:
            name: app-config
EOF
```

Inspect and verify:

```bash
kubectl get configmap app-config -o yaml
kubectl logs configmap-demo
```

## Verification

```bash
kubectl logs configmap-demo
```

- Output includes: `MODE=lab LEVEL=debug`.

```bash
kubectl get configmap app-config -o yaml
```

- `data` section shows `APP_MODE: lab` and `LOG_LEVEL: debug`.

## Break & repair

**Symptom:** Pod is `Running` but logs show `MODE= LEVEL=` (empty) or the Pod is `CreateContainerConfigError`.

**Cause:** `configMapRef.name` points to a ConfigMap that does not exist, or the ConfigMap was created in another namespace.

**Repair:**

```bash
kubectl get configmap app-config
kubectl describe pod configmap-demo
```

- If missing, recreate the ConfigMap (command above).
- Ensure you are in namespace `kube-lab`: `kubectl config set-context --current --namespace=kube-lab`.
- Delete and recreate the Pod if it was created before the ConfigMap existed:

```bash
kubectl delete pod configmap-demo
```

Then re-apply the Pod YAML block.

## You should now be able to…

- [ ] Explain what a ConfigMap is and what should **not** go in one.
- [ ] Create a ConfigMap from literals with `kubectl create configmap`.
- [ ] Inject all keys into a Pod with `envFrom.configMapRef`.
- [ ] Confirm values via `kubectl logs` and `kubectl get configmap -o yaml`.

---

Previous: [StatefulSet](04-statefulset.md) · Next: [Secret](06-secret.md)
