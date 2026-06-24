# 03— `kubectl` basics (get/describe/apply/logs/exec)

**Before this section:** [Cluster architecture](02-cluster-architecture.md) — you know what a cluster and a Pod are at a high level.

Commands run in your **local terminal**. Your kubectl **context** should point at your lab cluster with namespace `kube-lab` (see [Prerequisites](00-prerequisites.md)).

---

## Explanation

`**kubectl`** (kube-control) is the program you use to talk to the Kubernetes **API** — the front door of the cluster. Every command either reads state or asks for a change.

Think of it like a remote control for the cluster:


| Command            | What it does                                           | When you use it                       |
| ------------------ | ------------------------------------------------------ | ------------------------------------- |
| `kubectl get`      | Lists resources (short summary)                        | “What is running?”                    |
| `kubectl describe` | Detailed view + **Events** (timeline of what happened) | “Why is this Pod failing?”            |
| `kubectl apply -f` | Create or update from a YAML file                      | “Make the cluster match this file”    |
| `kubectl logs`     | Prints container output                                | “What did the app print?”             |
| `kubectl exec`     | Run a command inside a running container               | “Let me check files/processes inside” |


**YAML file:** a text file describing *desired* state (kind, name, image, etc.). `apply` sends that wish to the cluster; controllers and the scheduler make it real.

## Implementation (commands)

Start by checking the lab namespace is active:

```bash
kubectl config view --minify | grep namespace
```

(`grep` searches text output; if you use `rg` — ripgrep — that works too.)

Create a simple test Pod:

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: kubectl-demo
spec:
  containers:
    - name: web
      image: nginx:1.27
      ports:
        - containerPort: 80
EOF
```

Now practice the core commands:

```bash
kubectl get pod kubectl-demo -o wide
kubectl describe pod kubectl-demo
kubectl logs kubectl-demo
kubectl exec -it kubectl-demo -- nginx -v
kubectl exec -it kubectl-demo -- sh -c 'ls /proc/1 && cat /proc/1/cmdline | tr "\0" " "'
```

**Note on `exec`:** The official **nginx** image is minimal — it includes `sh` but often **not** `ps`. That is normal (same idea as Docker lesson: small images, fewer tools). Use `nginx -v` or read `/proc/1/cmdline` to see the main process instead of `ps aux`.

Quick YAML apply/update:

```bash
kubectl get pod kubectl-demo -o yaml | grep -n "image:\|command:\|args:"
```

## Verification

You should be able to:

- See the Pod in `kubectl get pod`
- Read events in `kubectl describe pod`
- Execute a command in the container via `kubectl exec`
- Get logs successfully with `kubectl logs`

## Break & repair

**Break idea 1: Wrong image tag**

Replace image with a non-existing tag and apply:

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: kubectl-demo
spec:
  containers:
    - name: web
      image: nginx:does-not-exist
      ports:
        - containerPort: 80
EOF
```

Verify failure:

```bash
kubectl get pod kubectl-demo
kubectl describe pod kubectl-demo
kubectl logs kubectl-demo
kubectl get events --sort-by=.lastTimestamp | tail -n 20
```

Repair:

```bash
kubectl delete pod kubectl-demo
```

Recreate correctly:

```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: kubectl-demo
spec:
  containers:
    - name: web
      image: nginx:1.27
      ports:
        - containerPort: 80
EOF
```

**Break idea 2: `kubectl exec` into a terminating container**

Create a Pod that exits quickly:

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: kubectl-exec-dead
spec:
  restartPolicy: Never
  containers:
    - name: box
      image: busybox:1.36
      command: ["sh", "-c", "echo hi && exit 0"]
EOF
```

Then try `exec`:

```bash
kubectl exec -it kubectl-exec-dead -- sh
```

Repair by deleting:

```bash
kubectl delete pod kubectl-exec-dead
```

---

## You should now be able to…

- Run `get`, `describe`, `logs`, and `exec` and say when each is useful
- Create a Pod with `kubectl apply -f` from inline YAML
- Diagnose a bad image using `describe` and Events

Previous: [Cluster architecture](02-cluster-architecture.md) · Next: [YAML manifests](04-yaml-manifests.md)