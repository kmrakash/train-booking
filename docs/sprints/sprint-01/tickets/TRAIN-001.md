# TRAIN-001: Monorepo & Project Scaffolding

## Summary
Set up the monorepo folder structure, initialize the Spring Boot backend project, initialize the Angular frontend project, and add placeholder README files so the repo is navigable from day one.

## Type
Task

## Priority
Critical

## Story Points
3

## Epic
Sprint 1 — MVP

## Sprint
Sprint 1

---

## Description

Before any feature code can be written, the repo needs a clean, agreed-upon structure that all future tickets build on. This ticket creates the skeleton — no business logic, just project setup.

The backend will use Spring Initializr defaults for a Spring Boot 3 + Java 21 project. The frontend will use the Angular CLI to scaffold a new app. Both projects must compile and run (even if they show a default page) before this ticket is closed.

---

## Acceptance Criteria

- [ ] Root folder is `train-booking/` (monorepo root)
- [ ] `apps/backend/` contains a valid Spring Boot 3 project generated via Spring Initializr
  - Java 21
  - Dependencies: Spring Web, Spring Data JPA, PostgreSQL Driver, Flyway Migration, Lombok, Spring Boot DevTools
  - Group: `com.trainbooking`
  - Artifact: `backend`
  - Packaging: JAR
- [ ] `apps/frontend/` contains a valid Angular 17+ app generated via Angular CLI
  - Standalone components (no NgModules)
  - Routing enabled
  - CSS (not SCSS for now, keep it simple)
- [ ] `infra/` folder exists (empty, with a `.gitkeep`)
- [ ] `docs/` folder exists with this ticket and sprint files
- [ ] Root `README.md` explains the repo structure and how to run each app
- [ ] Backend starts with `./mvnw spring-boot:run` (may fail on DB connection — that is expected and OK)
- [ ] Frontend starts with `ng serve` and shows the Angular default page at `localhost:4200`
- [ ] `.gitignore` at root covers Java (`target/`, `*.class`) and Node (`node_modules/`, `dist/`)

---

## Technical Notes

- Use [https://start.spring.io](https://start.spring.io) or the CLI to generate the backend
- Use `ng new frontend --routing --style=css --standalone` for Angular
- Do NOT add any business logic in this ticket
- The Spring Boot app will crash on startup because PostgreSQL is not yet running — this is fine; the ticket is about scaffolding, not connectivity

---

## TDD Note

No tests to write in this ticket. Validation is structural (do the projects exist and compile).

---

## Dependencies

None — this is the first ticket.

---

## Definition of Done

- [ ] `apps/backend/` compiles: `./mvnw clean compile` exits 0
- [ ] `apps/frontend/` compiles: `ng build` exits 0
- [ ] Root `README.md` exists and describes how to run both apps
- [ ] All files committed to git

---

## Validation Steps

Run these commands exactly to confirm the ticket is complete:

```bash
# From monorepo root

# 1. Backend compiles
cd apps/backend && ./mvnw clean compile
# Expected: BUILD SUCCESS

# 2. Frontend compiles
cd ../frontend && ng build
# Expected: Build at apps/frontend/dist/ — no errors

# 3. Check structure
ls ../../
# Expected: apps/  docs/  infra/  README.md  .gitignore

ls apps/
# Expected: backend/  frontend/

ls apps/backend/src/main/java/com/trainbooking/
# Expected: BackendApplication.java

ls apps/frontend/src/app/
# Expected: app.component.ts  app.config.ts  app.routes.ts
```

---

## Notes / Decisions

- Lombok is included from day one to reduce boilerplate; it will be used starting TRAIN-003
- Flyway is included from day one even though migrations are written in TRAIN-002 — having it on the classpath keeps the Spring context clean
