# Docker Compose

**Prerequisite:** [Volumes and networking](04-volumes-and-networking.md).

Commands run in your **local terminal** from the project folder that contains `docker-compose.yml`.

This lesson introduces **richer Dockerfiles** (`WORKDIR`, `RUN`, `ENV`, `CMD`, …) and **Compose** — one file to run a multi-container stack with network and build settings together.

---

## Explanation

### What is Docker Compose?

**Docker Compose** describes a **multi-container application** in one YAML file (`docker-compose.yml`).

**Problem it solves:** Lesson 04 started three containers and a network by hand. In real life you might have a web server, API, and database. Compose lets you declare the whole stack once and run:

```bash
docker compose up -d    # start everything
docker compose down     # stop and remove
```

Compose automatically:

- Creates a **user-defined network** for the project 
- Builds images from **Dockerfiles** when you use `build:`
- Applies **ports**, **volumes**, and **environment** variables per service

**CLI:** use `docker compose` (space). Older docs may say `docker-compose` (hyphen) — same file format.

### Top-level keys in `docker-compose.yml`

Compose files are **YAML** — indentation matters (use spaces, not tabs).

```text
docker-compose.yml
├── services:     ← required — what runs (containers)
├── networks:     ← optional — how services talk to each other
└── volumes:     ← optional — named disks shared between restarts
```

| Key | Required? | Purpose |
|---|---|---|
| `services` | **Yes** | Each entry = one logical app component (`api`, `web`, `redis`…) |
| `networks` | No | Custom networks; if omitted, Compose still creates a **default** network for the project |
| `volumes` | No | Named volumes declared here, then mounted under each service |

Each **service** = one container definition (Compose creates a container per service unless you scale — advanced).

### How Compose names things at runtime

When you run `docker compose up` in folder `compose-demo`:

| Concept | Example | Used for |
|---|---|---|
| **Project name** | `compose-demo` (folder name by default) | Prefixes containers, networks, volumes |
| **Service name** | `api`, `web` | DNS hostname **inside** the stack (`http://api:8000`) |
| **Container name** | `compose-demo-api-1` | `docker ps`, `docker logs` — auto-generated |
| **Network name** | `compose-demo_stack` | `stack` in YAML + project prefix |
| **Image name** | `compose-demo-api` | Built images tagged by project + service |

**Rule for labs:** use the **service name** in Compose commands and URLs (`docker compose logs api`, `curl http://api:8000/` from sibling container).

### Common keys **inside** a service

| Key | Purpose |
|---|---|
| `image:` | Use a pre-built image (`nginx:1.27`, `redis:7-alpine`) — no Dockerfile needed |
| `build:` | Build from a `Dockerfile` in a folder (`./api`) — mutually exclusive with `image:` for the same service (build wins if both set; we use only `build` here) |
| `ports:` | **Publish** to your Mac: `"HOST:CONTAINER"` — e.g. `"8080:80"` → `localhost:8080` |
| `expose:` | Document a port for **other services only** — no access from your Mac |
| `environment:` | Environment variables inside the container (`KEY: value` or `KEY=value`) |
| `volumes:` | Mount named volumes (`redis-data:/data`) or host paths |
| `networks:` | Attach this service to one or more networks listed under top-level `networks:` |
| `depends_on:` | Start order hint — see limitations below |
| `healthcheck:` | When a service is “ready” (capstone lesson) |
| `command:` | Override `CMD` from image or Dockerfile |

### `ports` vs `expose` — critical difference

```text
Your Mac (host)                Docker network "stack"
─────────────                  ────────────────────────

localhost:8080 ──ports──► web:80
                                │
                                │  expose / internal
                                ▼
                           api:8000   (NOT on localhost)
```

| | `ports: "8080:80"` | `expose: "8000"` |
|---|---|---|
| Reachable from Mac | **Yes** (`curl localhost:8080`) | **No** |
| Reachable from sibling container | Yes | **Yes** (`http://api:8000`) |
| Use case | Public entry (web, proxy) | Internal API, database |

