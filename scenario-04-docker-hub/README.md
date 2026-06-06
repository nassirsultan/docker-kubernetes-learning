# Scenario 04 — Push Image to Docker Hub

## Goal
Tag and push the Employee Service image to Docker Hub so it can be pulled from anywhere.

---

## What is Docker Hub?

Docker Hub is like **GitHub but for Docker images**.

- GitHub hosts your **code** — anyone can clone it
- Docker Hub hosts your **images** — anyone can pull it

When you wrote `mysql:8.0` or `eclipse-temurin:17-jdk` in your Dockerfile, Docker pulled those images from Docker Hub automatically. Now we're doing the same — publishing our own image.

---

## Why Push to Docker Hub?

Right now your `employee-service` image only exists on your laptop. To deploy it to AWS, share it with a teammate, or use it in a CI/CD pipeline — you need it hosted somewhere accessible.

**Workflow:**
```
Your Laptop → docker push → Docker Hub → docker pull → AWS / Any Server
```

---

## Core Concepts Learned

| Concept | What It Means |
|---------|--------------|
| **Image tag** | A version label for your image (e.g. `latest`, `1.0`, `2.0`) |
| **Namespaced image** | Images on Docker Hub must be prefixed with your username: `username/image-name` |
| **docker tag** | Renames/retags an existing image without rebuilding it |
| **docker push** | Uploads your image to Docker Hub |
| **docker pull** | Downloads an image from Docker Hub |

---

## Commands

```bash
# Retag your local image with your Docker Hub username
docker tag employee-service nassirsultan/employee-service:latest

# Verify the tag was applied (both point to same Image ID)
docker images

# Login to Docker Hub (if not already logged in via Docker Desktop)
docker login

# Push image to Docker Hub
docker push nassirsultan/employee-service:latest

# Pull image from Docker Hub (from any machine)
docker pull nassirsultan/employee-service:latest
```

---

## Key Observation

After running `docker tag`, both images show the **same Image ID**:

```
employee-service               latest    7830452170df   ...
nassirsultan/employee-service  latest    7830452170df   ...
```

This confirms `docker tag` does not copy or rebuild the image — it just adds a new name pointing to the same image.

---

## Image Versioning

`:latest` is the default tag. In real projects you'd version your images:

```bash
docker tag employee-service nassirsultan/employee-service:1.0
docker tag employee-service nassirsultan/employee-service:2.0
```

This lets you roll back to a previous version anytime by pulling a specific tag.

---

## Key Takeaway

> Pushing to Docker Hub decouples your image from your laptop. Once it's on Docker Hub, any machine — a cloud server, a teammate, a CI/CD pipeline — can pull and run it with a single command. This is the foundation of how microservices get deployed in the real world.
