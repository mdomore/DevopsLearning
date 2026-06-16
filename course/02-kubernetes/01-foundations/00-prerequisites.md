# Prerequisites

**Before this chapter:** Complete [Phase 0 — Tooling setup](../../00-getting-started/00-tooling-setup.md) so `kubectl`, Docker, and a local cluster (for example `kind` with cluster name `kube-lab`) are installed and working.

Commands in this section run in your **local terminal** against your Kubernetes cluster unless noted.

---

## Explanation

This chapter assumes your machine can talk to a Kubernetes cluster using **`kubectl`** — the command-line client for the Kubernetes API.

**Problem this section solves:** Later labs create Pods, Deployments, and Services. Without a working cluster and `kubectl` context, those commands fail immediately. This page confirms the basics before you invest time in architecture and YAML.

You will use a dedicated lab namespace **`kube-lab`** so exercises stay isolated and easy to clean up.

---

## Implementation

Verify the kubectl client is installed:

```bash
kubectl version --client
```

Confirm kubectl knows which cluster it talks to (**context** = cluster + user + default namespace):

```bash
kubectl config current-context
kubectl cluster-info
```

Check that at least one **node** (worker machine in the cluster) is ready:

```bash
kubectl get nodes
```

Create the lab namespace and make it the default for your current context (optional but recommended for this course):

```bash
kubectl create namespace kube-lab
kubectl config set-context --current --namespace=kube-lab
```

Confirm you can create resources in that namespace:

```bash
kubectl auth can-i create pods --namespace=kube-lab
```

Expect `yes` if your user has normal lab permissions.

---

## Verification

You should see:

- `kubectl version --client` prints `Client Version` without “command not found”.
- `kubectl get nodes` shows at least one node with `STATUS` **Ready**.
- `kubectl get ns kube-lab` exists after you create it.
- `kubectl auth can-i create pods --namespace=kube-lab` returns **yes**.

---

## Break & repair

**Break:** Point kubectl at a missing or stopped cluster (wrong context):

```bash
kubectl config get-contexts
# If you have a bogus context, switch to it temporarily, or stop your kind cluster:
# kind stop cluster --name kube-lab   # example; your setup may differ
kubectl get nodes
```

**Observe:** Connection refused, “Unable to connect to the server”, or no nodes Ready.

**Repair:** Start your local cluster again (see [Where to run your cluster](01-where-to-run-cluster.md) and Phase 0 setup), then switch back to the correct context:

```bash
kubectl config use-context kind-kube-lab
kubectl get nodes
```

**Break:** Forget to set namespace and run lab commands in `default`:

```bash
kubectl config set-context --current --namespace=default
kubectl get pods
```

**Repair:** Switch back to the lab namespace:

```bash
kubectl config set-context --current --namespace=kube-lab
```

---

## You should now be able to…

- Run `kubectl version`, `kubectl cluster-info`, and `kubectl config current-context` and explain what each shows.
- Confirm nodes are Ready and you have permission to create Pods in `kube-lab`.
- Create and select the `kube-lab` namespace for hands-on work.

Next: [Where to run your cluster](01-where-to-run-cluster.md)
