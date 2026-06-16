# Volumes and networking

**Prerequisite:** [Build, run, logs, exec](03-build-run-logs-exec.md).

Commands run in your **local terminal**.

---

## Explanation

Containers are **ephemeral** by default: when you remove a container, data written inside its filesystem is lost. **Volumes** solve persistence and sharing files between host and container.

Two common patterns:

| Type | What it is | Typical use |
|---|---|---|
| **Named volume** | Docker-managed storage on the host | Database data, “keep this across restarts” |
| **Bind mount** | Maps a host folder into the container | Live code edit during dev (`./src` → `/app`) |

**Networking:** Containers on the default **bridge** network get private IPs. Containers on the **same user-defined network** can reach each other **by container name** (Docker provides DNS).

**Problem this solves:** Apps need durable data and multi-container communication (app → database) without hard-coding changing IP addresses.

---

## Implementation

### Part A — Named volume (data survives container delete)

Run Alpine, write a file into a mounted volume at `/data`:

```bash
docker run --rm -d --name vol-demo \
  -v vol-demo-data:/data \
  alpine:3.20 sh -c "echo hello-$(date +%s) > /data/message.txt && sleep 3600"
```

Read the file from inside the container:

```bash
docker exec vol-demo cat /data/message.txt
```

Stop and **remove** the container (volume remains):

```bash
docker stop vol-demo
```

Start a **new** container using the **same** volume name:

```bash
docker run --rm -d --name vol-demo2 \
  -v vol-demo-data:/data \
  alpine:3.20 sleep 3600
docker exec vol-demo2 cat /data/message.txt
```

Inspect the volume:

```bash
docker volume ls | grep vol-demo-data
docker volume inspect vol-demo-data
```

Cleanup:

```bash
docker stop vol-demo2
docker volume rm vol-demo-data
```

### Part B — User-defined network (reach by name)

Create a network:

```bash
docker network create app-net
```

Start a tiny HTTP server container named `backend`:

```bash
docker run -d --name backend --network app-net \
  python:3.12-alpine \
  python -m http.server 8000
```

Start another container on the same network and curl `backend` by name:

```bash
docker run --rm --network app-net curlimages/curl:8.5.0 \
  curl -s http://backend:8000/ | head -n 3
```

Cleanup:

```bash
docker rm -f backend
docker network rm app-net
```

---

## Verification

- After deleting the first container, the second container still shows the same `message.txt` content — the **named volume** persisted data.
- `curl http://backend:8000/` from the `curl` container succeeds — **DNS on user-defined network** resolves `backend` to the other container.

---

## Break & repair

**Break:** Use a bind mount to a path that does not exist on the host — Docker often creates it as a directory:

```bash
mkdir -p /tmp/bind-test
echo "host file" > /tmp/bind-test/note.txt
docker run --rm alpine:3.20 cat /tmp/bind-test/note.txt
```

**Observe:** File not found — nothing was mounted.

**Repair:** Mount explicitly:

```bash
docker run --rm -v /tmp/bind-test:/mnt alpine:3.20 cat /mnt/note.txt
```

**Break:** Put containers on **different** networks and try to curl by name:

```bash
docker network create net-a
docker network create net-b
docker run -d --name svc-a --network net-a python:3.12-alpine python -m http.server 8000
docker run --rm --network net-b curlimages/curl:8.5.0 curl -s --max-time 2 http://svc-a:8000/
```

**Observe:** Name does not resolve or connection fails — they are isolated.

**Repair:** Attach client to `net-a` or connect both to a shared network:

```bash
docker run --rm --network net-a curlimages/curl:8.5.0 curl -s http://svc-a:8000/ | head -n 1
docker rm -f svc-a
docker network rm net-a net-b
```

---

## You should now be able to…

- Explain named volume vs bind mount and when each is useful.
- Persist data across container replacement using `-v volume-name:/path`.
- Create a user-defined network and reach another container by its `--name`.

Previous: [Build, run, logs, exec](03-build-run-logs-exec.md) · Next: [Docker Compose](05-docker-compose.md)
