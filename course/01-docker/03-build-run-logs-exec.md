# Build, run, logs, exec

**Prerequisite:** [Dockerfile basics](02-dockerfile-basics.md).

Commands run in your **local terminal**. Examples reuse `~/docker-lab/hello-web` from the previous lesson (or recreate it there).

---

## Explanation

Once you have a Dockerfile, day-to-day Docker work is a small set of commands:

- **`docker build`** — create or update an image from a Dockerfile.
- **`docker run`** — create and start a container from an image.
- **`docker logs`** — read stdout/stderr from a container (essential for debugging).
- **`docker exec`** — run a command **inside** a running container (like SSH without SSH).

**Problem this solves:** You need to start apps in the background, watch what they print, and inspect the filesystem or run tools inside an isolated environment — without rebuilding the image each time.

**Detached (`-d`)** vs **interactive (`-it`)**:

- `-d` — container runs in the background; you get a container ID back.
- `-it` — attach a terminal for shell-style interaction (common with `docker run -it ... sh`).

**Port mapping (`-p hostPort:containerPort`)** publishes a container port on your machine so `curl localhost:hostPort` reaches the app.

---

## Implementation

Ensure the image exists (from lesson 02):

```bash
cd ~/docker-lab/hello-web
docker build -t hello-web:1.0 .
```

Run in the **background** with a name and port map:

```bash
docker run -d --name hello-web -p 8080:80 hello-web:1.0
```

Check it is running:

```bash
docker ps --filter name=hello-web
```

Read **logs** (nginx access/error output):

```bash
docker logs hello-web
```

Generate traffic, then read logs again:

```bash
curl -s http://localhost:8080/ > /dev/null
docker logs --tail 5 hello-web
```

**Exec** into the container and list the web root:

```bash
docker exec hello-web ls -la /usr/share/nginx/html
```

Open an interactive shell inside the container:

```bash
docker exec -it hello-web sh
```

Inside the shell, try:

```sh
cat /usr/share/nginx/html/index.html
exit
```

Stop and remove when done:

```bash
docker stop hello-web
docker rm hello-web
```

**Interactive one-off** (no `-d`) — container exits when you leave the shell:

```bash
docker run --rm -it hello-web:1.0 sh
```

Type `exit` to leave.

---

## Verification

- `docker ps` shows `hello-web` with `0.0.0.0:8080->80/tcp` in the PORTS column.
- `docker logs hello-web` shows nginx startup lines (and access lines after `curl`).
- `docker exec hello-web ls ...` lists `index.html`.

---

## Break & repair

**Break:** Map the wrong host port (container still listens on 80 internally):

```bash
docker run -d --name hello-web -p 9090:80 hello-web:1.0
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:8080/
```

**Observe:** Connection fails or non-200 on 8080 — nothing is published there.

**Repair:** Use the mapped port or recreate with `-p 8080:80`:

```bash
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:9090/
docker rm -f hello-web
docker run -d --name hello-web -p 8080:80 hello-web:1.0
```

**Break:** Exec into a **stopped** container:

```bash
docker stop hello-web
docker exec hello-web ls
```

**Observe:** Error — container is not running.

**Repair:**

```bash
docker start hello-web
docker exec hello-web ls /usr/share/nginx/html
```

---

## You should now be able to…

- Run a container in the background with `-d`, `-p`, and `--name`.
- Read container output with `docker logs` (including `--tail`).
- Use `docker exec` and `docker exec -it` to run commands inside a running container.

Previous: [Dockerfile basics](02-dockerfile-basics.md) · Next: [Volumes and networking](04-volumes-and-networking.md)
