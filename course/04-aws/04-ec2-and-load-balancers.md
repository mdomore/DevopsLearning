# EC2 and load balancers

## Prerequisites

- [VPC networking](03-vpc-networking.md) — subnets, security groups, public vs private.
- [Kubernetes — ClusterIP and NodePort](../02-kubernetes/02-core-concepts/clusterip-service.md) — how Services expose pods inside the cluster.

---

## Explanation

**EC2** (Elastic Compute Cloud) is AWS's service for virtual machines (**instances**). You choose:

| Term | Plain meaning |
|---|---|
| **AMI** (Amazon Machine Image) | Template disk image (OS + optional software) |
| **Instance type** | Size: vCPU, RAM, network (e.g. `t3.small`) |
| **Key pair** | SSH public key AWS installs on Linux instances (optional for EKS nodes) |

For Kubernetes, you often do **not** manage individual EC2 instances by hand — **EKS node groups** launch them for you. Still, EC2 concepts explain what a "node" is in the cloud.

**Load balancer** spreads traffic across multiple targets (instances or IP addresses):

| Type | Layer | Typical use |
|---|---|---|
| **ALB** (Application Load Balancer) | L7 (HTTP/HTTPS) | Web apps, Ingress, path/host routing |
| **NLB** (Network Load Balancer) | L4 (TCP/UDP) | Low latency, static IP, non-HTTP |

**Target group** — the pool of backends the load balancer health-checks and forwards to.

**Health check** — periodic probe; unhealthy targets stop receiving traffic (similar idea to Kubernetes **readiness probes**).

**Kubernetes link:** a Service of type `LoadBalancer` on AWS creates an external load balancer (often NLB or CLB/ALB depending on annotations) that points at your pods/nodes.

---

## What you will do

1. Explore instance types and AMIs in the console (no launch required for learning).
2. Understand ALB vs NLB and target groups.
3. Map ALB health checks to Kubernetes readiness probes conceptually.
4. (Optional preview) Launch one tiny instance — **stop/terminate when done** ([Cost control](07-cost-control.md)).

Full EKS + LoadBalancer Service labs: [Kubernetes hands-on labs](../02-kubernetes/04-hands-on-labs/) on a cloud cluster and [EKS overview](06-eks-overview.md).

---

## Implementation

### Step 1 — Browse EC2 (console)

1. AWS Console → **EC2** → **Instances** → **Launch instances** (do not launch yet).
2. Note: **Amazon Linux 2023** AMI, instance type `t3.micro` or `t3.small`, subnet, security group.
3. Cancel — this walkthrough is awareness only unless you choose the optional launch below.

### Step 2 — ALB concepts (console)

1. EC2 → **Load Balancers** → **Create load balancer**.
2. Compare **Application** vs **Network** — read the short descriptions.
3. Open **Target groups** — see protocol, port, health check path (for ALB HTTP checks).

Map to Kubernetes:

| AWS | Kubernetes |
|---|---|
| Target group health check | Readiness probe on Pod |
| Unhealthy target removed | Pod not ready → removed from Service endpoints |
| Listener on port 443 | Ingress or Service port |

### Step 3 — Optional hands-on (single EC2)

Only on a lab account with budget awareness:

```bash
# Example: describe instance types available (no cost)
aws ec2 describe-instance-type-offerings --profile lab --region eu-west-1 \
  --location-type availability-zone \
  --filters Name=instance-type,Values=t3.micro \
  --query 'InstanceTypeOfferings[0].InstanceType' --output text
```

If you launch an instance via console:

- Use a **security group** allowing SSH only from your IP (not `0.0.0.0/0` unless you accept the risk).
- **Stop or terminate** when finished.

### Step 4 — Service type LoadBalancer (preview)

When you deploy on EKS:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 8080
  selector:
    app: web
```

AWS provisions a load balancer; `kubectl get svc` shows `EXTERNAL-IP` or hostname. Coming in depth: [Ingress lab](../02-kubernetes/04-hands-on-labs/11-ingress.md) and [EKS overview](06-eks-overview.md).

---

## Verification checklist

- [ ] You can explain AMI and instance type in one sentence each.
- [ ] You can state when to pick ALB vs NLB (HTTP vs TCP/low-level).
- [ ] You can describe how a target group health check relates to a readiness probe.
- [ ] You know to terminate/stop lab EC2 instances when idle.

---

## Break & repair

| Symptom | Likely cause | Fix |
|---|---|---|
| ALB returns 502/503 | No healthy targets | Check target group health; fix app or probe path |
| Cannot SSH to EC2 | SG blocks port 22 | Allow SSH from your IP only |
| LoadBalancer pending forever | Subnet tags / controller missing | On EKS, ensure AWS Load Balancer Controller (advanced); check subnets |
| High bill | Forgotten EC2 or LB | Terminate instances; delete load balancers ([Cost control](07-cost-control.md)) |

**Break on purpose:** In the target group docs, change a health check path to `/wrong` mentally and predict why targets would go unhealthy.

---

## You should now be able to…

- Describe EC2 instances as Kubernetes nodes in EKS.
- Compare ALB (L7) and NLB (L4).
- Connect AWS health checks to Kubernetes readiness probes.
- Explain what happens when you set `Service` type to `LoadBalancer` on AWS.

Previous: [VPC networking](03-vpc-networking.md) · Next: [ECR container registry](05-ecr.md)
