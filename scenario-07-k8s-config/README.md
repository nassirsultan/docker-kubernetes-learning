# Scenario 07 — K8s ConfigMaps & Secrets

## Goal
Separate configuration from code using Kubernetes ConfigMaps (non-sensitive) and Secrets (sensitive), and inject them into the Employee Service Deployment.

---

## The Problem

Hardcoding credentials in `deployment.yaml`:

```yaml
env:
  - name: SPRING_DATASOURCE_PASSWORD
    value: root  # ← exposed in GitHub!
```

Problems:
- DB password visible in GitHub
- Different credentials for dev/staging/prod means maintaining multiple deployment files
- Password rotation requires code changes and redeployment

---

## The Solution — ConfigMap + Secret

| Resource | Stores | Example |
|----------|--------|---------|
| **ConfigMap** | Non-sensitive config | DB URL, server port, log level |
| **Secret** | Sensitive config | DB password, API keys, JWT secret |

Both are **Kubernetes resources** — not pods. They live inside the cluster and pods reference them at startup.

---

## Core Concepts Learned

| Concept | What It Means |
|---------|--------------|
| **ConfigMap** | K8s resource storing non-sensitive key-value config |
| **Secret** | K8s resource storing sensitive config in Base64 encoding |
| **Base64** | Encoding format used by K8s Secrets (not encryption — just encoding) |
| **configMapKeyRef** | Reference a value from a ConfigMap in deployment env |
| **secretKeyRef** | Reference a value from a Secret in deployment env |
| **Spring Boot env override** | Environment variables automatically override application.yml values |

---

## How Spring Boot Reads Environment Variables

Spring Boot automatically overrides `application.yml` with environment variables using this convention:

```
SPRING_DATASOURCE_URL      →  spring.datasource.url
SPRING_DATASOURCE_USERNAME →  spring.datasource.username
SPRING_DATASOURCE_PASSWORD →  spring.datasource.password
```

Uppercase + underscores → lowercase + dots. No code changes needed.

**Priority order:**
```
Environment variables (highest)
  ↓
application.yml
  ↓
Default values (lowest)
```

---

## The configmap.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: employee-config
data:
  DB_URL: jdbc:mysql://mysql-service:3306/employee_db
  DB_NAME: employee_db
  SERVER_PORT: "8080"
```

---

## The secret.yaml

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  DB_PASSWORD: cm9vdA==
  DB_USERNAME: cm9vdA==
```

> `cm9vdA==` is Base64 encoding of `root`
> To encode: `echo -n "root" | base64`
> `type: Opaque` means generic secret

---

## Updated deployment.yaml (env section)

```yaml
env:
  - name: SPRING_DATASOURCE_URL
    valueFrom:
      configMapKeyRef:
        name: employee-config
        key: DB_URL
  - name: SPRING_DATASOURCE_USERNAME
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: DB_USERNAME
  - name: SPRING_DATASOURCE_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: DB_PASSWORD
```

---

## Commands

```bash
# Apply ConfigMap
kubectl apply -f configmap.yaml

# Apply Secret
kubectl apply -f secret.yaml

# Apply updated Deployment
kubectl apply -f deployment.yaml

# Verify ConfigMap exists
kubectl get configmaps

# Verify Secret exists
kubectl get secrets

# View ConfigMap contents
kubectl describe configmap employee-config

# View Secret (values are base64 encoded)
kubectl describe secret db-secret
```

---

## How Teams Manage Multiple Environments

```
k8s/
  ├── dev/
  │   ├── configmap.yaml      # dev DB URL
  │   └── secret.yaml         # dev password
  ├── staging/
  │   ├── configmap.yaml      # staging DB URL
  │   └── secret.yaml         # staging password
  └── prod/
      ├── configmap.yaml      # prod DB URL
      └── secret.yaml         # prod password
```

Same `deployment.yaml` for all environments — only ConfigMap and Secret values change per environment.

For larger teams, tools like **Helm** (K8s package manager) or **Kustomize** (overlay based config) manage this more elegantly.

---

## What Happened When We Applied the Updated Deployment

```
kubectl apply -f deployment.yaml
→ K8s detected template changed (new env variables)
→ Rolling update triggered automatically
→ Old pods (5f95fc75d7) terminated one by one
→ New pods (76949b5fb9) created with ConfigMap + Secret injected
→ Zero downtime
```

---

## Key Takeaway

> Never hardcode credentials in your deployment files or code. ConfigMaps and Secrets separate configuration from deployment — same image runs in dev, staging, and prod with different config injected at runtime. This follows the 12-factor app principle: store config in the environment, not in the code.
