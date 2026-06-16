# Phase 0 — Tooling setup (how to install)

Goal: install the tools used later in this course.  
You are **not** learning Docker or Kubernetes concepts here — only preparing your machine.

Pick the guide for your operating system:

| OS | Guide |
|---|---|
| macOS | [00-tooling-setup-macos.md](00-tooling-setup-macos.md) |
| Linux (Ubuntu LTS) | [00-tooling-setup-linux.md](00-tooling-setup-linux.md) |

**Undo / cleanup:** [00-tooling-setup-cleanup.md](00-tooling-setup-cleanup.md) — remove cluster, namespace, AWS lab profile, Docker junk, optional uninstall.

Commands run in your **local terminal** unless noted.

---

## What you are installing (and why)

| Tool | Role in this course |
|---|---|
| **Docker** | Runs containers locally; required by `kind` for a local Kubernetes cluster |
| **kubectl** | Command-line client to talk to a Kubernetes cluster |
| **kind** | Creates a small local Kubernetes cluster inside Docker |
| **Terraform** | Infrastructure as code (used in later AWS/EKS chapters) |
| **AWS CLI** | Talks to AWS from your terminal (used in later chapters) |

**Why Docker before Kubernetes learning?**  
Local Kubernetes labs need a container runtime. Tools like `kind` use Docker to spin up cluster nodes — you install Docker now so Phase 2 labs work, but you learn how containers work in Phase 1.

---

## Phase 0 completion checklist

Mark Phase 0 done when all of these work (on any OS):

- [ ] `docker version` (client + server)
- [ ] `kubectl version --client`
- [ ] `kubectl get nodes` shows a `Ready` node
- [ ] `kind get clusters` lists `kube-lab`
- [ ] `terraform version`
- [ ] `aws --version`
- [ ] `aws sts get-caller-identity` **or** `aws sts get-caller-identity --profile lab` (if you created a lab profile)

Next: [Phase 1 — Docker](../01-docker/) (learn what containers are and how to use them).

---

## You should now be able to…

- Name the five tools installed in Phase 0 and what each is for
- Run the verification commands in this checklist without errors
- Explain why Docker is installed before the Docker *learning* chapter (runtime vs concepts)
