# Docker Compose

**Prerequisite:** [Volumes and networking](04-volumes-and-networking.md).

Commands run in your **local terminal** from the project folder that contains `docker-compose.yml`.

---

## Explanation

**Docker Compose** defines **multi-container** apps in one YAML file (`docker-compose.yml`). One command starts or stops the whole stack — web app, database, cache — with networks and volumes wired correctly.

**Problem this solves:** Running three containers by hand (`docker run` × 3, custom networks, env vars) is error-prone. Compose declares “these services belong together” and gives you reproducible dev environments.

This lesson also introduces **richer Dockerfiles** (`RUN`, `WORKDIR`, `CMD`, …) that we skipped in [Dockerfile basics](02-dockerfile-basics.md) to keep the first example small.

Common keys:

| Key | Role |
|---|---|
| `services` | Named containers to run |
| `image` / `build` | Use an existing image or build from Dockerfile |
| `ports` | Publish ports (`"8080:80"`) |
| `volumes` | Named or bind mounts |
| `environment` | Env vars inside the container |
| `depends_on` | Start order hint (does not wait for app readiness) |

Modern CLI: `docker compose` (space). Older installs may use `docker-compose` (hyphen) — same file format.

---

## Implementation

Create a two-service stack: **web** (nginx) + **api** (Python HTTP server). The web container will curl the api by service name.

```bash
mkdir -p ~/docker-lab/compose-demo
cd ~/docker-lab/compose-demo
```

```bash
cat <<'EOF' > docker-compose.yml
services:
  api:
    image: python:3.12-alpine
    command: python -m http.server 8000
    networks:
      - stack

  web:
    image: curlimages/curl:8.5.0
    depends_on:
      - api
    command: sh -c "sleep 2 && curl -s http://api:8000/ && sleep 3600"
    networks:
      - stack

networks:
  stack:
EOF
```

Start the stack in the background:

```bash
docker compose up -d
```

Watch status:

```bash
docker compose ps
```

Confirm the web service reached the api (logs should show HTML directory listing from Python’s server):

```bash
docker compose logs web
```

Optional — run a one-off command in a service:

```bash
docker compose exec web curl -s http://api:8000/ | head -n 3
```

Tear down containers and the Compose network:

```bash
docker compose down
```

---

## Verification

- `docker compose ps` shows `api` and `web` with `running` (or `web` exited 0 if you shorten sleep — for this lab `web` stays up due to `sleep 3600`).
- `docker compose logs web` includes output from `curl http://api:8000/` — proof of **service-to-service DNS** on the Compose network.

---

## Break & repair

**Break:** Change the hostname in `web`’s curl command to a typo (`http://apii:8000/`):

```yaml
command: sh -c "sleep 2 && curl -s http://apii:8000/ && sleep 3600"
```

Apply:

```bash
docker compose up -d --force-recreate web
docker compose logs web
```

**Observe:** `Could not resolve host` or connection error.

**Repair:** Fix the name back to `api` and recreate:

```bash
# Edit docker-compose.yml: apii -> api
docker compose up -d --force-recreate web
docker compose logs web
```

**Break:** Stop only the api while web keeps running:

```bash
docker compose stop api
docker compose exec web curl -s --max-time 2 http://api:8000/ || echo "connection failed"
```

**Repair:**

```bash
docker compose start api
docker compose exec web curl -s http://api:8000/ | head -n 1
```

Full cleanup:

```bash
docker compose down
```

---

## You should now be able to…

- Describe the purpose of `services`, `ports`, `volumes`, and `environment` in Compose.
- Start and stop a multi-container stack with `docker compose up -d` and `docker compose down`.
- Verify one service can reach another by **service name** on the Compose network.

Previous: [Volumes and networking](04-volumes-and-networking.md) · Next: [Kubernetes chapter](../02-kubernetes/)
