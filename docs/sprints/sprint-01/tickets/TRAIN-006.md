# TRAIN-006: View & Cancel Booking API

## Summary
Implement endpoints for an authenticated user to list their bookings and cancel a specific booking. A user can only see and cancel their own bookings.

## Type
Story

## Priority
High

## Story Points
3

## Epic
Sprint 1 — MVP

## Sprint
Sprint 1

---

## Description

After booking a seat, users need to manage their bookings. This ticket adds:
1. **List my bookings** — returns all bookings for the logged-in user
2. **Cancel a booking** — changes a booking's status from `CONFIRMED` to `CANCELLED`

Cancellation is a soft delete — we never delete a booking row. We change `status` to `CANCELLED` so we have a history. This also means the freed seat becomes available for others.

---

## API Contract

### List My Bookings

```
GET /api/bookings
Authorization: Basic base64(email:password)
```

**Success — 200 OK:**
```json
[
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
  },
  {
    "id": 43,
    "trainId": 2,
    "trainName": "Shatabdi Express",
    ...
    "status": "CANCELLED",
    ...
  }
]
```

Returns all bookings (CONFIRMED and CANCELLED) for the current user. Empty array if none.

---

### Cancel a Booking

```
DELETE /api/bookings/{id}
Authorization: Basic base64(email:password)
```

**Success — 200 OK:**
```json
{
  "id": 42,
  "status": "CANCELLED",
  ...all other fields...
}
```

**Booking not found — 404 Not Found:**
```json
{
  "error": "Booking not found"
}
```

**Booking belongs to another user — 403 Forbidden:**
```json
{
  "error": "You are not allowed to cancel this booking"
}
```

**Booking already cancelled — 400 Bad Request:**
```json
{
  "error": "Booking is already cancelled"
}
```

---

## Acceptance Criteria

- [ ] `GET /api/bookings` returns all bookings (any status) for the authenticated user only
- [ ] `GET /api/bookings` returns empty array (not 404) when user has no bookings
- [ ] `DELETE /api/bookings/{id}` sets `status = CANCELLED`
- [ ] A user cannot cancel another user's booking (403)
- [ ] Cancelling an already-cancelled booking returns 400
- [ ] Cancelling a non-existent booking ID returns 404
- [ ] Both endpoints require authentication (401 if missing)
- [ ] `GET /api/bookings` includes bookings of all statuses (so user can see their history)
- [ ] After a successful cancel, the seat becomes available again — verify via the search API

---

## Technical Notes

### Ownership Check

When cancelling, always check the booking belongs to the authenticated user:

```java
if (!booking.getUser().getEmail().equals(currentUserEmail)) {
    throw new ForbiddenException("You are not allowed to cancel this booking");
}
```

### Soft Delete Pattern

```java
booking.setStatus("CANCELLED");
bookingRepository.save(booking);
```

Never call `bookingRepository.delete(booking)`.

### Repository Method

```java
List<Booking> findByUserId(Long userId);
```

---

## TDD Note

Write these tests FIRST:

**Unit test — `BookingServiceTest` (add to existing test class):**
1. `getMyBookings()` returns only bookings belonging to the current user
2. `cancelBooking()` sets status to `CANCELLED` when user owns the booking
3. `cancelBooking()` throws `ForbiddenException` when booking belongs to another user
4. `cancelBooking()` throws `AlreadyCancelledException` when booking is already cancelled
5. `cancelBooking()` throws `BookingNotFoundException` for unknown booking ID

**Integration test — `BookingControllerTest` (add to existing):**
1. `GET /api/bookings` returns only current user's bookings (not another user's)
2. `DELETE /api/bookings/{id}` with valid owned booking → 200 with CANCELLED status
3. `DELETE /api/bookings/{id}` for another user's booking → 403
4. `DELETE /api/bookings/{id}` for already-cancelled booking → 400
5. `DELETE /api/bookings/99999` → 404

---

## Dependencies

- TRAIN-005 (Booking entity and repository)

---

## Definition of Done

- [ ] Both endpoints implemented
- [ ] All unit and integration tests pass
- [ ] Verified manually: cancel a booking, then search shows that seat as available again
- [ ] User isolation verified: User A cannot see or cancel User B's bookings

---

## Validation Steps

```bash
# Prerequisites: app running, user registered, booking exists (from TRAIN-005)

# 1. List bookings
curl http://localhost:8080/api/bookings \
  -u "akash@example.com:secret123"
# Expected: 200 array with at least 1 booking, status=CONFIRMED

# 2. Cancel the booking (replace 42 with actual booking ID)
curl -X DELETE http://localhost:8080/api/bookings/42 \
  -u "akash@example.com:secret123"
# Expected: 200 with booking JSON, status=CANCELLED

# 3. List bookings again — cancelled booking should appear in history
curl http://localhost:8080/api/bookings \
  -u "akash@example.com:secret123"
# Expected: 200 array, booking 42 now shows status=CANCELLED

# 4. Verify seat is available again in search
curl "http://localhost:8080/api/trains/search?source=Delhi&destination=Mumbai&date=2026-11-01" \
  -u "akash@example.com:secret123"
# Expected: availableSeats back to original count

# 5. Cancel same booking again (already cancelled)
curl -X DELETE http://localhost:8080/api/bookings/42 \
  -u "akash@example.com:secret123"
# Expected: 400 {"error":"Booking is already cancelled"}

# 6. Register a second user and try to cancel user 1's booking
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"name":"Other User","email":"other@example.com","password":"pass123"}'

curl -X DELETE http://localhost:8080/api/bookings/42 \
  -u "other@example.com:pass123"
# Expected: 403 Forbidden

# 7. Run tests
./mvnw test
# Expected: BUILD SUCCESS
```
