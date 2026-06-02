# Scenario 03 — Docker Compose

## Goal
Run the Employee Service and MySQL together using Docker Compose — with automatic networking between containers.

---

## The Problem Docker Compose Solves

With plain `docker run`, managing multiple containers is tedious:
- You have to start/stop each container separately
- Containers can't find each other by name — you'd have to manually wire them together
- No single command to bring everything up or down

Docker Compose solves all of this with one file.

---

## Core Concepts Learned

| Concept | What It Means |
|---------|--------------|
| **docker-compose.yml** | A file that defines all your services, their config, and how they connect |
| **Service name as hostname** | Containers talk to each other using service names instead of `localhost` |
| **depends_on** | Tells Docker Compose to start one service before another |
| **environment variables** | Override `application.yml` values without rebuilding the image |
| **Official images** | Pre-built images on Docker Hub (e.g. `mysql:8.0`) — no Dockerfile needed |

> **Key insight:** Inside a container, `localhost` means the container itself — not your PC. To reach another container, use its **service name** defined in `docker-compose.yml`.

---

## The docker-compose.yml

```yaml
version: '3.8'
services:
  employee-service:
    image: employee-service
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql-service:3306/employee_db
      SPRING_DATASOURCE_USERNAME: root
      SPRING_DATASOURCE_PASSWORD: root
    depends_on:
      - mysql-service

  mysql-service:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: employee_db
```

---

## Commands

```bash
# Start all containers (foreground - shows live logs)
docker compose up

# Start all containers (background - detached mode)
docker compose up -d

# Stop and remove all containers
docker compose down

# View logs of a specific service
docker compose logs employee-service

# Follow live logs of a specific service
docker compose logs -f employee-service
```

---

## Errors I Faced & How I Fixed Them

### Error 1 — H2 Driver rejecting MySQL URL
**Error:** `Driver org.h2.Driver claims to not accept jdbcUrl, jdbc:mysql://...`

**Cause:** `application.yml` still had `driver-class-name: org.h2.Driver` hardcoded. Even though Docker Compose passed MySQL environment variables, Spring Boot was explicitly told to use the H2 driver.

**Fix:** Removed `driver-class-name` from `application.yml`. Spring Boot auto-detects the correct driver from the URL.

---

### Error 2 — Wrong Hibernate dialect
**Error:** `SQLSyntaxErrorException: Unknown table 'SEQUENCES'`

**Cause:** `application.yml` had `database-platform: org.hibernate.dialect.H2Dialect` — Hibernate was generating H2-style SQL against a MySQL database.

**Fix:** Changed to `database-platform: org.hibernate.dialect.MySQLDialect`.

---

### Error 3 — Lombok compile errors when rebuilding JAR
**Error:** `variable service not initialized in the default constructor`, `cannot find symbol: getUsername()`

**Cause:** Maven CLI wasn't processing Lombok annotations correctly. Previously the JAR was built through STS which has its own Lombok integration.

**Fix:** Built the JAR using STS → Right click project → Run As → Maven build → Goals: `clean package -DskipTests`. Also added Lombok to `annotationProcessorPaths` in `pom.xml` for future CLI builds.

---

## Important Reminder — Rebuild Order

Whenever you change code or config:

```
1. Fix code / application.yml
2. Build JAR from STS (clean package -DskipTests)
3. docker build -t employee-service .
4. docker compose down
5. docker compose up
```

Docker images don't auto-update — you must rebuild manually.

---

## Key Takeaway

> Docker Compose is the glue for multi-container applications. In microservices, every service runs as its own container. Compose lets you define, connect, and manage them all in one place — which is exactly how real-world Spring Boot microservices are run locally and in staging environments.
