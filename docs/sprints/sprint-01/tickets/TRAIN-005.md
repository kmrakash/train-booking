# TRAIN-005: Create Booking API

## Summary
Implement the endpoint that lets an authenticated user book a specific seat on a train for a journey date. The system must check seat availability before confirming.

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

This is the core feature of the entire system. A logged-in user selects a train, a seat number, and a journey date, and the system creates a confirmed booking — but only if that seat is not already taken.

> **Important:** In this sprint we do a simple availability check (read seats, if available write booking). We are deliberately NOT solving the concurrent booking problem here. That is the explicit bottleneck we will hit in Sprint 2 when two users try to book the same seat at the same moment. Sprint 2 introduces transactions and row-level locking to fix this. For now, the naive approach is intentional.

---

## API Contract

### Endpoint
```
POST /api/bookings
Authorization: Basic base64(email:password)
Content-Type: application/json
```

### Request Body
```json
{
  "trainId": 1,
  "journeyDate": "2026-11-01",
  "seatNumber": 15
}
```

### Success — 201 Created
```json
{
  "id": 42,
  "trainId": 1,
  "trainName": "Rajdhani Express",
  "trainNumber": "12301",
  "source": "Delhi",
  "destination": "Mumbai",
  "journeyDate": "2026-11-01",
  "seatNumber": 15,
  "status": "CONFIRMED",
  "bookedAt": "2026-10-24T10:30:00"
}
```

### Seat already taken — 409 Conflict
```json
{
  "error": "Seat 15 on train 12301 for 2026-11-01 is already booked"
}
```

### Train not found — 404 Not Found
```json
{
  "error": "Train not found"
}
```

### Seat number out of range — 400 Bad Request
```json
{
  "error": "Seat number must be between 1 and {totalSeats}"
}
```

### Missing fields — 400 Bad Request
```json
{
  "error": "trainId, journeyDate, and seatNumber are required"
}
```

---

## Acceptance Criteria

- [ ] `Booking` JPA entity maps to the `bookings` table from TRAIN-002
- [ ] `BookingRepository` extends `JpaRepository<Booking, Long>`
- [ ] `POST /api/bookings` requires authentication (401 if no auth)
- [ ] Booking is created with `status = CONFIRMED`
- [ ] The authenticated user from the Security context is automatically set as the booking owner (client does NOT send user ID)
- [ ] Validates seat is not already booked for that (train, date, seat) combination
- [ ] Validates seat number is between 1 and train's `totalSeats`
- [ ] Validates train exists
- [ ] Returns full booking details including train information in the response
- [ ] Attempting to book the same seat twice returns 409

---

## Technical Notes

### Getting the current user from Spring Security

The logged-in user is available from the security context — do not accept `userId` from the request body:

```java
@PostMapping
public ResponseEntity<BookingResponse> createBooking(
    @RequestBody BookingRequest request,
    @AuthenticationPrincipal UserDetails userDetails) {
    // userDetails.getUsername() is the email
    // look up the user entity by email
}
```

### Availability Check (naive — intentional for Sprint 1)

```java
boolean seatTaken = bookingRepository.existsByTrainIdAndJourneyDateAndSeatNumberAndStatus(
    trainId, journeyDate, seatNumber, "CONFIRMED"
);
if (seatTaken) throw new SeatAlreadyBookedException(...);
```

This is a race condition waiting to happen (Sprint 2's problem). Write it this way on purpose.

### Response DTO

```java
public record BookingResponse(
    Long id,
    Long trainId,
    String trainName,
    String trainNumber,
    String source,
    String destination,
    LocalDate journeyDate,
    int seatNumber,
    String status,
    LocalDateTime bookedAt
) {}
```

---

## TDD Note

Write these tests FIRST:

**Unit test — `BookingServiceTest`:**
1. Booking succeeds when seat is available → returns `BookingResponse` with `status = CONFIRMED`
2. Booking fails when seat is already taken → throws `SeatAlreadyBookedException`
3. Booking fails when seat number exceeds `totalSeats` → throws `InvalidSeatException`
4. Booking fails when train ID does not exist → throws `TrainNotFoundException`

**Integration test — `BookingControllerTest`:**
1. `POST /api/bookings` with valid body and auth → 201 with booking JSON
2. `POST /api/bookings` without auth → 401
3. `POST /api/bookings` for an already-booked seat → 409
4. `POST /api/bookings` for a non-existent train → 404

---

## Dependencies

- TRAIN-003 (Train entity and repository)
- TRAIN-004 (User entity and Spring Security config)

---

## Definition of Done

- [ ] Booking entity, repository, service, controller all implemented
- [ ] All unit and integration tests pass
- [ ] Double-booking the same seat returns 409 (tested manually)
- [ ] Unauthenticated request returns 401

---

## Validation Steps

```bash
# Prerequisites: app running, user registered from TRAIN-004

# 1. Book a seat
curl -X POST http://localhost:8080/api/bookings \
  -H "Content-Type: application/json" \
  -u "akash@example.com:secret123" \
  -d '{"trainId":1,"journeyDate":"2026-11-01","seatNumber":15}'
# Expected: 201 with booking JSON, status=CONFIRMED

# 2. Try to book the same seat again
curl -X POST http://localhost:8080/api/bookings \
  -H "Content-Type: application/json" \
  -u "akash@example.com:secret123" \
  -d '{"trainId":1,"journeyDate":"2026-11-01","seatNumber":15}'
# Expected: 409 {"error":"Seat 15 on train 12301 for 2026-11-01 is already booked"}

# 3. Book a seat without auth
curl -X POST http://localhost:8080/api/bookings \
  -H "Content-Type: application/json" \
  -d '{"trainId":1,"journeyDate":"2026-11-01","seatNumber":20}'
# Expected: 401 Unauthorized

# 4. Book a non-existent seat number
curl -X POST http://localhost:8080/api/bookings \
  -H "Content-Type: application/json" \
  -u "akash@example.com:secret123" \
  -d '{"trainId":1,"journeyDate":"2026-11-01","seatNumber":999}'
# Expected: 400 {"error":"Seat number must be between 1 and 100"}

# 5. Verify available seats decreased in the search API
curl "http://localhost:8080/api/trains/search?source=Delhi&destination=Mumbai&date=2026-11-01" \
  -u "akash@example.com:secret123"
# Expected: availableSeats is now totalSeats - 1 (was 100, now 99)

# 6. Run tests
./mvnw test
# Expected: BUILD SUCCESS
```
