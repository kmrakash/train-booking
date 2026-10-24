# Train Booking System — Docs

This folder is the single source of truth for all planning, requirements, and tickets.

## Structure

```
docs/
  README.md                  ← you are here
  sprints/
    sprint-01/
      sprint-overview.md     ← goals, scope, team capacity
      backlog.md             ← ordered list of all tickets in this sprint
      tickets/
        TRAIN-001.md
        TRAIN-002.md
        ...
```

## Ticket ID Convention

```
TRAIN-NNN
```

- `TRAIN` — project prefix
- `NNN`   — sequential number, never reused

## Sprint Naming

Sprints map 1-to-1 with phases in the project plan.

| Sprint | Phase | Theme |
|--------|-------|-------|
| Sprint 1 | Phase 1 | MVP — runs locally with Docker Compose |
| Sprint 2 | Phase 2 | Real SQL + Auth |
| Sprint 3 | Phase 3 | CI/CD + GitHub Actions |
| Sprint 4 | Phase 4 | Microservices Split |
| Sprint 5 | Phase 5 | Async + Queues (Kafka) |
| Sprint 6 | Phase 6 | Caching + Performance (Redis) |
| Sprint 7 | Phase 7 | AWS + Production |
| Sprint 8 | Phase 8 | 1 Million User Scale |

## How to Work a Ticket

1. Read `backlog.md` — tickets are ordered by dependency, pick the top unstarted one.
2. Open the ticket file, read **Acceptance Criteria** and **Validation Steps** end-to-end before writing a line of code.
3. Write the failing test first (see ticket's **TDD Note**).
4. Implement until tests pass.
5. Walk through **Validation Steps** manually.
6. Mark checklist items done in the ticket file.
7. Move to next ticket.
