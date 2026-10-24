# Sprint 1 — MVP

## Sprint Goal

Deliver a fully working Train Booking System that runs on a developer's laptop
via Docker Compose. No cloud, no scale — just working end-to-end flows.

## Duration

2 weeks (adjust to your pace — treat each ticket as a day's work)

## Scope

| In Scope | Out of Scope |
|----------|-------------|
| Search trains by source, destination, date | Payment |
| Book a seat (single passenger) | Email/SMS notifications |
| View my bookings | Admin panel |
| Cancel a booking | Seat class selection |
| User registration + login (Basic Auth) | JWT / OAuth |
| PostgreSQL schema (trains, users, bookings) | Redis, Kafka, S3 |
| Docker Compose to start everything locally | AWS, Kubernetes |

## Deliverable

Running `docker compose up` brings up:
- Spring Boot API on `http://localhost:8080`
- Angular app on `http://localhost:4200`
- PostgreSQL on `localhost:5432`

A user can register, search trains, book a seat, view their booking, and cancel it.

## Learning Objectives

By the end of this sprint you will understand:
- Spring Boot project structure (controllers, services, repositories)
- REST API design (resource naming, HTTP verbs, status codes)
- JPA / Hibernate basics (entities, relationships, repositories)
- PostgreSQL schema design (tables, foreign keys, constraints)
- Angular fundamentals (components, services, routing, HttpClient)
- Docker Compose multi-service setup

## Team (solo project — you wear all hats)

| Role | Responsibility |
|------|---------------|
| Backend Dev | Spring Boot APIs, JPA, DB schema |
| Frontend Dev | Angular UI, services, routing |
| DevOps | Docker Compose, environment config |

## Ticket Execution Order

Work tickets strictly in this order — each one unblocks the next.

```
TRAIN-001  →  TRAIN-002  →  TRAIN-003  →  TRAIN-004
                                              ↓
TRAIN-011  ←  TRAIN-010  ←  TRAIN-009  ←  TRAIN-005
                                              ↓
                                           TRAIN-006
                                              ↓
                                           TRAIN-007
                                              ↓
                                           TRAIN-008
```

Simplified linear order to follow:

1. TRAIN-001 — Monorepo & Project Scaffolding
2. TRAIN-002 — Database Schema & Flyway Migrations
3. TRAIN-003 — Train Entity & Search API
4. TRAIN-004 — User Registration & Basic Auth
5. TRAIN-005 — Create Booking API
6. TRAIN-006 — View & Cancel Booking API
7. TRAIN-007 — Angular App Setup & Routing
8. TRAIN-008 — Train Search UI
9. TRAIN-009 — Booking Flow UI
10. TRAIN-010 — My Bookings UI (View & Cancel)
11. TRAIN-011 — Docker Compose for Local Dev

## Definition of Done (Sprint Level)

- [ ] All 11 tickets marked complete
- [ ] `docker compose up` starts the full stack with no manual steps
- [ ] End-to-end flow works: register → login → search → book → view → cancel
- [ ] All API endpoints tested with curl or Postman
- [ ] All unit tests pass (`./mvnw test`)
- [ ] Angular app builds with no errors (`ng build`)

## Sprint Retrospective Template (fill in when done)

**What we built:**

**What broke / surprised us:**

**What we learned:**

**What the next bottleneck will be (hint for Sprint 2):**