### `depends_on` — what it does **not** do

```yaml
depends_on:
  - api
```

- Compose starts **`api` before `web`**
- Compose does **not** wait until the API process listens on port 8000
- If `web` needs a **ready** dependency, use `healthcheck` + `condition: service_healthy` (see [capstone](06-capstone-mini-stack.md))

### What happens when you run `docker compose up -d --build`

1. Read `docker-compose.yml`
2. Create network `compose-demo_stack` (if missing)
3. **Build** images for services with `build:` (`api`, `web`)
4. **Create and start** containers in dependency order
5. Attach containers to network `stack`
6. Apply `ports`, `environment`, etc.

`docker compose down` stops containers and removes the project network (not named volumes unless `-v`).

---

## Implementation — Step 0: Project layout

**Why:** Real projects separate each service into its own folder with its own `Dockerfile`.

```bash
mkdir -p ~/docker-lab/compose-demo/api
mkdir -p ~/docker-lab/compose-demo/web
cd ~/docker-lab/compose-demo
```

Final layout:

```text
compose-demo/
├── docker-compose.yml
├── api/
│   ├── Dockerfile
│   └── app.py
└── web/
    ├── Dockerfile
    └── index.html
```

---

## Implementation — Step 1: API Dockerfile

**Why:** The API is a small Python program packaged in a custom image — closer to how real services are built.

**Instructions used :**


| Instruction | Role in this Dockerfile                               |
| ----------- | ----------------------------------------------------- |
| `FROM`      | Base image with Python                                |
| `WORKDIR`   | Default directory inside the image (`/app`)           |
| `COPY`      | Copy `app.py` into the image                          |
| `ENV`       | Default environment variable (overridable in Compose) |
| `EXPOSE`    | Document port 8000                                    |
| `CMD`       | Start the Python app when the container runs          |


Create `app.py`:

```bash
cat <<'EOF' > api/app.py
from http.server import HTTPServer, BaseHTTPRequestHandler
import json
import os

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        body = json.dumps({
            "service": "api",
            "version": os.environ.get("APP_VERSION", "1.0"),
            "message": "Hello from Compose stack",
        }).encode()
        self.send_response(200)
        self.send_header("Content-type", "application/json")
        self.end_headers()
        self.wfile.write(body)

if __name__ == "__main__":
    HTTPServer(("0.0.0.0", 8000), Handler).serve_forever()
EOF
```

Create `api/Dockerfile`:

```bash
cat <<'EOF' > api/Dockerfile
FROM python:3.12-alpine
WORKDIR /app
COPY app.py .
ENV APP_VERSION=1.0
EXPOSE 8000
CMD ["python", "app.py"]
EOF
```

Test the API image alone (optional):

```bash
docker build -t compose-api:test ./api
docker run --rm -p 8000:8000 compose-api:test
# another terminal: curl -s http://localhost:8000/
# Ctrl+C to stop
```

---

## Implementation — Step 2: Web Dockerfile (nginx + static page)

**Why:** The **web** service is what users hit from the browser. It uses nginx like lesson 02, but lives in its own build context.

```bash
cat <<'EOF' > web/index.html
<!DOCTYPE html>
<html>
  <head><title>Compose demo</title></head>
  <body>
    <h1>Web tier (nginx)</h1>
    <p>This page is served by the <strong>web</strong> service.</p>
    <p>The API runs on the same Compose network — test with:</p>
    <pre>docker compose exec web curl -s http://api:8000/</pre>
  </body>
</html>
EOF
```

```bash
cat <<'EOF' > web/Dockerfile
FROM nginx:1.27
RUN apt-get update \
    && apt-get install -y --no-install-recommends curl \
    && rm -rf /var/lib/apt/lists/*
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
EOF
```

**Why `RUN apt-get install curl`?** The official nginx image is minimal — it has **no** `wget` or `curl`. We add `curl` so you can test `http://api:8000` from inside the **web** container (same network, service DNS).

