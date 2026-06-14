# Scenario 06 — K8s Deployments & Services

## Goal
Deploy the Employee Service using a Kubernetes Deployment for self-healing and scaling, and expose it via a Kubernetes Service for stable access and load balancing.

---

## Core Concepts Learned

| Concept | What It Means |
|---------|--------------|
| **Deployment** | Manages pod lifecycle — scaling, self-healing, rolling updates |
| **Replicas** | Number of pod instances Deployment maintains at all times |
| **Labels** | Key-value tags stamped on pods by the Deployment |
| **Selector** | Tells the Deployment which pods to manage (by matching labels) |
| **Service** | Stable network entry point in front of pods — load balances traffic |
| **NodePort** | Service type that exposes app on a port of the node — accessible from outside |
| **ClusterIP** | Service type accessible only inside the cluster — for internal communication |
| **LoadBalancer** | Service type that requests cloud provider to create a load balancer (AWS ALB etc.) |
| **Rolling update** | K8s updates pods one by one — zero downtime deployments |

---

## Deployment vs Plain Pod

| | Plain Pod (Scenario 05) | Deployment (Scenario 06) |
|---|---|---|
| Self-healing | ❌ Stays dead if crashed | ✅ Auto-recreates |
| Scaling | ❌ Manual | ✅ Set replicas |
| Rolling updates | ❌ Not possible | ✅ Built-in |

---

## How Labels & Selectors Work

```
You create Deployment
  → reads template → creates Pod 1 → stamps label: app=employee-service
  → reads template → creates Pod 2 → stamps label: app=employee-service
  → reads template → creates Pod 3 → stamps label: app=employee-service

selector: matchLabels: app=employee-service
  → finds all 3 pods → monitors them
  → if one crashes → creates fresh pod from template → stamps same label
```

> The Deployment creates the pods — they don't exist before. The template is the blueprint.

---

## The deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: employee-service-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: employee-service
  template:
    metadata:
      labels:
        app: employee-service
    spec:
      containers:
        - name: employee-service
          image: nassirsultan/employee-service:latest
          ports:
            - containerPort: 8080
```

---

## The service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: employee-service
spec:
  selector:
    app: employee-service
  ports:
    - port: 8080
      targetPort: 8080
  type: NodePort
```

> `selector: app: employee-service` — Service finds pods by their label, not by Deployment name.
> `port` = Service listens here. `targetPort` = pod's internal port where app runs.

---

## Service Types

| Type | Accessible From | Use Case |
|------|----------------|----------|
| ClusterIP | Inside cluster only | Internal microservice communication |
| NodePort | Outside cluster (your laptop) | Local testing, Minikube |
| LoadBalancer | Internet via cloud load balancer | Production on AWS/GCP |

> For internal services (e.g. Payment Service only called by Policy Service) → use ClusterIP.
> For public-facing services → use NodePort (local) or LoadBalancer (cloud).

---

## Rolling Updates

When you deploy a new image version, K8s updates pods one by one — never all at once:

```
Step 1: Pod1(v2) Pod2(v1) Pod3(v1)  ← 2 pods still serving v1
Step 2: Pod1(v2) Pod2(v2) Pod3(v1)  ← 1 pod still serving v1
Step 3: Pod1(v2) Pod2(v2) Pod3(v2)  ← all updated, zero downtime
```

Default behavior in Deployment — no extra configuration needed.

---

## Commands

```bash
# Apply deployment
kubectl apply -f deployment.yaml

# Apply service
kubectl apply -f service.yaml

# List pods (should show 3)
kubectl get pods

# List services
kubectl get services

# Get service URL (Minikube)
minikube service employee-service --url

# Test self-healing — delete a pod and watch it recreate
kubectl delete pod <pod-name>
kubectl get pods

# Scale up or down
kubectl scale deployment employee-service-deployment --replicas=5
```

---

## Self-Healing Demo

```
kubectl delete pod employee-service-deployment-xxxxx
→ Pod deleted
→ kubectl get pods → still shows 3 pods (new one created automatically)
```

Deployment detected desired state (3 replicas) didn't match actual state (2 pods) and immediately created a replacement.

---

## Key Takeaway

> A Deployment + Service is the standard unit of deployment in Kubernetes. Deployment keeps your pods alive and scaled. Service makes them reliably accessible. Together they give you production-grade reliability — self-healing, scaling, load balancing, and zero-downtime deployments — all declaratively defined in two YAML files.
