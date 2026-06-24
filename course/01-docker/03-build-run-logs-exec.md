# Build, run, logs, exec

**Prerequisite:** [Dockerfile basics](02-dockerfile-basics.md).

Commands run in your **local terminal**. Examples reuse `~/docker-lab/hello-web` from the previous lesson (or recreate it there).

---

## Explanation

Once you have a Dockerfile, most day-to-day Docker work is a small loop:

1. **Change** app files or Dockerfile  
2. **`docker build`** — create a new **image version** (a new tag)  
3. **`docker run`** — start a **container** from that image  
4. **`docker logs`** / **`docker exec`** — see output and debug inside the running container  

**Problem this solves:** You need to start apps in the background, ship a new version safely, and inspect what is running — without guessing what happened inside the container.

| Command | What it does | When you use it |
|---|---|---|
| `docker build -t name:tag .` | Builds an **image** from the Dockerfile | After code or Dockerfile changes |
| `docker run` | Creates and starts a **container** from an image | Deploy / test a version |
| `docker logs` | Reads stdout/stderr from the container | App errors, access logs, startup messages |
| `docker exec` | Runs a command inside a **running** container | Inspect files, run tools, open a shell |

**Image vs container (recap):**  
Updating your app means building a **new image tag** (`hello-web:1.1`), then replacing the **container** that runs the old tag (`hello-web:1.0`). Docker does not “patch” a running container in place — you stop/remove the old one and run a new one.

**Detached (`-d`)** vs **interactive (`-it`)**:

- `-d` — container runs in the background; terminal is free.
- `-it` — interactive terminal (shell inside the container).

**Port mapping (`-p hostPort:containerPort`)** — publishes a container port on your machine (`8080` on Mac → `80` in container).

---

## Implementation — Part A: run, logs, and exec (one version)

**Goal:** Run `hello-web:1.0`, read logs, and exec inside the container.

### Step 1 — Build and run version 1.0

**Why:** Confirm the image from lesson 02 exists and a container is serving traffic.

```bash
cd ~/docker-lab/hello-web
docker build -t hello-web:1.0 .
docker run -d --name hello-web -p 8080:80 hello-web:1.0
```

- `-d` — background  
- `--name hello-web` — fixed name for logs/exec  
- `-p 8080:80` — browser/curl uses port 8080 on your machine  

Check it is running:

```bash
docker ps --filter name=hello-web
curl -s http://localhost:8080/
```

### Step 2 — Read logs

**Why:** Containers print app output to stdout/stderr. `docker logs` is your first debug tool when something fails.

> **Debugging habit:** When anything breaks, always check **state** (`docker ps -a`) then **logs** (`docker logs --tail 20 …`) before changing config. Full checklist: [Debugging containers](00-debugging-containers.md).

```bash
docker logs hello-web
```

Generate traffic and read the last lines only:

```bash
curl -s http://localhost:8080/ > /dev/null
docker logs --tail 5 hello-web
```

`--tail 5` shows only the last 5 log lines (useful when logs are long).

Follow logs live (optional — `Ctrl+C` to stop following):

```bash
docker logs -f hello-web
```

### Step 3 — Exec into the running container

**Why:** Logs show output; **exec** lets you look around — list files, read configs, run commands — without rebuilding the image.

One-off command:

```bash
docker exec hello-web ls -la /usr/share/nginx/html
docker exec hello-web cat /usr/share/nginx/html/index.html
```

Interactive shell (`-it`):

```bash
docker exec -it hello-web sh
```

Inside the shell:

```sh
cat /usr/share/nginx/html/index.html
exit
```

**Note:** `exec` only works while the container is **running**. If you `docker stop` first, exec fails until you `docker start` again.

---

## Implementation — Part B: real-life version update (1.0 → 1.1)

**Goal:** Simulate what you do when releasing a new app version: change files, build a new tag, replace the running container, verify with curl and logs.

This is the normal **dev loop** before Kubernetes or CI/CD automate it for you.

### Step 1 — Version 1.0 is already running

From Part A you should still have:

```bash
docker ps --filter name=hello-web
curl -s http://localhost:8080/
```

You should see `Built from a Dockerfile` (version 1.0 content).

### Step 2 — Change the app (new HTML)

**Why:** A new release means different files in the build context. Here we only change `index.html` — no Dockerfile change needed.

