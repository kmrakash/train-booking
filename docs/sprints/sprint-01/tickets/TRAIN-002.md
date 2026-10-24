# TRAIN-002: Database Schema & Flyway Migrations

## Summary
Design the initial PostgreSQL schema for trains, users, and bookings. Implement it as a Flyway migration so the database is version-controlled and reproducible.

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

Every feature in this sprint reads from or writes to the database. Getting the schema right first prevents painful data migrations mid-sprint.

This ticket covers three tables:
- `trains` — the train catalog (static data for MVP)
- `users` — registered users
- `bookings` — the link between a user and a seat on a train

The schema must be created via Flyway so it is applied automatically when the app starts. No manual `CREATE TABLE` scripts — the migration file IS the schema.

---

## Schema Design

### `trains`
```sql
id            BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY
train_number  VARCHAR(20) NOT NULL UNIQUE          -- e.g. "12345"
train_name    VARCHAR(100) NOT NULL                -- e.g. "Rajdhani Express"
source        VARCHAR(100) NOT NULL                -- departure city
destination   VARCHAR(100) NOT NULL                -- arrival city
departure_time TIME NOT NULL                       -- e.g. 06:00
arrival_time   TIME NOT NULL
total_seats   INT NOT NULL
```

### `users`
```sql
id            BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY
name          VARCHAR(100) NOT NULL
email         VARCHAR(150) NOT NULL UNIQUE
password_hash VARCHAR(255) NOT NULL
created_at    TIMESTAMP NOT NULL DEFAULT NOW()
```

### `bookings`
```sql
id            BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY
user_id       BIGINT NOT NULL REFERENCES users(id)
train_id      BIGINT NOT NULL REFERENCES trains(id)
journey_date  DATE NOT NULL
seat_number   INT NOT NULL
status        VARCHAR(20) NOT NULL DEFAULT 'CONFIRMED'  -- CONFIRMED | CANCELLED
booked_at     TIMESTAMP NOT NULL DEFAULT NOW()

CONSTRAINT uq_seat UNIQUE (train_id, journey_date, seat_number)
```

---

## Acceptance Criteria

- [ ] Flyway migration file `V1__initial_schema.sql` exists at `apps/backend/src/main/resources/db/migration/`
- [ ] Migration creates all three tables with the columns defined above
- [ ] Foreign key constraints are present (`bookings.user_id → users.id`, `bookings.train_id → trains.id`)
- [ ] Unique constraint on `(train_id, journey_date, seat_number)` prevents double-booking at DB level
- [ ] A seed data migration `V2__seed_trains.sql` inserts at least 5 trains so searches return results
- [ ] Application starts and Flyway runs both migrations successfully (check logs for `Successfully applied 2 migrations`)
- [ ] `application.properties` has correct datasource config pointing to the local PostgreSQL container

---

## Technical Notes

### Flyway file naming
Flyway migrations follow this pattern:
```
V{version}__{description}.sql
```
- `V1__initial_schema.sql` — creates tables
- `V2__seed_trains.sql` — inserts sample data

### `application.properties` datasource config
```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/trainbooking
spring.datasource.username=trainuser
spring.datasource.password=trainpass
spring.datasource.driver-class-name=org.postgresql.Driver

spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=true

spring.flyway.enabled=true
spring.flyway.locations=classpath:db/migration
```

`ddl-auto=validate` means Hibernate will check that the JPA entities match the DB schema but will NOT create/alter tables itself — Flyway owns the schema.

### Sample seed trains (use in V2)
```sql
INSERT INTO trains (train_number, train_name, source, destination, departure_time, arrival_time, total_seats) VALUES
('12301', 'Rajdhani Express',   'Delhi',   'Mumbai',   '16:00', '08:00', 100),
('12302', 'Shatabdi Express',   'Chennai', 'Bangalore','06:00', '11:00',  80),
('12303', 'Duronto Express',    'Kolkata', 'Delhi',    '20:00', '10:00', 120),
('12304', 'Garib Rath',         'Mumbai',  'Goa',      '23:00', '07:00',  60),
('12305', 'Vande Bharat Express','Delhi',  'Varanasi', '06:00', '14:00',  90);
```

---

## TDD Note

Write a Spring Boot integration test that:
1. Starts the application context with an embedded H2 (or Testcontainers PostgreSQL)
2. Asserts that the `trains` table has 5 rows after migrations run
3. Asserts that inserting a duplicate `(train_id, journey_date, seat_number)` throws a `DataIntegrityViolationException`

Write the test FIRST, watch it fail, then write the migration to make it pass.

---

## Dependencies

- TRAIN-001 must be complete (Spring Boot project must exist)

---

## Definition of Done

- [ ] `V1__initial_schema.sql` and `V2__seed_trains.sql` exist in the correct Flyway path
- [ ] `./mvnw spring-boot:run` logs `Successfully applied 2 migrations` on first run
- [ ] Schema integration test passes
- [ ] `psql -U trainuser trainbooking -c "\dt"` shows `trains`, `users`, `bookings` tables

---

## Validation Steps

```bash
# Start PostgreSQL (you will have a docker-compose in TRAIN-011, for now run manually)
docker run -d \
  --name traindb \
  -e POSTGRES_DB=trainbooking \
  -e POSTGRES_USER=trainuser \
  -e POSTGRES_PASSWORD=trainpass \
  -p 5432:5432 \
  postgres:16

# Start the Spring Boot app
cd apps/backend
./mvnw spring-boot:run

# Watch the logs — you should see:
# Flyway: Successfully applied 2 migrations to schema "public"

# Connect and verify
docker exec -it traindb psql -U trainuser trainbooking

# In psql:
\dt
-- Expected: bookings, trains, users

SELECT count(*) FROM trains;
-- Expected: 5

SELECT * FROM trains;
-- Expected: 5 rows with train data

\d bookings
-- Expected: columns including the UNIQUE constraint on (train_id, journey_date, seat_number)
```
