# Capstone — Mini production stack

**Prerequisite:** [Docker Compose](05-docker-compose.md).

This is the **Docker chapter finale** — a small three-tier stack that mirrors patterns you will see in production and in Kubernetes later:

- **Edge / frontend** — only public entrypoint (nginx reverse proxy)
- **API** — internal application server (custom Dockerfile)
- **Data store** — Redis with a **named volume** (data survives restarts)

Commands run in your **local terminal**.

---

## Explanation

### What “production-like” means here

Real deployments usually separate:


| Tier                 | Role                                  | Exposed to internet?           |
| -------------------- | ------------------------------------- | ------------------------------ |
| **Reverse proxy**    | Routes `/` to web, `/api/` to backend | **Yes** (one port)             |
| **API**              | Business logic                        | **No** (internal network only) |
| **Database / cache** | Persistent or fast storage            | **No**                         |


**Problem this solves:** You practice **defense in depth** and **service discovery** — users never connect directly to Redis or the API; they only hit nginx on port 8080.

### What you will build

```text
Browser / curl :8080
        │
        ▼
   [ proxy ]  nginx — publishes 8080:80
        │
        ├── /      → static HTML (web)
        └── /api/  → http://api:8000/ (internal)
                           │
                           ▼
                      [ api ]  Python — counts visits in Redis
                           │
                           ▼
                      [ redis ]  — named volume redis-data
```

Compose file declares all three services, one network, one volume, and environment variables.

---

## Implementation — Step 0: Project layout

```bash
mkdir -p ~/docker-lab/capstone/{proxy,api,web}
cd ~/docker-lab/capstone
```

---

## Implementation — Step 1: Redis (data tier)

**Why:** API needs shared state. Redis is lightweight and common for caches and counters.

No custom Dockerfile — use the official image and a **named volume** for persistence (lesson 04).

We define the volume in Compose (Step 5); Redis stores data in `/data` by default for AOF/RDB — for this lab we use Redis in-memory with optional persistence via volume mount at `/data`.

---

## Implementation — Step 2: API (application tier)

**Why:** Custom image with `WORKDIR`, `COPY`, `RUN` (install redis client library), `ENV`, `CMD`.

Create `api/app.py`:

```bash
cat <<'EOF' > api/app.py
import json
import os
import redis
from http.server import HTTPServer, BaseHTTPRequestHandler

REDIS_HOST = os.environ.get("REDIS_HOST", "redis")
REDIS_PORT = int(os.environ.get("REDIS_PORT", "6379"))

def get_redis():
    return redis.Redis(host=REDIS_HOST, port=REDIS_PORT, decode_responses=True)

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        try:
            r = get_redis()
            visits = r.incr("visits")
            body = json.dumps({
                "service": "api",
                "visits": visits,
                "redis": f"{REDIS_HOST}:{REDIS_PORT}",
            })
            code = 200
        except redis.RedisError as e:
            body = json.dumps({"error": "redis unavailable", "detail": str(e)})
            code = 503
        self.send_response(code)
        self.send_header("Content-type", "application/json")
        self.end_headers()
        self.wfile.write(body.encode())

if __name__ == "__main__":
    HTTPServer(("0.0.0.0", 8000), Handler).serve_forever()
EOF
```

Create `api/Dockerfile`:

```bash
cat <<'EOF' > api/Dockerfile
FROM python:3.12-alpine
WORKDIR /app
RUN pip install --no-cache-dir redis==5.0.1
COPY app.py .
ENV REDIS_HOST=redis
ENV REDIS_PORT=6379
EXPOSE 8000
CMD ["python", "app.py"]
EOF
```

**Key lines:**

- `RUN pip install ...` — runs **at build time** to add the Redis Python client
- `ENV REDIS_HOST=redis` — hostname matches the **Compose service name** `redis`

---

## Implementation — Step 3: Static web (optional HTML tier)

Simple page for `/`:

```bash
cat <<'EOF' > web/index.html
<!DOCTYPE html>
<html>
  <head><title>Capstone stack</title></head>
  <body>
    <h1>Mini production stack</h1>
    <p>Try the API through the proxy:</p>
    <pre>curl http://localhost:8080/api/</pre>
  </body>
</html>
EOF
```

We will serve this via nginx in the **proxy** container (keeps one nginx entrypoint).

---

## Implementation — Step 4: Proxy (edge tier)

**Why:** **Reverse proxy** — one public port, routes traffic by URL path. Same pattern as production ingress / load balancers.

Copy the static page into the proxy build folder (Docker can only copy files **inside** the build context):

```bash
cp web/index.html proxy/index.html
```

Create `proxy/nginx.conf`:

```bash
cat <<'EOF' > proxy/nginx.conf
events {}
http {
  server {
    listen 80;
    location / {
      root /usr/share/nginx/html;
      index index.html;
    }
    location /api/ {
      proxy_pass http://api:8000/;
    }
  }
}
EOF
```

**What `location /api/` does:** Request to `http://localhost:8080/api/` → nginx forwards to `http://api:8000/` on the internal Compose network.

Create `proxy/Dockerfile`:

```bash
cat <<'EOF' > proxy/Dockerfile
FROM nginx:1.27
COPY nginx.conf /etc/nginx/nginx.conf
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
EOF
```

---

## Implementation — Step 5: `docker-compose.yml`

**Prerequisite reading:** [Docker Compose — full `docker-compose.yml` explanation](05-docker-compose.md) (services, networks, volumes, `ports` vs `expose`, naming).

This capstone adds `**volumes`**, `**healthcheck**`, and `**depends_on` with `condition: service_healthy**` on top of the same patterns.

```bash
cat <<'EOF' > docker-compose.yml
services:
  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data
    networks:
      - internal
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5

  api:
    build: ./api
    depends_on:
      redis:
        condition: service_healthy
    environment:
      REDIS_HOST: redis
      REDIS_PORT: "6379"
    networks:
      - internal

  proxy:
    build: ./proxy
    depends_on:
      - api
    ports:
      - "8080:80"
    networks:
      - internal

volumes:
  redis-data:

networks:
  internal:
EOF
```

### Line-by-line highlights


| Part                                            | Meaning                                                                |
| ----------------------------------------------- | ---------------------------------------------------------------------- |
| `redis` + `redis-data` volume                   | Data tier; volume survives `docker compose down` (unless `-v`)         |
| `healthcheck` on redis                          | Compose can wait until Redis answers `PING`                            |
| `depends_on: redis: condition: service_healthy` | Start API only after Redis is healthy (stronger than lesson 05)        |
| `api` — no `ports`                              | API **not** reachable from Mac directly — only via proxy               |
| `proxy` — `8080:80`                             | **Only** public entrypoint                                             |
| `networks: internal`                            | All services on one private network; DNS names `redis`, `api`, `proxy` |


---

## Implementation — Step 6: Run the stack

```bash
cd ~/docker-lab/capstone
docker compose up -d --build
docker compose ps
```

**What to expect:** `redis` healthy, `api` running, `proxy` publishing 8080.

### Test the full path

Static page:

```bash
curl -s http://localhost:8080/ | head -n 5
```

API **through proxy** (production path):

```bash
curl -s http://localhost:8080/api/
curl -s http://localhost:8080/api/
```

Expected: `"visits": 1` then `"visits": 2` — Redis counter increments.

Direct API from your Mac should **fail** (not published):

```bash
curl -s --max-time 2 http://localhost:8000/ || echo "expected failure — API is internal"
```

**Why this test exists**

In `docker-compose.yml`, only **proxy** has `ports:`:

```yaml
proxy:
  ports:
    - "8080:80"    # Mac port 8080 → container port 80  ✓ public

api:
  # no ports: section                              ✓ internal only
```


