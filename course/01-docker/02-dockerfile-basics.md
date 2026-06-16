# Dockerfile basics

**Prerequisite:** [Docker basics](01-docker-basics.md) and [Phase 0 Docker install](../00-getting-started/00-tooling-setup.md).

Commands run in your **local terminal**. Create a working directory anywhere you like; examples use `~/docker-lab/hello-web`.

---

## Explanation

A **Dockerfile** is a text file of instructions Docker uses to **build** a custom image. Instead of only using pre-made images from Docker Hub, you define your own: base OS, files to copy, commands to run, and how the container starts.

**Problem this solves:** You need a repeatable way to package *your* app (static site, script, binary) with the right files and startup command — same result on every build.

Dockerfiles are read **top to bottom**. This lesson uses only three instructions plus one **inherited** from the base image:

| Instruction | Used in our example? | Role |
|---|---|---|
| `FROM` | Yes | Starting base image — every Dockerfile begins here |
| `COPY` | Yes | Copy files from the build context into the image |
| `EXPOSE` | Yes | Document which port the app uses inside the container (does not publish it on your host) |
| `CMD` | Inherited | Default command when the container starts — already defined in the official `nginx` image; we do not repeat it |

**Build context** — the folder you pass to `docker build` (usually `.`). Only files in that folder (and subfolders) can be `COPY`’d.

Other instructions (`RUN`, `WORKDIR`, `ENV`, `ENTRYPOINT`, …) appear in real projects and in [Docker Compose](05-docker-compose.md), when we build multi-container apps from scratch.

---

## Implementation

We will build a tiny website served by nginx, packaged as **your own image** — not `nginx:1.27` directly, but `hello-web:1.0` with your HTML inside.

### Step 1 — Create a project folder (build context)

**Why:** `docker build` needs a directory of files on your machine. Docker sends that folder to the engine; only files inside it can be copied into the image with `COPY`. We call this folder the **build context**.

```bash
mkdir -p ~/docker-lab/hello-web
cd ~/docker-lab/hello-web
```

After `cd`, every command in this lab runs from `~/docker-lab/hello-web`. The final `.` in `docker build` means “this folder”.

### Step 2 — Add the app file we want inside the image

**Why:** A Dockerfile alone is not enough — you need something to package. Here it is a single HTML file. In a real project this could be your built frontend, config files, or scripts.

**What we create:** `index.html` — a minimal page so we can see our custom content in the browser later.

```bash
cat <<'EOF' > index.html
<!DOCTYPE html>
<html>
  <head><title>Hello Docker</title></head>
  <body><h1>Built from a Dockerfile</h1></body>
</html>
EOF
```

You can open the file in an editor to confirm it exists: `cat index.html`

### Step 3 — Write the Dockerfile (build instructions)

**Why:** This file tells Docker *how* to assemble the image — line by line, top to bottom.

**What each line does:**

| Line | Meaning |
|---|---|
| `FROM nginx:1.27` | Start from the official nginx image (OS + nginx already installed). We reuse it instead of building nginx from scratch. |
| `COPY index.html /usr/share/nginx/html/index.html` | Copy **from** your laptop (build context) **into** the path where nginx serves files inside the image. |
| `EXPOSE 80` | Documents that the app listens on port 80 inside the container. Does **not** open the port on your Mac by itself — `-p` at `docker run` does that. |

```bash
cat <<'EOF' > Dockerfile
FROM nginx:1.27
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
EOF
```

Note: we do not set `CMD` because the base `nginx` image already defines how to start nginx. We inherit that behavior.

### Step 4 — Build the image

**Why:** `docker build` reads the Dockerfile and creates a new **image layer** on top of `nginx:1.27` with your HTML baked in.

**Flags:**

- `-t hello-web:1.0` — **tag** (name) the result so you can `docker run hello-web:1.0` instead of a long image ID
- `.` — build context = current directory (`~/docker-lab/hello-web`)

```bash
docker build -t hello-web:1.0 .
```

Watch the output during the build. Docker reads each Dockerfile line as a step.

**Modern Docker (BuildKit — default on Docker Desktop):** you see a progress bar and lines like:

```text
[+] Building ... FINISHED
 => [1/2] FROM docker.io/library/nginx:1.27
 => [2/2] COPY index.html /usr/share/nginx/html/index.html
 => => naming to docker.io/library/hello-web:1.0
 => => unpacking to docker.io/library/hello-web:1.0
```

Success = **`FINISHED`** and **`naming to ... hello-web:1.0`**. You will **not** always see the older text `Successfully tagged hello-web:1.0` — that is normal.

**Older / legacy builder:** may print `Successfully tagged hello-web:1.0` at the end instead. Either format means the build worked.

