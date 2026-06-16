# Topic A — Cluster architecture (Control plane vs workers)

**Before this section:** [Prerequisites](00-prerequisites.md) — cluster reachable with `kubectl`.

---

## Explanation (what to understand)

A **Kubernetes cluster** is a set of machines (physical or virtual) that work together to run your applications. You do not log into each machine manually — you send instructions through `kubectl`, and the cluster decides how to carry them out.

In Kubernetes, there are two big zones:

- **Control plane**: decides and records “desired state”.
  - `kube-apiserver`: accepts API requests (through `kubectl`)
  - `etcd`: stores the cluster state database
  - `kube-scheduler`: assigns Pods to nodes
  - `kube-controller-manager`: runs controllers (Deployment controller, etc.)
  - (often) **admission controllers**: validate/mutate requests before persisting them
- **Worker nodes**: run workloads.
  - `kubelet`: agent on each node that ensures Pods are running
  - container runtime: `containerd` / `CRI-O` (implementation detail)
  - `kube-proxy` (or eBPF datapath depending on CNI): enables Service forwarding on nodes

What you can observe from the outside:

- Nodes and their labels/capacity
- Workloads (Pods) running on nodes
- Control-plane components visibility varies by cluster type (kind/minikube often simplify it).

## Implementation (commands)

Create a lab namespace first (safe and reversible):

```bash
kubectl create namespace kube-lab
kubectl config set-context --current --namespace=kube-lab
```

Observe workers:

```bash
kubectl get nodes -o wide
kubectl describe node "$(kubectl get nodes -o name | head -n 1)"
```

Observe Pods and where they run:

```bash
kubectl get pods -o wide
kubectl get pods -A -o wide | head
```

Look for control-plane components (may or may not be present depending on cluster):

```bash
kubectl get pods -A | grep -i "etcd\|kube-apiserver\|scheduler\|controller-manager"
```

Also inspect core system namespaces:

```bash
kubectl get pods -n kube-system -o wide
kubectl get deployments -n kube-system
```

## Verification (what “correct” looks like)

- `kubectl get nodes` returns at least one node with a `STATUS` of `Ready`.
- You can see Pods with a `NODE` column showing where they run.
- You can identify (if visible) some control-plane pods in `-A` listing, or at least understand that your cluster hides them.

## Break & repair (controlled experiments)

**Break idea 1: Make a Pod scheduled “somewhere” then see it land**

1. Create a Pod that sleeps:

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: arch-sleep
spec:
  containers:
    - name: box
      image: busybox:1.36
      command: ["sh", "-c", "sleep 3600"]
EOF
```

2. Verify it ran on a node:

```bash
kubectl get pod arch-sleep -o wide
```

3. Repair: delete it (clean state):

```bash
kubectl delete pod arch-sleep
```

**Break idea 2: Create a Pod that never becomes Ready**

Create a Pod using a wrong command so it terminates quickly:

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: arch-crash
spec:
  containers:
    - name: box
      image: busybox:1.36
      command: ["sh", "-c", "exit 1"]
EOF
```

Observe and diagnose:

```bash
kubectl get pod arch-crash -o wide
kubectl describe pod arch-crash
kubectl logs arch-crash
kubectl get events --sort-by=.lastTimestamp | tail -n 20
```

Repair (delete it):

```bash
kubectl delete pod arch-crash
```

Notes:
- This doesn’t “break architecture” itself, but it gives you a practical control-plane/worker mental model using scheduling + events + logs.

---

## You should now be able to…

- Name the two main zones of a cluster (control plane vs workers) and one component in each
- Run `kubectl get nodes` and `kubectl get pods -o wide` and explain what the `NODE` column means
- Create a Pod and find scheduling/errors using `describe` and Events

Previous: [Where to run your cluster](01-where-to-run-cluster.md) · Next: [`kubectl` basics](03-kubectl-basics.md)
