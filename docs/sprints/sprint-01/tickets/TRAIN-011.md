# TRAIN-011: Docker Compose for Local Dev

## Summary
Write a `docker-compose.yml` that starts PostgreSQL, the Spring Boot backend, and the Angular frontend with a single command so any developer can run the entire stack locally without any manual setup.

## Type
Task

## Priority
Medium

## Story Points
2

## Epic
Sprint 1 — MVP

## Sprint
Sprint 1

---

## Description

The sprint's final deliverable is the ability to clone the repo, run `docker compose up`, and have a fully working system. This ticket creates that single-command local setup.

The three services:
1. **db** — PostgreSQL 16 with a named volume so data survives container restarts
2. **backend** — Spring Boot app, built from `apps/backend/Dockerfile`, depends on db
3. **frontend** — Angular app served by Nginx, built from `apps/frontend/Dockerfile`

---

## Services

| Service | Image / Build | Port | Depends On |
|---------|--------------|------|-----------|
| db | `postgres:16` | 5432 | — |
| backend | `./apps/backend` (Dockerfile) | 8080 | db |
| frontend | `./apps/frontend` (Dockerfile) | 4200 → 80 | backend |

---

## Acceptance Criteria

- [ ] `docker-compose.yml` exists at the monorepo root
- [ ] `apps/backend/Dockerfile` builds the Spring Boot JAR and runs it
- [ ] `apps/frontend/Dockerfile` builds the Angular app and serves it with Nginx on port 80
- [ ] `docker compose up` starts all three services with no manual steps
- [ ] PostgreSQL data persists across `docker compose down` and `docker compose up` via a named volume
- [ ] Backend connects to the DB using environment variables (not hardcoded in `application.properties`)
- [ ] Frontend's Nginx config proxies `/api/**` to the backend so CORS is not an issue in Docker
- [ ] After `docker compose up --build`:
  - `http://localhost:8080/api/trains/search?source=Delhi&destination=Mumbai&date=2026-11-01` returns JSON
  - `http://localhost:4200` loads the Angular app
- [ ] `docker compose down -v` tears everything down cleanly (including volumes)

---

## Technical Notes

### `docker-compose.yml`

```yaml
version: '3.9'
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_DB: trainbooking
      POSTGRES_USER: trainuser
      POSTGRES_PASSWORD: trainpass
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U trainuser -d trainbooking"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build: ./apps/backend
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/trainbooking
      SPRING_DATASOURCE_USERNAME: trainuser
      SPRING_DATASOURCE_PASSWORD: trainpass
    depends_on:
      db:
        condition: service_healthy

  frontend:
    build: ./apps/frontend
    ports:
      - "4200:80"
    depends_on:
      - backend

volumes:
  postgres_data:
```

### `apps/backend/Dockerfile` (multi-stage build)

```dockerfile
# Stage 1: Build
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app
COPY mvnw pom.xml ./
COPY .mvn .mvn
RUN ./mvnw dependency:go-offline -q
COPY src src
RUN ./mvnw clean package -DskipTests -q

# Stage 2: Run
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### `apps/frontend/Dockerfile` (multi-stage build)

```dockerfile
# Stage 1: Build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Serve
FROM nginx:alpine
COPY --from=builder /app/dist/frontend/browser /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

### `apps/frontend/nginx.conf`

```nginx
server {
    listen 80;

    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ /index.html;   # Angular SPA routing
    }

    location /api/ {
        proxy_pass http://backend:8080/api/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

The Nginx proxy for `/api/` means the frontend in Docker can call `/api/trains/search` without specifying a host — Nginx forwards it to the backend container. This removes all CORS issues in the Docker environment.

### Environment-based backend config

In `application.properties`, replace hardcoded values with env var placeholders:
```properties
spring.datasource.url=${SPRING_DATASOURCE_URL:jdbc:postgresql://localhost:5432/trainbooking}
spring.datasource.username=${SPRING_DATASOURCE_USERNAME:trainuser}
spring.datasource.password=${SPRING_DATASOURCE_PASSWORD:trainpass}
```
The `${VAR:default}` syntax uses the env var when set (Docker) and the default when not (local dev).

---

## TDD Note

No unit tests for Docker configuration. Validation is functional — the system works end-to-end via `docker compose up`.

---

## Dependencies

- All other tickets (TRAIN-001 through TRAIN-010) must be complete

---

## Definition of Done

- [ ] `docker compose up --build` starts cleanly with no errors
- [ ] All three services are healthy and reachable
- [ ] End-to-end flow works: register → login → search → book → view → cancel
- [ ] `docker compose down` stops everything cleanly

---

## Validation Steps

```bash
# From the monorepo root

# 1. Build and start everything
docker compose up --build

# Wait for all services to be ready (watch logs)
# Expected: 
# - db: "database system is ready to accept connections"
# - backend: "Started BackendApplication"
# - frontend: nginx ready

# 2. Test the backend directly
curl "http://localhost:8080/api/trains/search?source=Delhi&destination=Mumbai&date=2026-11-01"
# Must add auth: -u "akash@example.com:secret123" (register first)

# 3. Open the frontend
open http://localhost:4200
# Expected: Angular app loads, register/login works

# 4. Full end-to-end flow via the browser:
#    - Register a new account
#    - Search Delhi → Mumbai for 2026-11-01
#    - Book seat 10
#    - Navigate to My Bookings
#    - Cancel the booking
#    - Verify seat is available again in search

# 5. Stop and restart — data should persist
docker compose down
docker compose up
# Re-open http://localhost:4200 — previously registered user can still log in
# Previously booked (non-cancelled) bookings still show

# 6. Full teardown
docker compose down -v
# Volumes removed — next start is a clean state
```

---

## Sprint 1 Complete

After this ticket, the sprint is done. Fill in the retrospective in `sprint-overview.md`:
- What we built
- What broke / surprised us
- What we learned
- What the next bottleneck will be (hint: try booking the same seat from two browser windows simultaneously — what happens?)
