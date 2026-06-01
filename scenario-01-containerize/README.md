# Scenario 01 — Containerize the Spring Boot App

## Goal
Package the Employee Service into a Docker image and run it as a container.

---

## What I Did

Wrote a Dockerfile for my Spring Boot Employee Service, built a Docker image, and ran it as a container on my local machine.

---

## Core Concepts Learned

| Concept | What It Means |
|---------|--------------|
| **Dockerfile** | A file with instructions telling Docker how to build an image |
| **Image** | A bundle containing the app + everything it needs to run (like Java) |
| **Container** | A running instance of an image |

> **Analogy:** Image is like a Java class. Container is like an object (instance) of that class.

---

## The Dockerfile

```dockerfile
FROM eclipse-temurin:17-jdk
COPY target/employee-management-service-0.0.1-SNAPSHOT.jar app.jar
CMD ["java", "-jar", "app.jar"]
```

**Line by line:**
- `FROM eclipse-temurin:17-jdk` — Use a base image that already has Java 17 installed
- `COPY` — Copy the fat JAR from my local `target/` folder into the image
- `CMD` — When the container starts, run the JAR with Java

---

## Commands I Ran

```bash
# 1. Build the fat JAR first
mvn clean package -DskipTests

# 2. Build the Docker image
docker build -t employee-service .

# 3. Verify image was created
docker images

# 4. Run the container
docker run -p 8080:8080 employee-service

# 5. Verify app is healthy
# Hit http://localhost:8080/actuator/health in browser → {"status":"UP"}
```

---

## Errors I Faced & How I Fixed Them

### Error 1 — Docker Desktop not starting
**Error:** `open //./pipe/dockerDesktopLinuxEngine: The system cannot find the file specified`

**Cause:** Docker Desktop was not running. WSL (Windows Subsystem for Linux) needed to be updated.

**Fix:** 
1. Ran `wsl --update` as administrator
2. Restarted the machine
3. Opened Docker Desktop as administrator

**What I learned:** Docker on Windows runs on top of WSL2 — a lightweight Linux environment inside Windows. Without WSL running correctly, Docker cannot start.

---

### Error 2 — H2 Driver not found inside container
**Error:** `Cannot load driver class: org.h2.Driver`

**Cause:** H2 dependency in `pom.xml` was set to `<scope>test</scope>`. This means H2 is only available during tests — not when the app actually runs. The fat JAR did not include H2.

**Fix:** Changed scope from `test` to `runtime`:
```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```
Then rebuilt JAR and image in the correct order:
```bash
mvn clean package -DskipTests   # Step 1 — always first
docker build -t employee-service .  # Step 2 — after JAR is ready
docker run -p 8080:8080 employee-service  # Step 3 — run it
```

**What I learned:** Always rebuild the JAR before rebuilding the Docker image. The Dockerfile just copies whatever JAR exists in `target/` — it does not rebuild it.

---

## Key Takeaway

> A Docker image is self-contained. It bundles the app + Java + all runtime dependencies together. This is why it runs the same way on any machine — your laptop, a teammate's machine, or a cloud server.
