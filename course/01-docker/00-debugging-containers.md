# Debugging containers — a repeatable workflow

**When to use this:** Any time a container misbehaves — won’t start, exits immediately, wrong response, or “connection refused”. Return here whenever you are stuck.

**Prerequisite:** You can run containers ([Docker basics](01-docker-basics.md)) and read logs ([Build, run, logs, exec](03-build-run-logs-exec.md)).

Commands run in your **local terminal**.

---

## Explanation

### The skill that matters most

Tools change; **how you debug** stays the same. Professionals do not guess — they **observe**, **read errors**, **narrow the layer** (host → container → app → config), then **fix one thing** and **verify**.

Use this checklist every time:

| Step | Question | What to run |
|---|---|---|
| **1. Observe** | What is actually wrong? | `curl`, browser, error message |
| **2. State** | Is the container running or exited? | `docker ps -a` or `docker compose ps -a` |
| **3. Logs** | What did the process print before it failed? | `docker logs …` or `docker compose logs …` |
| **4. Inspect** | Exit code? Port mapping? | `docker inspect …` |
| **5. Reproduce** | Can I see the error in the foreground? | `docker run …` or `docker compose up <service>` (no `-d`) |
| **6. Fix & verify** | One change, then test again | rebuild/restart, then `curl` / `ps` / logs |

**Rule:** Read the **last few log lines** first. The real error is usually there — not in step 1 of the startup script.

### Single container vs Compose

| Context | List state | Read logs | Shell inside (if running) |
|---|---|---|---|
| **One container** (`docker run --name foo`) | `docker ps -a --filter name=foo` | `docker logs foo` | `docker exec -it foo sh` |
| **Compose stack** | `docker compose ps -a` | `docker compose logs <service>` | `docker compose exec <service> sh` |

**Common mistake:** `docker compose logs capstone-proxy-1` fails. Compose wants the **service name** from `docker-compose.yml` (`proxy`, `api`, `web`) — not the full container name.

---

## Implementation — Step 1: Check container state

**Why:** A container that **Exited** is not serving traffic. Status tells you whether to read crash logs or network logs.

### Single container

```bash
docker ps -a
```

Look at **STATUS**:

- `Up …` — process is running; check ports and app logic.
- `Exited (1)` — process crashed on startup; **read logs immediately**.
- `Exited (0)` — process finished normally (common with `--rm` one-shot commands).

Filter one container:

```bash
docker ps -a --filter name=hello-web
```

### Compose

```bash
docker compose ps -a
```

- `-a` includes **stopped** services (essential when debugging crashes).
- **PORTS** column: `0.0.0.0:8080->80/tcp` means host port 8080 reaches the container. Empty PORTS often means the service is internal only.

---

## Implementation — Step 2: Read logs

**Why:** Applications and entrypoint scripts print errors to **stdout/stderr**. Docker captures that — `docker logs` is your primary tool.

### Single container

```bash
docker logs hello-web
docker logs --tail 20 hello-web
docker logs -f hello-web
```

- `--tail 20` — last 20 lines only (start here when logs are long).
- `-f` — follow live (`Ctrl+C` to stop).

### Compose

```bash
docker compose logs proxy
docker compose logs api --tail 30
docker compose logs
docker compose logs -f proxy
```

- One service: `docker compose logs <service>`
- All services: `docker compose logs` (no service name)
- `--tail` and `-f` work the same as `docker logs`

**What good log reading looks like:** Scroll to the **bottom**. Find lines with `error`, `Error`, `failed`, `emerg`, `panic`, or `exit`.

---

## Implementation — Step 3: Inspect details

**Why:** Logs tell you *what* failed; inspect tells you *how* the container was configured.

Exit code and last error:

```bash
docker inspect hello-web --format='ExitCode={{.State.ExitCode}} Error={{.State.Error}}'
```

Port bindings:

```bash
docker inspect hello-web --format='{{json .NetworkSettings.Ports}}'
```

Compose — same inspect, use the container name from `docker compose ps -a`:

```bash
docker inspect capstone-proxy-1 --format='ExitCode={{.State.ExitCode}}'
```

**Exit codes (rough guide):**

| Code | Often means |
|---|---|
| `0` | Clean exit |
| `1` | Generic app/config error (read logs) |
| `125` | Docker daemon / `docker run` problem |
| `127` | Command not found inside container |
| `137` | Killed (often out of memory) |

---

