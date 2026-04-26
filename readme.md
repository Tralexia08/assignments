# Docker Session 1 — Containers vs VMs: Hands-On Assignment

**Student:** tralexiaaudige  
**Date:** April 26, 2026  
**Topic:** Running your first container, inspecting container state, and understanding port mapping

---

## Objective

Demonstrate the ability to pull a Docker image from Docker Hub, run a named container in detached mode with port mapping, verify it is serving traffic in a browser, and use `docker ps` / `docker container ls` to inspect running and stopped containers.

---

## Step 1 — Run an nginx container in detached mode

The following command was used to pull the `nginx:1.25` image from Docker Hub and start it as a background container named `web`, mapping host port `8080` to container port `80`:

```bash
docker container run -d --name web -p 8080:80 nginx:1.25
```

**What each flag does:**

| Flag | Meaning |
|------|---------|
| `-d` | Detached mode — runs the container in the background |
| `--name web` | Assigns the human-readable name `web` to the container |
| `-p 8080:80` | Maps port 8080 on the host to port 80 inside the container |
| `nginx:1.25` | The image to use — pulled from Docker Hub if not cached locally |

### Terminal output

Since `nginx:1.25` was not cached locally, Docker pulled all image layers from Docker Hub before starting the container. Each hash (e.g. `77cea143f3c3`) represents a separate image layer. Docker downloads and caches layers independently — if another image shares a layer, it is not re-downloaded. The final long hash on the last line is the **container ID** of the newly created `web` container.

![Terminal showing docker container run pulling nginx:1.25 layers and docker ps confirming the container is up](<Screenshot 2026-04-26 at 4.57.54 PM.png>)

> The `docker ps` output confirms the container is running: status `Up 11 seconds`, port mapping `0.0.0.0:8080->80/tcp`, and name `web`.

---

## Step 2 — Verify nginx is serving in the browser

After the container started, navigating to `http://localhost:8080` confirmed nginx was running and serving its default welcome page. This works because of the `-p 8080:80` port mapping — traffic sent to port 8080 on the host is forwarded by Docker into the container on port 80, where nginx is listening.

> **Note:** Add the browser screenshot to this repo to render it here:  
> `![nginx welcome page at localhost:8080](<Screenshot 2026-04-26 at 4.57.43 PM.png>)`

---

## Step 3 — Inspect running and stopped containers

Three variants of the container listing command were used to show different views of container state:

```bash
docker container ls        # running containers only (same as docker ps)
docker container ls -a     # all containers including stopped/exited
docker container ls -q     # just the container IDs (useful in shell scripts)
```

![Terminal showing docker ps, docker container ls, docker container ls -a, and docker container ls -q with full container history](<Screenshot 2026-04-26 at 5.03.50 PM.png>)

### Output breakdown

`docker ps` / `docker container ls` (running only) showed one active container:

| CONTAINER ID | IMAGE | STATUS | PORTS | NAME |
|---|---|---|---|---|
| a094f4412914 | nginx:1.25 | Up 6 minutes | 0.0.0.0:8080→80/tcp | web |

`docker container ls -a` (all containers) revealed the full history of containers on this machine:

| NAME | IMAGE | STATUS |
|---|---|---|
| web | nginx:1.25 | Up 6 minutes |
| c2 | alpine | Exited (0) 9 minutes ago |
| inspectme | alpine | Exited (0) 2 hours ago |
| elegant_rhodes | ubuntu | Exited (0) 3 hours ago |
| kind_goodall | alpine | Exited (0) 3 hours ago |
| determined_mayer | nginx | Exited (0) 31 hours ago |
| nifty_murdock | busybox | Exited (0) 2 days ago |

**Key observations:**
- Stopped containers remain on disk until explicitly removed with `docker rm`
- Docker auto-generates memorable names (e.g. `elegant_rhodes`, `nifty_murdock`) when `--name` is not specified
- Exit code `(0)` means the container stopped cleanly with no error

---

## Key Concepts Demonstrated

### Port mapping (`-p host:container`)
The host machine and the container have separate network namespaces. Port 80 inside the container is not automatically accessible from outside — you must explicitly publish it. `-p 8080:80` creates a forwarding rule so `localhost:8080` routes into the container's port 80.

### Detached mode (`-d`)
Without `-d`, the container's stdout/stderr stream directly to your terminal and stop when you press Ctrl+C. With `-d`, the container runs as a background process and you get your terminal back immediately.

### Image layers
Each `Pull complete` line in the output is a separate layer in the nginx image. Docker stores layers in a content-addressable cache — if two images share a layer, it is stored only once on disk and never downloaded twice.

### Container lifecycle
A container can be in one of several states:

```
created → running → paused → running → stopped/exited → removed
```

`docker ps` only shows `running`. `docker ps -a` shows all states. Exited containers still occupy disk space until removed with `docker rm`.

---

## Commands Reference

```bash
# Run a named container in detached mode with port mapping
docker container run -d --name web -p 8080:80 nginx:1.25

# List running containers
docker ps
docker container ls

# List all containers including stopped
docker ps -a
docker container ls -a

# List just container IDs (useful in scripts)
docker container ls -q

# Stop a running container
docker stop web

# Remove a stopped container
docker rm web

# Force stop and remove in one step
docker rm -f web

# Remove all stopped containers at once
docker container prune
```

---

## Summary

1. `docker container run -d --name web -p 8080:80 nginx:1.25` pulled `nginx:1.25` layer by layer, created the container, and started nginx in the background
2. Port mapping (`-p 8080:80`) made the container accessible at `localhost:8080`, confirming the nginx welcome page loaded successfully
3. `docker ps` and `docker container ls -a` showed both the running container and a full history of stopped containers, demonstrating that exited containers persist on disk until explicitly removed with `docker rm`
