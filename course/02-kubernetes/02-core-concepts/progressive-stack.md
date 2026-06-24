# Progressive stack — build the cluster step by step

Read concept pages **in this order**. Each page adds **one new feature** to namespace **`kube-lab`**.

**Shell:** local terminal · `kubectl config set-context --current --namespace=kube-lab`

| Step | Concept | What you add to the cluster | Object name(s) |
|---|---|---|---|
| 0 | [Namespace](namespace.md) | Lab sandbox | `kube-lab` |
| 1 | [Pod](pod.md) | First workload — one container | `concept-pod` |
| 2 | [ReplicaSet](replicaset.md) | Always 2 identical Pods | `concept-rs` |
| 3 | [Deployment](deployment.md) | Managed app + rolling updates | `concept-web` |
| 4 | [ConfigMap](configmap.md) | Non-secret config | `concept-config` |
| 5 | [Secret](secret.md) | Sensitive value | `concept-secret` |
| 6 | [ServiceAccount](serviceaccount.md) | Pod identity | `concept-sa` |
| 7 | [ClusterIP Service](clusterip-service.md) | Stable internal IP/DNS | `concept-web` (Service) |
| 8 | [NodePort Service](nodeport-service.md) | Reach app from your Mac | same Service, type NodePort |
| 9 | [StorageClass](storageclass.md) | *(inspect)* cluster storage policy | default SC |
| 10 | [PVC](pvc.md) | Request 1 GiB disk | `concept-pvc` |
| 11 | [PV](pv.md) | *(observe)* bound volume | auto-created PV |
| 12 | [StatefulSet](statefulset.md) | Stable Pod names + storage | `concept-sts` |
| 13 | [Ingress](ingress.md) | HTTP route from outside | `concept-ing` |

After each step, run the **Verify** commands on that page before continuing.

**Cleanup one object:**

```bash
kubectl delete pod concept-pod --ignore-not-found
kubectl delete replicaset concept-rs --ignore-not-found
kubectl delete deployment concept-web --ignore-not-found
kubectl delete statefulset concept-sts --ignore-not-found
kubectl delete svc concept-web --ignore-not-found
kubectl delete configmap concept-config --ignore-not-found
kubectl delete secret concept-secret --ignore-not-found
kubectl delete serviceaccount concept-sa --ignore-not-found
kubectl delete pvc concept-pvc --ignore-not-found
kubectl delete ingress concept-ing --ignore-not-found
```

Full lab reset: [Lab 12 — Cleanup](../04-hands-on-labs/12-cleanup.md)

See also: [Relationship map](relationship-map.md)