## Implementation — Step 4: Run in the foreground

**Why:** With `-d` (detached), the terminal hides startup errors. Foreground mode prints them directly.

### Single container

Stop the broken one, then run without `-d`:

```bash
docker rm -f hello-web
docker run --name hello-web -p 8080:80 hello-web:1.0
```

Leave it running to test; `Ctrl+C` stops it.

### Compose

```bash
docker compose up proxy
```

No `-d` — logs stream to your terminal. Fix the issue, `Ctrl+C`, then:

```bash
docker compose up -d proxy
```

---

## Implementation — Step 5: Exec and validate config

**Why:** Sometimes the container starts but behavior is wrong — wrong file, wrong env, bad config syntax.

### Shell inside a **running** container

```bash
docker exec -it hello-web sh
# inside: ls, cat, env, curl ...
exit
```

Compose:

```bash
docker compose exec api sh
docker compose exec web curl -s http://api:8000/
```

**Note:** `docker exec` fails if the container is **stopped**. Fix startup first (logs), then exec.

### Validate before restart (example: nginx)

Config files baked into the image must be **rebuilt** after you edit them on disk:

```bash
docker compose build proxy
docker compose run --rm --no-deps proxy nginx -t
docker compose up -d proxy
```

`nginx -t` = “test config syntax” — catches typos before nginx crashes.

---

## Worked example — proxy exits with code 1

**Symptom:**

```bash
docker compose ps -a
# proxy   Exited (1)
curl http://localhost:8080/
# (empty / connection refused)
```

**Step 2 — logs:**

```bash
docker compose logs proxy
```

Example output:

```text
nginx: [emerg] unexpected "}" in /etc/nginx/nginx.conf:11
```

**Interpret:** Line 11 of `nginx.conf` has a syntax error — often a typo on the line **above** (nginx reports the closing `}` when the previous line never ended).

**Fix:** Open `proxy/nginx.conf`. Wrong:

```nginx
proxy_pass http://api:8000/\;
```

Correct:

```nginx
proxy_pass http://api:8000/;
```

**Verify:**

```bash
docker compose build proxy
docker compose run --rm --no-deps proxy nginx -t
docker compose up -d proxy
docker compose ps
curl -s http://localhost:8080/
```

---

## Quick reference — “symptom → try this”

| Symptom | First commands |
|---|---|
| Container not in `docker ps` (only in `ps -a`) | `docker logs <name>` — crashed at startup |
| `curl: connection refused` on mapped port | `docker ps` — is container Up? PORTS show `8080->80`? |
| Compose service missing from `ps` | `docker compose ps -a` then `docker compose logs <service>` |
| `no such service: capstone-proxy` | Use service name: `docker compose logs proxy` |
| Changed Dockerfile or copied config, no effect | `docker compose up -d --build` or `docker build …` then recreate |
| `exec: "wget": executable file not found` | Minimal image — install tool in Dockerfile or use `curl` |
| App works in container, not from Mac | Check `-p` / `ports:` mapping and use correct host port |
| One service OK, another cannot connect | DNS: use **service name** (`api`), not `localhost`, from sibling containers |

---

## Break & repair — practice the workflow

**Scenario A — intentional crash**

```bash
docker run --name debug-demo nginx:1.27 nginx -t -c /no/such/file.conf
docker ps -a --filter name=debug-demo
docker logs debug-demo
docker rm debug-demo
```

→ Logs show the config path error. You never needed to guess.

**Scenario B — wrong Compose service name for logs**

```bash
cd ~/docker-lab/compose-demo
docker compose logs web-demo
```

→ `no such service`. Fix: `docker compose logs web` (name under `services:` in YAML).

---

## You should now be able to…

- Follow the **observe → state → logs → inspect → reproduce → fix → verify** loop without skipping steps.
- Use `docker ps -a` / `docker compose ps -a` to spot exited containers.
- Read logs with `--tail` and know when to use `-f`.
- Use **service names** (Compose) vs **container names** (`docker logs`) correctly.
- Run a service in the **foreground** to see startup errors.
- Know that **Dockerfile / config changes require a rebuild**, not just restart.

**Used in:** [Build, run, logs, exec](03-build-run-logs-exec.md) · [Docker Compose](05-docker-compose.md) · [Capstone](06-capstone-mini-stack.md)

Previous: [Docker chapter](README.md) · Next: [Docker basics](01-docker-basics.md)
