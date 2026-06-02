# Scenario 02 — Docker CLI Basics

## Goal
Learn to control containers using Docker CLI commands — running in background, viewing logs, getting inside containers, and cleaning up.

---

## Commands Learned

### Running a Container in the Background

```bash
docker run -d -p 8080:8080 employee-service
```

| Flag | Meaning |
|------|---------|
| `-d` | Detached mode — runs container in background, frees up terminal |
| `-p 8080:8080` | Port mapping — `host:container` |

> Without `-d`, the terminal is locked to the container. With `-d`, you get your terminal back immediately.

---

### Listing Containers

```bash
docker ps        # only running containers
docker ps -a     # all containers (running + stopped)
```

> Every `docker run` creates a **new** container. Over time `docker ps -a` shows a history of all runs.

---

### Viewing Logs

```bash
docker logs <container_id>       # print all logs
docker logs -f <container_id>    # follow logs in real time (like tail -f)
```

> `-f` streams live logs. Press `Ctrl + C` to stop following — container keeps running.

---

### Getting Inside a Container

```bash
docker exec -it <container_id> bash
```

| Part | Meaning |
|------|---------|
| `exec` | Execute a command inside a running container |
| `-it` | Interactive terminal — gives you a shell prompt |
| `bash` | The shell to open |

> Once inside, you're in a Linux shell. You can see your `app.jar`, check files, inspect environment variables.
> Type `exit` to come out — container keeps running.

---

### Stopping and Starting Containers

```bash
docker stop <container_id>    # stop a running container
docker start <container_id>   # restart an existing stopped container
```

> **Key difference:**
> - `docker run` → creates a **new** container from an image
> - `docker start` → restarts an **existing** stopped container

---

### Cleaning Up Containers

```bash
docker rm <container_id>      # remove a specific stopped container
docker container prune        # remove ALL stopped containers at once
```

> You can only remove **stopped** containers. Stop them first with `docker stop`.

---

## Full Command Reference

```bash
docker run -d -p 8080:8080 employee-service   # new container from image (detached)
docker start <container_id>                    # restart existing container
docker stop <container_id>                     # stop running container
docker ps                                      # list running containers
docker ps -a                                   # list all containers
docker logs <container_id>                     # view logs
docker logs -f <container_id>                  # follow live logs
docker exec -it <container_id> bash            # get inside container
docker rm <container_id>                       # remove stopped container
docker container prune                         # remove all stopped containers
```

---

## Key Takeaway

> A container is not the same as an image. Every `docker run` creates a fresh container instance from the same image. Think of it like creating multiple objects from the same class — each object is independent.