Confirm the new image exists alongside `nginx:1.27`:

```bash
docker images | grep -E 'hello-web|nginx'
```

You now have **two** related images: the base `nginx:1.27` and your child `hello-web:1.0`.

### Step 5 — Run a container from your image

**Why:** An image is only a template until you **run** it. We start a container in the background (`-d`) with a memorable name (`--name hello-web`).

**Port mapping `-p 8080:80`:**

- **8080** — port on your Mac (what you type in the browser / `curl`)
- **80** — port **inside** the container where nginx listens

Traffic: `localhost:8080` → Docker forwards → container port `80` → nginx.

```bash
docker run -d --name hello-web -p 8080:80 hello-web:1.0
```

Check it is running:

```bash
docker ps --filter name=hello-web
```

### Step 6 — Test the app

**Why:** Verify the container serves **your** HTML, not the default nginx welcome page.

```bash
curl -s http://localhost:8080/ | head -n 5
```

You should see `<h1>Built from a Dockerfile</h1>`. Or open http://localhost:8080 in a browser.

Optional — see nginx logs from this container:

```bash
docker logs hello-web
```

---

## Verification

Build succeeded if **either**:

- BuildKit: last lines include `FINISHED` and `naming to ... hello-web:1.0`, **or**
- Legacy builder: `Successfully tagged hello-web:1.0`

Then confirm the image exists:

```bash
docker images hello-web
```

- `curl http://localhost:8080/` returns your HTML heading `Built from a Dockerfile`.

---

## Break & repair

**Break:** Remove `index.html` from the build context but keep `COPY index.html ...` in the Dockerfile, then rebuild:

```bash
mv index.html index.html.bak
docker build -t hello-web:1.0 .
```

**Observe:** Build fails with `file not found` on `COPY`.

**Repair:** Restore the file and rebuild:

```bash
mv index.html.bak index.html
docker build -t hello-web:1.0 .
```

**Break:** Change `EXPOSE 80` to `EXPOSE 9999` but keep `-p 8080:80`. Site still works — `EXPOSE` is metadata; **port mapping** at `docker run` is what matters.

**Repair:** Understand that publishing uses `-p`, not `EXPOSE` alone. Remove the wrong mental model; optional fix:

```bash
docker rm -f hello-web 2>/dev/null || true
docker run -d --name hello-web -p 8080:80 hello-web:1.0
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:8080/
```

Expect `200`.

---

## Cleanup / revert

Removing a lab has **two separate steps**: delete the **container** (running instance), then delete the **image** (template).

### Step 1 — Container (you already did this)

```bash
docker stop hello-web    # send stop signal (skip if already stopped)
docker rm hello-web      # remove the container record
```

`docker rm hello-web` printing `hello-web` means success.

### Step 2 — Image (use the full tag)

When you built, you tagged the image **`hello-web:1.0`** (`name:tag`).

| Command | What Docker looks for |
|---|---|
| `docker rmi hello-web:1.0` | Correct — matches your build tag |
| `docker rmi hello-web` | Same as `hello-web:latest` — **not** your image, so you get `No such image: hello-web:latest` |

Remove the image:

```bash
docker rmi hello-web:1.0
```

Verify it is gone:

```bash
docker images hello-web
```

Expected: empty table (no rows for `hello-web`).

### List images — correct commands

| Command | Works? |
|---|---|
| `docker images` | Yes — list images (what you used) |
| `docker images -a` | Yes — include intermediate layers (rarely needed) |
| `docker image ls` | Yes — same as `docker images` |
| `docker image -a` | **No** — `docker image` is a group command; `-a` is not valid here |

### If `docker rmi` still fails

**Error: image is being used by a container**

```bash
docker ps -a --filter ancestor=hello-web:1.0
docker rm <container_id>
docker rmi hello-web:1.0
```

**Error: conflict / must force**

```bash
docker rmi -f hello-web:1.0
```

Use `-f` only when you understand no container should keep using that image.

Keep `nginx:1.27` if other labs still need it (base layer may remain shared under the hood).

More cleanup patterns: [Docker basics — clean up](01-docker-basics.md#implementation--clean-up-images-and-containers).

---

## You should now be able to…

- Explain what `FROM`, `COPY`, and `EXPOSE` do in our example.
- Explain why we do not write `CMD` in this Dockerfile (inherited from `nginx`).
- Explain why `EXPOSE 80` is not enough to reach the app from your browser without `-p`.
- Build an image with `docker build -t name:tag .` and explain what the `.` (build context) means.
- Run the image with `-p 8080:80` and confirm the app responds.

Previous: [Docker basics](01-docker-basics.md) · Next: [Build, run, logs, exec](03-build-run-logs-exec.md)
