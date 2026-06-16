# Lab 6 — Secret

## Explanation

Some values must stay **private**: database passwords, API tokens, TLS keys. A **Secret** is Kubernetes’ object for sensitive data. Like ConfigMaps, Secrets can be injected as env vars or files — but they are stored differently and should be handled with care (still not a full encryption solution by default).

**Problem it solves:** Keep credentials out of images and plain YAML in git; mount them into Pods at runtime.

**Why this lab?** Pairs with [Lab 5 — ConfigMap](05-configmap.md): same injection pattern, different sensitivity.

**Prerequisites**

- [Phase 0 tooling](../../00-getting-started/00-tooling-setup.md) — kind cluster `kube-lab`.
- [Lab 0 — Setup](00-setup.md) — namespace `kube-lab`.
- [Lab 5 — ConfigMap](05-configmap.md) — `envFrom` pattern.
- Concept recap: [Secret](../02-core-concepts/secret.md).

**Shell:** local terminal, `kubectl` → kind cluster, namespace `kube-lab`.

## Key fields (before you apply)

**Secret** (imperative create):

| Piece | What it controls |
|---|---|
| `kubectl create secret generic` | Creates an Opaque Secret with literal key=value pairs. |
| Stored as base64 | Values in etcd/API are encoded, not plain text in the object (still treat as sensitive). |

**Pod:**

| Field | What it controls |
|---|---|
| `envFrom.secretRef.name` | Loads all keys from Secret `app-secret` as environment variables. |

Never commit real production secrets to git. This lab uses fake credentials for learning.

## Implementation

Create the Secret:

```bash
kubectl create secret generic app-secret --from-literal=DB_USER=admin --from-literal=DB_PASSWORD='supersecure123'
```

Create a Pod that reads it:

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: secret-demo
spec:
  containers:
    - name: box
      image: busybox:1.36
      command: ["sh", "-c", "echo USER=$DB_USER; sleep 3600"]
      envFrom:
        - secretRef:
            name: app-secret
EOF
```

Inspect (values are encoded in API output):

```bash
kubectl get secret app-secret
kubectl logs secret-demo
kubectl get secret app-secret -o jsonpath='{.data.DB_PASSWORD}' | base64 --decode; echo
```

## Verification

```bash
kubectl logs secret-demo
```

- Output: `USER=admin` (password is not echoed in the command — good practice).

```bash
kubectl get secret app-secret -o jsonpath='{.data.DB_PASSWORD}' | base64 --decode; echo
```

- Decodes to: `supersecure123`.

```bash
kubectl get secret app-secret
```

- `TYPE` is `Opaque`, `DATA` shows **2** keys.

## Break & repair

**Symptom:** Pod `secret-demo` is `CreateContainerConfigError` or env vars are empty.

**Cause:** Secret name mismatch (`app-secrets` vs `app-secret`) or Secret created in wrong namespace.

**Repair:**

```bash
kubectl get secret app-secret
kubectl describe pod secret-demo
```

- Recreate Secret if needed (delete first if duplicate name with wrong keys):

```bash
kubectl delete secret app-secret
kubectl create secret generic app-secret --from-literal=DB_USER=admin --from-literal=DB_PASSWORD='supersecure123'
kubectl delete pod secret-demo
```

Then re-apply the Pod YAML.

## You should now be able to…

- [ ] Explain the difference between ConfigMap and Secret (sensitivity and usage).
- [ ] Create a generic Secret from literals.
- [ ] Inject Secret keys into a Pod with `envFrom.secretRef`.
- [ ] Decode a base64 Secret value for debugging (lab only — avoid logging real secrets).

---

Previous: [ConfigMap](05-configmap.md) · Next: [ServiceAccount](07-serviceaccount.md)
