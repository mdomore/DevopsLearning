# Volumes and networking

**Prerequisite:** [Build, run, logs, exec](03-build-run-logs-exec.md).

Commands run in your **local terminal**.

---

## Explanation

### Why volumes?

A container’s filesystem is **ephemeral** by default: files you write inside the container live in a thin writable layer. When you **remove** the container, that layer is deleted — data is gone.

**Problem this solves:** Databases, uploads, and config must **survive** container restarts and replacements. Developers also want to edit code on the laptop and see changes inside the container without rebuilding the image every time.

### Two ways to mount storage


| Type             | Syntax example                  | Who owns the storage                   | Typical use                                                     |
| ---------------- | ------------------------------- | -------------------------------------- | --------------------------------------------------------------- |
| **Named volume** | `-v vol-name:/data`             | Docker manages a directory on the host | Database data, “keep this across container deletes”             |
| **Bind mount**   | `-v /host/path:/container/path` | You choose the host folder             | Live dev: edit `./src` on laptop, app reads `/app` in container |


**Mount syntax:** `-v SOURCE:DESTINATION`

- **SOURCE** — named volume (`vol-demo-data`) or host path (`/tmp/bind-test`)
- **DESTINATION** — path **inside** the container (`/data`, `/mnt`)

### Why networking?

When you run a container, Docker connects it to a **network** — a private virtual LAN. Each container gets an **IP address** on that network (like a device on a home Wi‑Fi network).

**The situation:** You often run **more than one** container — for example a web app and an API. The web container must send requests to the API container.

**Two ways to point at the other container:**


| Approach          | Example                  | Problem                                                  |
| ----------------- | ------------------------ | -------------------------------------------------------- |
| By **IP address** | `http://172.18.0.3:8000` | IP changes every time you recreate the container         |
| By **name**       | `http://backend:8000`    | Only works if Docker provides **name → IP** lookup (DNS) |


You want a **stable name** (`backend`), not a number that changes after every `docker rm` / `docker run`.

#### Default network vs network you create

When you run `docker run` without `--network`, Docker attaches the container to the built-in `**bridge`** network.

Think of it like this:

- **Default `bridge`:** Containers can talk to the **outside world** (via port mapping `-p`) and to the **internet**, but Docker does **not** give you simple **name-based** calls between containers (e.g. `curl http://backend:8000` from another container usually **fails**).
- **User-defined network** (you create with `docker network create`): Containers you attach to the **same** network can reach each other **by the name you chose** with `--name`. Docker runs a small **DNS** service on that network — the name `backend` automatically resolves to the correct IP.

```
  Your Mac                    Docker user-defined network "app-net"
  ─────────                   ─────────────────────────────────────
  localhost:8080 ──p──►  [ web ] ──http://backend:8000──► [ backend ]
  (port map)              container                      container
                          name: web                      name: backend
                                                         ▲
                                                         │
                                    Docker DNS: "backend" → 172.x.x.x
```

**What you will do in Part C:** Create network `app-net`, start a container named `backend`, start another container on the same network, and run `curl http://backend:8000`. The hostname `backend` works because both containers share **one user-defined network**.

**Link to later topics:** Kubernetes does something similar with **Services** and cluster DNS — stable name in front of changing Pods. Docker Compose creates a user-defined network for your stack automatically.

---

## Implementation — Part A: Named volume (data survives container delete)

**Goal:** Write a file in a volume, delete the container, prove the file still exists in a **new** container.

### Step 1 — Run a container with a named volume

**Why:** `-v vol-demo-data:/data` creates (if needed) a Docker volume `vol-demo-data` and mounts it at `/data` inside the container.

**Flags:**

- `--rm` — remove the container automatically when it stops (the **volume** is kept)
- `-d` — background
- `sh -c "..."` — write a timestamped file, then sleep so the container stays up

```bash
docker run --rm -d --name vol-demo \
  -v vol-demo-data:/data \
  alpine:3.20 sh -c "echo hello-$(date +%s) > /data/message.txt && sleep 3600"
```

Read the file:

```bash
docker exec vol-demo cat /data/message.txt
```

Copy the output (e.g. `hello-1739123456`) — you will compare it after recreating the container.

### Step 2 — Remove the container, keep the volume

**Why:** Simulates deploying a new container version — old container gone, data must remain.

```bash
docker stop vol-demo
```

With `--rm`, stopping **deletes** the container record. The named volume `vol-demo-data` is **not** deleted.

Confirm the container is gone but the volume exists:

```bash
docker ps -a --filter name=vol-demo
docker volume ls | grep vol-demo-data
```

### Step 3 — New container, same volume

**Why:** A new container mounting the same volume name sees the **same files**.

```bash
docker run --rm -d --name vol-demo2 \
  -v vol-demo-data:/data \
  alpine:3.20 sleep 3600

docker exec vol-demo2 cat /data/message.txt
```

Expected: **same** `hello-...` text as before.

### Step 4 — Inspect the volume

**Why:** See where Docker stores metadata (not usually the raw path on Mac — Docker Desktop hides it — but `inspect` confirms the volume object exists).

```bash
docker volume inspect vol-demo-data
```

Look for `"Mountpoint"` in the JSON output.

---

## Implementation — Part B: Bind mount (host folder → container)

**Goal:** Show files on your **laptop** appearing inside a container — common for live development.

### Step 1 — Create a file on the host

```bash
mkdir -p /tmp/bind-test
echo "host file content" > /tmp/bind-test/note.txt
```

### Step 2 — Mount the host folder into the container

**Why:** `-v /tmp/bind-test:/mnt` maps host `/tmp/bind-test` to container path `/mnt`. Edits on either side show on the other (same inode tree).