---

## Implementation — Step 3: `docker-compose.yml` (line by line)

**Why:** This file wires both services, the network, and which ports reach your laptop.

Create the file:

```bash
cat <<'EOF' > docker-compose.yml
services:
  api:
    build: ./api
    environment:
      APP_VERSION: "1.0"
    expose:
      - "8000"
    networks:
      - stack

  web:
    build: ./web
    depends_on:
      - api
    ports:
      - "8080:80"
    networks:
      - stack

networks:
  stack:
EOF
```

### Full file — annotated

```yaml
services:                    # (1) All containers in this stack

  api:                       # (2) Service name = DNS hostname "api"
    build: ./api             # (3) docker build from ./api/Dockerfile
    environment:             # (4) Env vars passed into the container
      APP_VERSION: "1.0"     #     read by app.py via os.environ
    expose:                  # (5) Port visible to OTHER services only
      - "8000"               #     not published to your Mac
    networks:                # (6) Join custom network "stack"
      - stack

  web:                       # (7) Second service — public tier
    build: ./web
    depends_on:              # (8) Start api container before web
      - api                  #     (does not wait for API to be "ready")
    ports:                   # (9) Publish to host
      - "8080:80"            #     Mac localhost:8080 → container port 80
    networks:
      - stack                # (10) Same network → can curl http://api:8000

networks:                    # (11) Declare networks used above
  stack:                     # (12) Empty = default driver (bridge)
                             #     Runtime name: compose-demo_stack
```

### Field-by-field explanation

#### Top level: `services:`

List of logical components. Each key under `services` (`api`, `web`) becomes:

- a **service name** for `docker compose` commands
- a **DNS name** on the Compose network

#### Service `api`

| Line | YAML | Meaning |
|---|---|---|
| Name | `api:` | Hostname `api` for other containers. URL: `http://api:8000` |
| Build | `build: ./api` | Context = folder `api/` (contains `Dockerfile`). Image tagged `compose-demo-api` |
| Config | `environment: APP_VERSION: "1.0"` | Injects env var; overrides `ENV` in Dockerfile if different |
| Network port | `expose: - "8000"` | Documents port 8000 for linked services; **no** `localhost:8000` on Mac |
| Network | `networks: - stack` | Connects to network defined at bottom |

**Why no `ports` on `api`?** Same pattern as production: API is internal; only `web` is public.

#### Service `web`

| Line | YAML | Meaning |
|---|---|---|
| Build | `build: ./web` | nginx + static HTML from `web/Dockerfile` |
| Order | `depends_on: - api` | Container `api` created/started before `web` |
| Publish | `ports: - "8080:80"` | Format `"HOST_PORT:CONTAINER_PORT"`. Browser uses `http://localhost:8080` |
| Network | `networks: - stack` | Same network as `api` — required for `http://api:8000` to resolve |

**`ports` string quotes:** `"8080:80"` avoids YAML parsing `8080:80` as a time value in some editors.

#### Top level: `networks:`

```yaml
networks:
  stack:
```

- Declares a user-defined **bridge** network named `stack`
- At runtime Docker names it **`{project}_{network}`** → `compose-demo_stack`
- All services on `stack` get automatic DNS: `api`, `web`

### Traffic flow for this stack

```text
curl localhost:8080/          → web (nginx) → HTML
docker compose exec web \
  curl http://api:8000/       → api (Python) → JSON

curl localhost:8000/          → FAIL (api not published) ✓ expected
```

### Inspect what Compose created

After `docker compose up -d`:

```bash
docker compose ps              # services, ports, state
docker network ls | grep compose-demo
docker network inspect compose-demo_stack   # which containers are attached
docker compose config          # resolved YAML (merged, no secrets redacted)
```

`docker compose config` is useful to verify Compose **interpreted** your file correctly.

---

## Implementation — Step 4: Start and verify the stack

**Why:** One command builds (if needed) and starts every service.