```bash
cd ~/docker-lab/hello-web
cat <<'EOF' > index.html
<!DOCTYPE html>
<html>
  <head><title>Hello Docker</title></head>
  <body>
    <h1>Hello Web v1.1</h1>
    <p>Updated release — same Dockerfile, new content.</p>
  </body>
</html>
EOF
```

### Step 3 — Build image version 1.1 (do not stop yet)

**Why:** `docker build` creates a **new image tag**. The old container still serves 1.0 until you replace it — both images can exist at the same time.

```bash
docker build -t hello-web:1.1 .
docker images hello-web
```

Expected: two rows — `hello-web:1.0` and `hello-web:1.1`.

**Important:** The running container still uses the **old image layer** it was created from. Building 1.1 does **not** change what is already running.

Verify the old content is still live:

```bash
curl -s http://localhost:8080/ | grep h1
```

Still shows v1.0 until you replace the container.

### Step 4 — Replace the container (deploy 1.1)

**Why:** To run the new version on port 8080, stop the old container, remove it, and start a new one from `hello-web:1.1`. Same name and port — users still hit `localhost:8080`.

```bash
docker stop hello-web
docker rm hello-web
docker run -d --name hello-web -p 8080:80 hello-web:1.1
```

Check version in the browser or terminal:

```bash
curl -s http://localhost:8080/
docker logs --tail 3 hello-web
docker exec hello-web cat /usr/share/nginx/html/index.html
```

Expected: `Hello Web v1.1` in the HTML.

### Step 5 — Roll back to 1.0 (optional, same pattern)

**Why:** Rollback in Docker is “run the previous image tag again” — no special command.

```bash
docker stop hello-web
docker rm hello-web
docker run -d --name hello-web -p 8080:80 hello-web:1.0
curl -s http://localhost:8080/ | grep h1
```

Expected: back to `Built from a Dockerfile`.

In production, teams keep several tags (`1.0`, `1.1`, `latest`) in a registry and redeploy the tag they trust.

---

## Verification

- `docker ps` shows `hello-web` with `0.0.0.0:8080->80/tcp`.
- After Part B step 4, `curl http://localhost:8080/` shows **v1.1** heading.
- `docker images hello-web` can show both `1.0` and `1.1` tags.
- `docker logs hello-web` shows nginx lines after requests.
- `docker exec hello-web cat .../index.html` matches what `curl` returns.

---

## Break & repair

Use the [debugging workflow](00-debugging-containers.md) if you are unsure where to start: **observe → state → logs → fix → verify**.

**Wrong host port mapped**

```bash
docker run -d --name hello-web -p 9090:80 hello-web:1.1
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:8080/
```

→ Fails on 8080. Fix: use `http://localhost:9090/` or recreate with `-p 8080:80`.

**Built 1.1 but browser still shows 1.0**

→ Old container still running. Fix:

```bash
docker ps --filter name=hello-web
docker stop hello-web && docker rm hello-web
docker run -d --name hello-web -p 8080:80 hello-web:1.1
```

**Exec into stopped container**

```bash
docker stop hello-web
docker exec hello-web ls
```

→ Error. Fix: `docker start hello-web` then exec again.

**Cannot reuse name `hello-web`**

→ A container (running or stopped) already has that name. Fix: `docker rm hello-web` (after stop) before `docker run --name hello-web` again.

---

## Cleanup / revert

Stop the lab container and remove image tags you no longer need:

```bash
docker stop hello-web
docker rm hello-web
docker rmi hello-web:1.0 hello-web:1.1
```

Use the **full tag** with `rmi` (`hello-web:1.1`, not just `hello-web`). See [Dockerfile basics — cleanup](02-dockerfile-basics.md#cleanup--revert).

Keep `nginx:1.27` if other lessons need it.

---

## You should now be able to…

- Run a container in the background with `-d`, `-p`, and `--name`.
- Read container output with `docker logs` (including `--tail` and `-f`).
- Apply the [debugging workflow](00-debugging-containers.md) when a container exits or misbehaves.
- Use `docker exec` and `docker exec -it` inside a **running** container.
- Build a new image tag after a code change and **replace** the container to deploy it.
- Explain why `docker build` alone does not update a already-running container.

Previous: [Dockerfile basics](02-dockerfile-basics.md) · Next: [Volumes and networking](04-volumes-and-networking.md)