```bash
docker run --rm -v /tmp/bind-test:/mnt alpine:3.20 cat /mnt/note.txt
```

Expected: `host file content`

### Step 3 — See live sync (optional)

**Why:** Proves bind mounts are two views of the same data.

In one terminal, keep a container running:

```bash
docker run --rm -d --name bind-demo -v /tmp/bind-test:/mnt alpine:3.20 sleep 3600
```

On the host, change the file:

```bash
echo "updated from host" > /tmp/bind-test/note.txt
docker exec bind-demo cat /mnt/note.txt
```

Expected: `updated from host` without rebuilding anything.

Cleanup this demo:

```bash
docker stop bind-demo
```

---

## Implementation — Part C: User-defined network (reach by name)

**Goal:** Two containers talk using the name `backend` — like a mini service discovery.

### Step 1 — Create a network

**Why:** User-defined networks get automatic DNS: container name → IP.

```bash
docker network create app-net
docker network ls | grep app-net
```

### Step 2 — Start a “backend” server

**Why:** `python -m http.server 8000` is a tiny HTTP server for testing. `--network app-net` attaches it to our network. `--name backend` is the hostname others will use.

```bash
docker run -d --name backend --network app-net \
  python:3.12-alpine \
  python -m http.server 8000
```

#### Verify the container is attached to `app-net`

**Why run this?** Before testing `curl http://backend:8000`, confirm Docker really connected `backend` to `app-net`. If the network attachment is wrong, name-based DNS will not work in Step 3.

**Easier check (recommended):**

```bash
docker inspect backend --format '{{range $name, $config := .NetworkSettings.Networks}}{{$name}} {{end}}'
```

**What to expect:** one line printing:

```text
app-net
```

That means the container is on network `app-net`. If you see `bridge` only, it was not attached to your custom network.

**See IP address on that network (optional):**

```bash
docker inspect backend --format 'Network: {{range $name, $config := .NetworkSettings.Networks}}{{$name}} IP={{$config.IPAddress}} {{end}}'
```

**What to expect:** something like:

```text
Network: app-net IP=172.18.0.2
```

The exact IP may differ — that is fine. Other containers on `app-net` reach this container at that IP, but you will use the **name** `backend` instead of memorizing the number.

**Full JSON snippet (optional — for curious readers):**

```bash
docker inspect backend --format '{{json .NetworkSettings.Networks}}'
```

**What to expect:** one JSON object with a key `"app-net"` containing fields like `"IPAddress"`, `"Gateway"`, etc. We truncated this in older drafts; the full line is long — the commands above are enough for verification.

**Quick sanity check without inspect:**

```bash
docker ps --filter name=backend
```

Container should be `Up`. The network name does not show in `docker ps`; use `inspect` when you need to confirm attachment.

### Step 3 — Call backend from another container by name

**Why:** No need to know backend’s IP — Docker DNS resolves `backend` on `app-net`.

```bash
docker run --rm --network app-net curlimages/curl:8.5.0 \
  curl -s http://backend:8000/
```

Expected: HTML from Python’s directory listing (starts with `<!DOCTYPE HTML>` or similar).

**Contrast:** On the **default** bridge, name-based DNS between arbitrary containers is not what we rely on — that is why Compose and production setups use explicit networks.

---

## Verification

- **Part A:** Same `message.txt` content after container delete and recreate with the same volume name.
- **Part B:** `cat /mnt/note.txt` shows host file; optional edit on host visible in container.
- **Part C:** `curl http://backend:8000/` works from a client container on `app-net`.

---

## Break & repair

**Forgot to mount — file not found**

```bash
docker run --rm alpine:3.20 cat /tmp/bind-test/note.txt
```

→ Path inside container is empty/default — host path was not mounted.

**Repair:**

```bash
docker run --rm -v /tmp/bind-test:/mnt alpine:3.20 cat /mnt/note.txt
```

**Containers on different networks — name does not resolve**

```bash
docker network create net-a
docker network create net-b
docker run -d --name svc-a --network net-a python:3.12-alpine python -m http.server 8000
docker run --rm --network net-b curlimages/curl:8.5.0 curl -s --max-time 2 http://svc-a:8000/
```

→ Fails — `svc-a` is not on `net-b`.

**Repair:** Put the client on the same network:

```bash
docker run --rm --network net-a curlimages/curl:8.5.0 curl -s http://svc-a:8000/ | head -n 1
```

**Volume not empty after tests**

→ List and remove unused volumes:

```bash
docker volume ls
docker volume rm vol-demo-data
```

(Only remove volumes you created for this lab.)

---

## Cleanup / revert

Remove lab containers, network, and volume:

```bash
docker stop vol-demo2 bind-demo backend 2>/dev/null || true
docker rm -f backend 2>/dev/null || true
docker volume rm vol-demo-data 2>/dev/null || true
docker network rm app-net 2>/dev/null || true
```

Remove bind-mount test files on the host (optional):

```bash
# delete /tmp/bind-test when you no longer need it
```

If break/repair networks `net-a` / `net-b` were created:

```bash
docker rm -f svc-a 2>/dev/null || true
docker network rm net-a net-b 2>/dev/null || true
```

---

## You should now be able to…

- Explain why container filesystem data disappears when the container is removed (unless you use a volume).
- Use a **named volume** so data survives container replacement.
- Use a **bind mount** to share a host folder with a container.
- Create a **user-defined network** and reach another container by its `--name`.
- Explain why multi-container apps use named networks instead of hard-coded IPs.

Previous: [Build, run, logs, exec](03-build-run-logs-exec.md) · Next: [Docker Compose](05-docker-compose.md)