| From where                           | URL                          | Works?                                                           |
| ------------------------------------ | ---------------------------- | ---------------------------------------------------------------- |
| Your Mac                             | `http://localhost:8080/api/` | Yes — hits **proxy**, which forwards to `api:8000` inside Docker |
| Your Mac                             | `http://localhost:8000/`     | **No** — nothing on your Mac listens on 8000                     |
| Inside Docker (e.g. proxy container) | `http://api:8000/`           | Yes — private Compose network                                    |


The API **does** run on port 8000 — but **inside** the stack. Docker never maps that port to your Mac, on purpose (same pattern as production: only the edge is public).

**Reading the command**

- `--max-time 2` — stop after 2 seconds (connection refused fails fast).
- `|| echo "expected failure…"` — if `curl` fails, print that message. **Failure here is success** for this lab: it proves the API is not exposed directly.

Check with `docker compose ps`: **proxy** shows `0.0.0.0:8080->80/tcp`; **api** shows `8000/tcp` with **no** host mapping (no `0.0.0.0:8000->…`).

### Prove Redis persistence

```bash
docker compose restart api proxy
curl -s http://localhost:8080/api/
```

Visit count should **continue** from previous value (Redis + volume).

---

## Verification

- [ ] Only port **8080** is public (`docker compose ps`)
- [ ] `curl http://localhost:8080/api/` returns JSON with increasing `visits`
- [ ] `curl http://localhost:8000/` fails from Mac (API internal)
- [ ] After restart, visit counter does not reset to zero

---

## Break & repair

**Any service down or curl fails — start here**

```bash
docker compose ps -a
docker compose logs proxy --tail 30
docker compose logs api --tail 30
docker compose logs redis --tail 30
```

Follow [Debugging containers](00-debugging-containers.md): state → logs → inspect → fix → verify.

**Proxy `Exited (1)` — bad nginx config**

```bash
docker compose logs proxy
```

Look for `nginx: [emerg] … in /etc/nginx/nginx.conf`. Common typo: `proxy_pass http://api:8000/\;` (backslash before `;`) — must be `proxy_pass http://api:8000/;`. After editing `proxy/nginx.conf`:

```bash
docker compose build proxy
docker compose run --rm --no-deps proxy nginx -t
docker compose up -d proxy
```

**API returns `"redis unavailable"`**

```bash
docker compose ps
docker compose logs redis
docker compose logs api
```

→ API started before Redis was ready. Fix: ensure healthcheck + `service_healthy` in compose; `docker compose up -d --force-recreate`.

**502 / empty from `/api/`**

→ Check `proxy/nginx.conf` — `proxy_pass` must be `http://api:8000/` (service name `api`).

**Port 8080 busy**

→ Change to `"8081:80"` under `proxy.ports`.

---

## Cleanup / revert

```bash
cd ~/docker-lab/capstone
docker compose down
```

Remove volumes too (resets Redis data):

```bash
docker compose down -v
docker compose down --rmi local
```

---

## Docker chapter complete — what you achieved

You can now:

- Build images with real Dockerfiles (`RUN`, `ENV`, `WORKDIR`, `CMD`)
- Run, update, log, and exec containers
- Persist data with volumes and connect services on custom networks
- Describe a multi-service stack in Compose
- Run a **mini production pattern**: public proxy → internal API → Redis with persistence

**Next chapter:** [Kubernetes foundations](../02-kubernetes/01-foundations/00-prerequisites.md) — orchestration at cluster scale.

---

## You should now be able to…

- Draw the three tiers (proxy, api, redis) and explain which ports are public.
- Configure nginx `proxy_pass` to an internal service name.
- Use Compose `healthcheck` and `depends_on: condition: service_healthy`.
- Debug a failed service with `docker compose ps -a`, `docker compose logs <service>`, and foreground `docker compose up <service>`.
- Use a named volume for Redis and verify data survives container restarts.

Previous: [Docker Compose](05-docker-compose.md) · Next: [Kubernetes — prerequisites](../02-kubernetes/01-foundations/00-prerequisites.md)