```bash
cd ~/docker-lab/compose-demo
docker compose up -d --build
```

**Flags:**

- `-d` — detached (background)
- `--build` — rebuild images from Dockerfiles (use after code changes)

Watch status:

```bash
docker compose ps
```

**What to expect:** `api` and `web` state **running**. `web` shows `0.0.0.0:8080->80/tcp` under PORTS.

### Verify from your Mac (web tier)

```bash
curl -s http://localhost:8080/ | head -n 5
```

Expected: HTML with “Web tier (nginx)”.

### Verify service-to-service (web → api)

**Why:** Proves Compose DNS — same idea as lesson 04, but declared in YAML.

```bash
docker compose exec web curl -s http://api:8000/
```

Expected: JSON like `{"service": "api", "version": "1.0", ...}`.

### Logs

**Why:** When a service fails or returns the wrong response, logs are the first place to look. Use the **service name** from `docker-compose.yml` (`api`, `web`) — not the container name (`compose-demo-api-1`).

```bash
docker compose ps -a
docker compose logs api
docker compose logs web --tail 20
```

If a service shows **Exited**, read its logs before changing YAML:

```bash
docker compose logs api --tail 30
```

Full checklist: [Debugging containers](00-debugging-containers.md).

---

## Implementation — Step 5: Change API version (mini release)

**Why:** Simulates updating one service in a stack — like lesson 03 but with Compose.

Edit `docker-compose.yml` — under `api` → `environment`, change to `APP_VERSION: "1.1"`, then:

```bash
docker compose up -d --build --force-recreate api
docker compose exec web curl -s http://api:8000/
```

Expected: `"version": "1.1"` in JSON.

---

## Verification

- `docker compose ps` — both services running; port 8080 on `web` only.
- Browser or `curl http://localhost:8080/` — nginx page.
- `docker compose exec web curl -s http://api:8000/` — JSON from API.
- `docker images` — images named like `compose-demo-api`, `compose-demo-web` (project prefix).

---

## Break & repair

**Something is not working — default steps**

```bash
docker compose ps -a
docker compose logs <service> --tail 30
docker compose up <service>          # no -d: watch startup in the terminal
```

Replace `<service>` with `api` or `web`. See [Debugging containers](00-debugging-containers.md).

**Typo in service name (`apii` instead of `api`)**

Change curl URL to `http://apii:8000/` → “bad address” / connection error. Fix hostname back to `api`.

**API stopped but web still up**

```bash
docker compose stop api
docker compose exec web curl -s --max-time 2 http://api:8000/ || echo "connection failed"
docker compose start api
```

**Port 8080 already in use**

Change `ports` to `"8081:80"` and use `http://localhost:8081/`.

---

## Cleanup / revert

Stop containers, remove network, and optionally remove built images:

```bash
cd ~/docker-lab/compose-demo
docker compose down
docker compose down --rmi local
```

- `down` — stop and remove containers + project network  
- `--rmi local` — remove images **built by this compose file** (not pulled bases like `python:3.12-alpine`)

---

## You should now be able to…

- Explain top-level Compose keys: `services`, `networks`, `volumes`.
- Read a full `docker-compose.yml` line by line: `build`, `ports`, `expose`, `environment`, `depends_on`, `networks`.
- Explain the difference between **service name**, **container name**, and **project name**.
- Use `docker compose config` and `docker network inspect` to verify what Compose created.
- Write a Dockerfile with `WORKDIR`, `ENV`, and `CMD` and use it via `build: ./api`.
- Start/stop a stack with `docker compose up -d` and `docker compose down`.
- Debug a Compose stack with `docker compose ps -a` and `docker compose logs <service>`.
- Reach another service by **service name** from inside the stack.

**Docker chapter continues:** [Capstone — mini production stack](06-capstone-mini-stack.md) — reverse proxy, Redis, and a full three-service lab.

Previous: [Volumes and networking](04-volumes-and-networking.md) · Next: [Capstone — mini production stack](06-capstone-mini-stack.md)