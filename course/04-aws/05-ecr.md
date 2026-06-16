# ECR container registry

## Prerequisites

- [Docker basics](../01-docker/01-docker-basics.md) — image vs running container.
- [Docker — build, run, logs](../01-docker/03-build-run-logs-exec.md) — you can build and tag an image locally.
- [AWS account and CLI setup](01-account-cli-setup.md) — profile `lab` works.

---

## Explanation

**ECR** (Elastic Container Registry) is AWS's private store for **container images** — the same kind of images you built with Docker, but hosted in your AWS account instead of Docker Hub.

| Term | Plain meaning |
|---|---|
| **Repository** | Named home for one image line (e.g. `my-app`) |
| **Tag** | Version label on an image (e.g. `1.0`, `latest`) |
| **Registry URI** | Address like `123456789012.dkr.ecr.eu-west-1.amazonaws.com/my-app` |
| **Image scanning** | ECR can scan images for known vulnerabilities (overview) |
| **Lifecycle policy** | Rules to delete old untagged images and save storage cost |

Kubernetes **Deployments** reference an image URI. On EKS, nodes pull from ECR using IAM permissions on the **node role** — no Docker Hub rate limits for private apps.

---

## What you will do

1. Create an ECR repository.
2. Log Docker in to ECR.
3. Tag and push a small lab image.
4. Reference that image in a Kubernetes Deployment manifest (apply when you have a cluster).

---

## Implementation

Set variables for your session (replace account ID and region):

```bash
export AWS_PROFILE=lab
export AWS_REGION=eu-west-1
export ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
export REPO_NAME=course-lab-app
```

### Step 1 — Create repository

```bash
aws ecr create-repository --repository-name "$REPO_NAME" --region "$AWS_REGION"
```

Note the `repositoryUri` in the output.

### Step 2 — Authenticate Docker to ECR

```bash
aws ecr get-login-password --region "$AWS_REGION" | \
  docker login --username AWS --password-stdin \
  "$ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com"
```

Expected: `Login Succeeded`.

### Step 3 — Build, tag, push

Use any small image from the Docker chapter, or:

```bash
docker pull nginx:1.27
docker tag nginx:1.27 "$ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:1.27"
docker push "$ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:1.27"
```

### Step 4 — Verify in ECR

```bash
aws ecr describe-images --repository-name "$REPO_NAME" --region "$AWS_REGION"
```

Or: Console → **ECR** → **Repositories** → your repo → see tag `1.27`.

### Step 5 — Use in Kubernetes (when cluster exists)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: web
          image: ACCOUNT_ID.dkr.ecr.eu-west-1.amazonaws.com/course-lab-app:1.27
          ports:
            - containerPort: 80
```

Replace `ACCOUNT_ID` and region. Apply on local kind/minikube only if the cluster can reach ECR (often use local images locally; use this URI on **EKS** — [EKS overview](06-eks-overview.md)).

---

## Verification checklist

- [ ] Repository exists in console or `describe-repositories`.
- [ ] `docker push` succeeded for at least one tag.
- [ ] `aws ecr describe-images` lists your tag.
- [ ] You can write the full image URI for a Deployment.

---

## Break & repair

| Symptom | Likely cause | Fix |
|---|---|---|
| `no basic auth credentials` on push | Docker not logged in | Re-run `ecr get-login-password` + `docker login` |
| `AccessDenied` on push | IAM user lacks ECR permissions | Add `AmazonEC2ContainerRegistryPowerUser` on lab user (lab account only) |
| `ImagePullBackOff` on EKS | Node role cannot pull | Attach ECR read policy to **node IAM role** (EKS lab) |
| Wrong architecture | Apple Silicon vs amd64 | Build with `--platform linux/amd64` for typical EKS nodes |

**Break on purpose:** Push with a wrong tag name, fix the Deployment image field, redeploy.

---

## You should now be able to…

- Create an ECR repo and push a tagged image.
- Log in to ECR from Docker using the AWS CLI.
- Point a Kubernetes Deployment at an ECR image URI.

Previous: [EC2 and load balancers](04-ec2-and-load-balancers.md) · Next: [EKS overview](06-eks-overview.md)
