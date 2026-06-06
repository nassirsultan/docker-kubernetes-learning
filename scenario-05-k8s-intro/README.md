# Scenario 05 — Kubernetes Intro (Minikube + Pod)

## Goal
Install Minikube, understand core Kubernetes concepts, write a Pod manifest, deploy it, and access the running app.

---

## Core Concepts Learned

| Concept | What It Means |
|---------|--------------|
| **Kubernetes (K8s)** | A system that manages containers across multiple servers — self-healing, scaling, load balancing |
| **Cluster** | Multiple servers (nodes) connected together and managed by Kubernetes |
| **Node** | A single server in the cluster — either control plane (brain) or worker (runs containers) |
| **Control plane** | The master node — schedules pods, monitors health, restarts crashes |
| **Worker node** | Runs your actual containers as Pods |
| **Pod** | The smallest deployable unit in Kubernetes — a wrapper around one or more containers |
| **Manifest** | A YAML file describing what you want Kubernetes to run |
| **Minikube** | A single-node Kubernetes cluster that runs locally on your laptop |
| **kubectl** | The CLI tool to interact with Kubernetes |

> **Why Pod and not container directly?**
> A Pod can hold multiple tightly-related containers that share the same network and storage — for example, your app + a logging agent running side by side. Most of the time: one Pod = one container.

> **K8s = shorthand** for Kubernetes. "K" + 8 letters + "s". Like i18n = internationalization.

---

## Why Kubernetes for Microservices?

Without Kubernetes — if a container crashes at 2 AM, someone has to manually restart it. If traffic spikes, someone has to manually spin up more instances.

Kubernetes solves this automatically:
- **Self-healing** — detects crashed containers and restarts them automatically
- **Scaling** — spins up more Pod instances under heavy load
- **Load balancing** — distributes traffic across healthy instances
- **Scheduling** — decides which server a container should run on

---

## Where Your Pod Runs

```
Windows 11
  └── WSL2 (Linux layer)
        └── Docker Desktop
              └── Minikube (Docker container acting as K8s node)
                    └── Kubernetes
                          └── Pod: pod1
                                └── employee-service container
```

---

## The pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
    - name: employee-service
      image: nassirsultan/employee-service:latest
      ports:
        - containerPort: 8080
```

**Why `nassirsultan/employee-service:latest` and not just `employee-service`?**
Minikube runs in its own isolated environment — it cannot see images on your laptop's Docker. It pulls from Docker Hub instead. That's why we pushed to Docker Hub in Scenario 04.

---

## Commands

```bash
# Install Minikube (Windows)
winget install Kubernetes.minikube

# Verify installation
minikube version

# Start local K8s cluster (uses Docker as driver)
minikube start

# Check cluster status
minikube status

# Deploy pod from manifest
kubectl apply -f pod.yaml

# List pods and their status
kubectl get pods

# Forward laptop port to pod port (for local testing only)
kubectl port-forward pod/pod1 8080:8080

# Describe pod (useful for debugging)
kubectl describe pod pod1

# View pod logs
kubectl logs pod1
```

---

## How Port Forwarding Works

```
Postman → localhost:8080 → kubectl port-forward tunnel → pod1:8080
```

`kubectl port-forward` creates a temporary tunnel from your laptop into the pod. It only works while the terminal is open.

> **This is NOT production-ready.** Closing the terminal kills the tunnel and requests stop reaching the pod. The proper solution is a Kubernetes **Service** — covered in Scenario 06.

---

## Key Takeaway

> In Docker, you ran containers directly with `docker run`. In Kubernetes, you **declare** what you want in a YAML file and Kubernetes makes it happen — and keeps it that way. If the pod crashes, Kubernetes restarts it. That shift from imperative ("run this") to declarative ("I want this running") is the core idea of Kubernetes.
