# Docker basics

**Prerequisite:** [Phase 0 — Tooling setup](../00-getting-started/00-tooling-setup.md) (Docker installed and `docker version` works).

Commands in this section run in your **local terminal**.

This section covers the core ideas — **images** vs **containers** — plus everyday commands to list, clean up, and chain operations using container IDs.

---

## Explanation

Docker packages an application and its dependencies into an **image** — a read-only template, like a recipe or a snapshot of a filesystem.

When you **run** an image, Docker creates a **container** — a live, isolated process using that image as its root filesystem. You can start many containers from one image.

**Problem this solves:** “Works on my machine” usually means mismatched libraries or versions. Images capture a consistent runtime so the same artifact runs anywhere Docker is installed.

Key ideas:

| Term | Meaning |
|---|---|
| **Image** | Template on disk — `docker images` |
| **Container** | Running or stopped instance — `docker ps` |
| **Layer** | Images stack read-only layers; containers add a thin writable layer |
| **Tag** | Version label, e.g. `nginx:1.27` (`:latest` is just another tag) |
| **Container ID** | Short unique ID Docker assigns (first column of `docker ps`) |

---

## Implementation — images and containers

Verify Docker is running:

```bash
docker version
```

List images on your machine (may be empty at first):

```bash
docker images
```

Download a public image from **Docker Hub** (Docker’s default image catalog):

```bash
docker pull nginx:1.27
docker images nginx
```

Run a **one-off container** that prints nginx version and exits (`--rm` removes the container when it stops):

```bash
docker run --rm nginx:1.27 nginx -v
```

Start a long-running container in the background (`-d` = detached):

```bash
docker run -d --name web-demo nginx:1.27
docker ps
docker ps -a
```

Stop and remove that named container:

```bash
docker stop web-demo
docker rm web-demo
```

---

## Implementation — IDs, filters, and piping

Docker commands often accept a **container ID** or **name** as the last argument. You can copy an ID from `docker ps`, or let Docker print IDs only and pass them to another command.

### Print IDs only (`-q`)

`-q` means **quiet**: show IDs instead of a full table.

```bash
docker ps -q
docker ps -aq
```

- `docker ps -q` — IDs of **running** containers
- `docker ps -aq` — IDs of **all** containers (running + stopped)

### Stop or remove by ID (substitution)

`$( ... )` runs a command and inserts its output into the outer command:

```bash
docker stop $(docker ps -q)
docker rm $(docker ps -aq)
```

**What this does:** `docker ps -q` lists running IDs → `docker stop` stops each one.

**Warning:** If nothing is running, `docker ps -q` is empty and `docker stop` may print a usage error. That is harmless for learning; stop only named demos when unsure.

### Pipe to `xargs` (same idea, different style)

```bash
docker ps -q | xargs docker stop
docker ps -aq | xargs docker rm
```

`xargs` turns lines of text into arguments for the next command.

### Filter then act

Stop only containers you choose with **filters** (`-f`):

```bash
docker ps -q -f ancestor=nginx:1.27
docker ps -q -f name=web-demo
docker ps -aq -f status=exited
```

Example — remove all **stopped** containers:

```bash
docker rm $(docker ps -aq -f status=exited)
```

### Inspect one field from a container

```bash
docker inspect -f '{{.Id}}' web-demo
docker inspect -f '{{.State.Status}}' web-demo
```

Useful when a script needs a single value instead of full JSON.

---

## Implementation — clean up images and containers

### One container or image you know

```bash
docker stop web-demo
docker rm web-demo
docker rmi nginx:1.27
```

- `docker rm` — remove a **stopped** container (or `docker rm -f` to force-remove a running one)
- `docker rmi` — remove an **image** (fails if a container still uses it — remove the container first)

**Image tags matter for `rmi`:** an image is referenced as `name:tag`. If you built with `docker build -t hello-web:1.0 .`, remove it with `docker rmi hello-web:1.0`. If you run `docker rmi hello-web` alone, Docker assumes tag **`latest`** (`hello-web:latest`) — which may not exist.

### Remove unused containers (safe batch)

Removes **stopped** containers not referenced by a running container:

```bash
docker container prune
```

Docker asks for confirmation — read the prompt before accepting.

### Remove dangling / unused images

```bash
docker image prune
```

Removes untagged dangling images. For all unused images (not used by any container):

```bash
docker image prune -a
```

**Read the prompt** — `-a` can delete more than you expect.

### Nuclear tidy (lab machine only)

Removes stopped containers, unused networks, and dangling images in one step:

```bash
docker system prune
```

Add `-a` to include unused images too. Use only when you are sure nothing important is on this Docker host.

More undo options: [Phase 0 cleanup guide](../00-getting-started/00-tooling-setup-cleanup.md#6--clean-docker-lab-containers-and-images).

---

## Verification

You should see:

- `docker pull nginx:1.27` finishes with “Downloaded” or “up to date”.
- `docker run --rm nginx:1.27 nginx -v` prints an nginx version.
- `docker ps -q` prints one short ID per running container (or nothing if none run).
- After `docker stop` + `docker rm web-demo`, `docker ps -a` no longer lists `web-demo`.

---

## Break & repair

**Image tag does not exist**

```bash
docker run --rm nginx:does-not-exist-999 nginx -v
```

→ Error about unknown manifest. Fix: `docker images` and use a real tag.

**Duplicate container name**

```bash
docker run -d --name clash nginx:1.27
docker run -d --name clash nginx:1.27
```

→ Second command fails. Repair: `docker rm -f clash`

**Cannot remove image — in use**

```bash
docker run -d --name busy nginx:1.27
docker rmi nginx:1.27
```

→ Image is in use. Repair: `docker rm -f busy` then `docker rmi nginx:1.27`

---

## Cleanup / revert

After this section, remove lab objects:

```bash
docker rm -f web-demo clash busy 2>/dev/null || true
docker rmi nginx:1.27
```

Or batch-clean stopped containers and unused images:

```bash
docker container prune
docker image prune
```

---

## You should now be able to…

- Explain the difference between an image and a container in one sentence.
- Pull an image, run a container, list them with `docker images` and `docker ps -a`.
- Stop/remove containers using a **name**, an **ID**, or `$(docker ps -q)`.
- Clean up with `docker container prune` and `docker image prune` and know what each removes.

Previous: [Docker chapter](README.md) · Next: [Dockerfile basics](02-dockerfile-basics.md)
