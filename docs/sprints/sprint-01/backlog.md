# Sprint 1 Backlog

Pick tickets from the top. Do not start a ticket until all tickets it depends on are ✅.

| # | Ticket | Title | Type | Points | Priority | Status | Depends On |
|---|--------|-------|------|--------|----------|--------|-----------|
| 1 | TRAIN-001 | Monorepo & Project Scaffolding | Task | 3 | Critical | ⬜ To Do | — |
| 2 | TRAIN-002 | Database Schema & Flyway Migrations | Task | 3 | Critical | ⬜ To Do | TRAIN-001 |
| 3 | TRAIN-003 | Train Entity & Search API | Story | 5 | High | ⬜ To Do | TRAIN-002 |
| 4 | TRAIN-004 | User Registration & Basic Auth | Story | 5 | High | ⬜ To Do | TRAIN-002 |
| 5 | TRAIN-005 | Create Booking API | Story | 5 | High | ⬜ To Do | TRAIN-003, TRAIN-004 |
| 6 | TRAIN-006 | View & Cancel Booking API | Story | 3 | High | ⬜ To Do | TRAIN-005 |
| 7 | TRAIN-007 | Angular App Setup & Routing | Task | 3 | High | ⬜ To Do | TRAIN-001 |
| 8 | TRAIN-008 | Train Search UI | Story | 5 | High | ⬜ To Do | TRAIN-007, TRAIN-003 |
| 9 | TRAIN-009 | Booking Flow UI | Story | 5 | High | ⬜ To Do | TRAIN-008, TRAIN-005 |
| 10 | TRAIN-010 | My Bookings UI (View & Cancel) | Story | 3 | Medium | ⬜ To Do | TRAIN-009, TRAIN-006 |
| 11 | TRAIN-011 | Docker Compose for Local Dev | Task | 2 | Medium | ⬜ To Do | TRAIN-010 |

**Total Story Points: 42**

---

## Status Key

| Symbol | Meaning |
|--------|---------|
| ⬜ To Do | Not started |
| 🔄 In Progress | Currently being worked |
| 🔍 In Review | Done, under review / validation |
| ✅ Done | Accepted and complete |
| 🚫 Blocked | Waiting on a dependency |
