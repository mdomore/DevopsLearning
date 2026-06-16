# Lab 1 — Pod

## Explanation

A **Pod** is the smallest unit you deploy in Kubernetes. It wraps one or more **containers** that share network and storage.

**Problem this solves:** You need to run an application (here: nginx web server) inside the cluster. A Pod tells Kubernetes *which container image to run* and *how to run it*.

**Why nginx?** It is a small, well-known web server image — good for learning without building your own app yet. You learned containers in [Docker basics](../../01-docker/01-docker-basics.md); a Pod runs a container *inside* the cluster.

**Prerequisites**

- [Lab 0 — Setup](00-setup.md) — namespace `kube-lab` is active.
- Concept: [Pod](../02-core-concepts/pod.md).

**Shell:** local terminal, namespace `kube-lab`.

## Key fields (before you apply)

| Field | What it controls |
|---|---|
| `kind: Pod` | This object is a single Pod (not a Deployment or ReplicaSet). |
| `metadata.name` | Name you use with `kubectl get pod pod-demo`. |
| `metadata.labels` | Tags for selecting/grouping Pods later (used by Services). |
| `spec.containers[].image` | Which container image to pull and run (`nginx:1.27`). |
| `spec.containers[].ports` | Port the container listens on (informational for networking labs). |

## Implementation

Create a Pod running nginx:

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: pod-demo
  labels:
    app: pod-demo
spec:
  containers:
    - name: web
      image: nginx:1.27
      ports:
        - containerPort: 80
EOF
```

Inspect the Pod:

```bash
kubectl get pod pod-demo -o wide
kubectl describe pod pod-demo
kubectl logs pod-demo
```

## Verification

Expected:

- `kubectl get pod pod-demo` shows `STATUS` = `Running`
- `describe` shows Events with successful pull/start
- `logs` shows nginx startup lines (or is empty if nginx logged nothing yet)

## Break & repair

**Pod stays `ImagePullBackOff` or `ErrImagePull`**  
→ Cluster cannot pull the image (typo in name/tag or no network). Check `kubectl describe pod pod-demo` Events. Fix the `image:` line and re-apply.

**Pod `CrashLoopBackOff`**  
→ Container starts then exits. Read `kubectl logs pod-demo` and Events in `describe`.

## You should now be able to…

- Explain what a Pod is in one sentence
- Create a Pod from YAML with `kubectl apply`
- Check status with `get`, details with `describe`, output with `logs`

Previous: [Setup](00-setup.md) · Next: [ReplicaSet](02-replicaset.md)
