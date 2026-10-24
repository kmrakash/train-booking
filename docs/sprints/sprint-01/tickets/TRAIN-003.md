# TRAIN-003: Train Entity & Search API

## Summary
Create the `Train` JPA entity, its repository, service, and a REST endpoint that lets a user search for trains by source, destination, and journey date.

## Type
Story

## Priority
High

## Story Points
5

## Epic
Sprint 1 — MVP

## Sprint
Sprint 1

---

## Description

Users need to find trains before they can book one. This ticket delivers the first public-facing API:

```
GET /api/trains/search?source=Delhi&destination=Mumbai&date=2026-11-01
```

The response returns a list of trains that run on the given route, along with the number of available seats for that date (total seats minus confirmed bookings on that date).

---

## API Contract

### Endpoint
```
GET /api/trains/search
```

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| source | String | Yes | Departure city (case-insensitive) |
| destination | String | Yes | Arrival city (case-insensitive) |
| date | LocalDate (YYYY-MM-DD) | Yes | Journey date |

### Success Response — 200 OK
```json
[
  {
    "id": 1,
    "trainNumber": "12301",
    "trainName": "Rajdhani Express",
    "source": "Delhi",
    "destination": "Mumbai",
    "departureTime": "16:00",
    "arrivalTime": "08:00",
    "totalSeats": 100,
    "availableSeats": 97
  }
]
```

### Empty Result — 200 OK with empty array
```json
[]
```

### Validation Error — 400 Bad Request
```json
{
  "error": "source, destination, and date are required"
}
```

---

## Acceptance Criteria

- [ ] `Train` JPA entity maps to the `trains` table from TRAIN-002
- [ ] `TrainRepository` extends `JpaRepository<Train, Long>`
- [ ] `TrainRepository` has a method to find trains by source and destination (case-insensitive)
- [ ] `TrainService` calculates available seats: `totalSeats - confirmedBookingsOnDate`
- [ ] `TrainController` exposes `GET /api/trains/search` with the query params above
- [ ] Missing any required parameter returns 400
- [ ] Source/destination matching is case-insensitive (`delhi` matches `Delhi`)
- [ ] `availableSeats` is accurate — it counts only `CONFIRMED` bookings, not `CANCELLED`
- [ ] Response is a JSON array (empty array if no trains found, never null)

---

## Technical Notes

### Layer Structure (follow this for every feature)
```
TrainController   ← handles HTTP, validates input, calls service
    ↓
TrainService      ← business logic, no HTTP knowledge
    ↓
TrainRepository   ← data access only, extends JpaRepository
    ↓
Train             ← JPA entity, maps to trains table
```

### Available Seats Query
Available seats requires a subquery or JOIN to the bookings table. Write a custom JPQL query in the repository:

```java
@Query("SELECT COUNT(b) FROM Booking b WHERE b.train.id = :trainId AND b.journeyDate = :date AND b.status = 'CONFIRMED'")
long countConfirmedBookings(@Param("trainId") Long trainId, @Param("date") LocalDate date);
```

`availableSeats = train.totalSeats - confirmedBookingsCount`

### Response DTO
Do NOT return the JPA entity directly from the controller. Create a `TrainSearchResponse` DTO:
```java
public record TrainSearchResponse(
    Long id,
    String trainNumber,
    String trainName,
    String source,
    String destination,
    String departureTime,
    String arrivalTime,
    int totalSeats,
    int availableSeats
) {}
```

### CORS
Add `@CrossOrigin(origins = "http://localhost:4200")` to the controller for now (Sprint 3 will handle this properly via a global config).

---

## TDD Note

Write these tests FIRST, confirm they fail, then implement:

**Unit test — `TrainServiceTest`:**
1. When 2 bookings exist for a train on a date, `availableSeats` = `totalSeats - 2`
2. When 0 bookings exist, `availableSeats` = `totalSeats`
3. `CANCELLED` bookings do not reduce available seats

**Integration test — `TrainControllerTest`:**
1. `GET /api/trains/search?source=Delhi&destination=Mumbai&date=2026-11-01` returns 200 with a list
2. `GET /api/trains/search?source=Delhi&destination=Mumbai` (missing date) returns 400
3. `GET /api/trains/search?source=nowhere&destination=nowhere&date=2026-11-01` returns 200 with empty array

---

## Dependencies

- TRAIN-002 must be complete (DB schema with trains table and seed data)

---

## Definition of Done

- [ ] All three layers (entity, service, controller) implemented
- [ ] All unit and integration tests pass
- [ ] API tested manually with curl or Postman and returns correct JSON
- [ ] `availableSeats` decreases correctly when a booking exists in the DB

---

## Validation Steps

```bash
# Start the app (PostgreSQL must be running)
cd apps/backend && ./mvnw spring-boot:run

# 1. Search for a valid route
curl "http://localhost:8080/api/trains/search?source=Delhi&destination=Mumbai&date=2026-11-01"
# Expected: JSON array with at least 1 train, availableSeats = totalSeats (no bookings yet)

# 2. Case-insensitive search
curl "http://localhost:8080/api/trains/search?source=delhi&destination=mumbai&date=2026-11-01"
# Expected: same result as above

# 3. No trains on this route
curl "http://localhost:8080/api/trains/search?source=Delhi&destination=Chennai&date=2026-11-01"
# Expected: []

# 4. Missing parameter
curl "http://localhost:8080/api/trains/search?source=Delhi&destination=Mumbai"
# Expected: 400 Bad Request with error message

# 5. Run tests
./mvnw test -pl apps/backend
# Expected: BUILD SUCCESS, all tests pass
```
