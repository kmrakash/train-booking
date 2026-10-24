# Train Booking System — Master Project Context

> This file is the canonical reference for all future sprint planning.
> Read this before generating any new sprint's tickets.

---

## Goal

Build a Train Booking System as a learning project.
Start with a functional MVP, scale incrementally to 1 million users.
Teach by building — explain concepts only when they appear in the code.

---

## Stack (non-negotiable)

| Layer | Technology |
|-------|-----------|
| Backend | Java 21, Spring Boot 3, REST APIs, Microservices Architecture |
| Frontend | Angular 17+ |
| Database | PostgreSQL |
| DevOps | GitHub, GitHub Actions (CI/CD), Docker, Kubernetes |
| Cloud | AWS (EC2, RDS, S3, SQS, ElastiCache) |
| Extras | Redis, Kafka (introduced when the bottleneck demands it) |

---

## Repo Structure (monorepo)

```
/apps/backend   — Spring Boot monolith to start
/apps/frontend  — Angular app
/infra          — Docker, K8s manifests, CI/CD configs
/docs           — specs, ADRs, sprint tickets
```

---

## Phases / Sprints

### Sprint 1 — MVP (functional, no scale)
- Features: search trains, book a seat, view/cancel booking, basic auth
- Stack: Spring Boot monolith, Angular, PostgreSQL, basic SQL schema
- Deliverable: runs locally with Docker Compose
- Learning: Spring Boot fundamentals, Angular components/services/routing, REST API design, SQL schema design and joins

### Sprint 2 — Real SQL + Auth
- Add: Spring Security + JWT, seat availability with concurrent booking, normalised schema with proper indexing, complex SQL queries
- Problem to solve: two users booking the last seat simultaneously
- Learning: transactions, row-level locking, query optimisation, explain plans

### Sprint 3 — CI/CD + GitHub
- Add: GitHub Actions pipeline (build, test, lint, deploy to dev), Dockerise both apps, environment configs
- Deliverable: push to main auto-deploys to a staging environment
- Learning: CI/CD best practices, Docker multi-stage builds

### Sprint 4 — Microservices Split
- Split monolith into: booking-service, payment-service, notification-service, train-service, api-gateway
- Problem to solve: distributed transactions, service-to-service auth
- Learning: MSA patterns, API Gateway, service discovery, Feign clients

### Sprint 5 — Async + Queues
- Add: Kafka for booking events, async notifications (email/SMS), eventual consistency between services
- Problem to solve: payment confirmed but booking not updated (failure scenario)
- Learning: event-driven architecture, dead letter queues, idempotency

### Sprint 6 — Caching + Performance
- Add: Redis for seat availability cache, DB connection pooling, N+1 query fixes, pagination
- Problem to solve: 10,000 users searching the same train simultaneously
- Learning: cache invalidation strategies, read vs write optimisation

### Sprint 7 — AWS + Production
- Deploy to: AWS ECS or EKS, RDS PostgreSQL, ElastiCache (Redis), SQS, S3 for tickets/reports, CloudWatch monitoring
- Problem to solve: deploy with zero downtime, rollback strategy
- Learning: AWS core services, blue/green deployment, infra as code

### Sprint 8 — 1 Million User Scale
- Add: horizontal scaling, rate limiting, circuit breakers (Resilience4j), load testing with k6, database read replicas
- Problem to solve: Black Friday sale — 50,000 concurrent seat bookings
- Learning: load balancing, auto-scaling, chaos engineering basics

---

## AI Rules (how to behave while building)

1. One phase/sprint at a time. Do not mention a future sprint while in the current one.
2. When the user hits an error or bottleneck, ask leading questions first — do not give the fix directly.
3. Before each new concept, show WHERE the bottleneck is in the code, then introduce the solution.
4. Every feature must have a failing test before the implementation.
5. Keep code clean — no over-engineering for the current phase.
6. End each sprint with: what we built, what broke, what we learned, and what the next bottleneck will be.

---

## Sprint Progress Tracker

| Sprint | Status | Completed Date |
|--------|--------|---------------|
| Sprint 1 — MVP | 🔄 In Progress | — |
| Sprint 2 — Real SQL + Auth | ⬜ Not Started | — |
| Sprint 3 — CI/CD + GitHub | ⬜ Not Started | — |
| Sprint 4 — Microservices | ⬜ Not Started | — |
| Sprint 5 — Async + Queues | ⬜ Not Started | — |
| Sprint 6 — Caching | ⬜ Not Started | — |
| Sprint 7 — AWS | ⬜ Not Started | — |
| Sprint 8 — Scale | ⬜ Not Started | — |

---

## Ticket Progress — Sprint 1

Update this table as each ticket is completed.

| Ticket | Title | Status |
|--------|-------|--------|
| TRAIN-001 | Monorepo & Project Scaffolding | ⬜ To Do |
| TRAIN-002 | Database Schema & Flyway Migrations | ⬜ To Do |
| TRAIN-003 | Train Entity & Search API | ⬜ To Do |
| TRAIN-004 | User Registration & Basic Auth | ⬜ To Do |
| TRAIN-005 | Create Booking API | ⬜ To Do |
| TRAIN-006 | View & Cancel Booking API | ⬜ To Do |
| TRAIN-007 | Angular App Setup & Routing | ⬜ To Do |
| TRAIN-008 | Train Search UI | ⬜ To Do |
| TRAIN-009 | Booking Flow UI | ⬜ To Do |
| TRAIN-010 | My Bookings UI (View & Cancel) | ⬜ To Do |
| TRAIN-011 | Docker Compose for Local Dev | ⬜ To Do |

---

## How to Use This File When Starting a New Sprint

1. Check the Sprint Progress Tracker — mark the completed sprint done.
2. Read the next sprint's section above to know scope and learning goals.
3. Generate tickets only for that sprint — do not look further ahead.
4. Reference the completed sprint's tickets for any API contracts or DB schema decisions that carry forward.
