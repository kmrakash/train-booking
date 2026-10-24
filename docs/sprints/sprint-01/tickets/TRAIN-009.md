# TRAIN-009: Booking Flow UI

## Summary
Build the booking page where a user selects a seat number for a chosen train and date, confirms the booking, and sees a confirmation screen.

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

After clicking "Book" on the search results, the user lands on a booking page pre-filled with the train and date. They choose a seat number, confirm, and either see a success confirmation or an error (e.g., seat already taken).

---

## UI Layout

```
┌─────────────────────────────────────────────┐
│  [Header]                                    │
├─────────────────────────────────────────────┤
│                                             │
│  Book Your Seat                             │
│                                             │
│  Train:    Rajdhani Express (12301)         │
│  Route:    Delhi → Mumbai                   │
│  Date:     2026-11-01                       │
│  Available: 97 seats                        │
│                                             │
│  Seat Number: [ _____ ]  (1 to 100)        │
│                                             │
│            [ Confirm Booking ]              │
│                                             │
├─────────────────────────────────────────────┤
│  SUCCESS STATE:                             │
│  ✓ Booking Confirmed!                       │
│  Booking ID: #42                            │
│  Seat: 15  |  Status: CONFIRMED             │
│  [ View My Bookings ]                       │
├─────────────────────────────────────────────┤
│  ERROR STATE:                               │
│  ✗ Seat 15 is already taken. Try another.  │
└─────────────────────────────────────────────┘
```

States to handle:
- **Initial** — pre-filled train/date info, seat number input
- **Loading** — "Booking..." indicator, button disabled
- **Success** — confirmation message with booking ID, link to My Bookings
- **Error (409)** — "Seat already taken" message, form stays open so user can try another seat
- **Error (other)** — generic error message

---

## Acceptance Criteria

- [ ] `BookingComponent` reads `trainId` from the route param and `date` from query params
- [ ] On load, component calls `TrainService.getTrainById(trainId)` to display train details
- [ ] Seat number input is a number field, min=1, max=train.totalSeats
- [ ] "Confirm Booking" button is disabled if seat input is empty or out of range
- [ ] On submit, calls `BookingService.createBooking(trainId, date, seatNumber)`
- [ ] `BookingService` makes `POST /api/bookings` with the correct body
- [ ] On 201 success, shows confirmation with booking ID and seat number
- [ ] On 409 conflict, shows "Seat already taken" — form stays visible so user can try again
- [ ] On other error, shows generic error message
- [ ] "View My Bookings" button on success screen navigates to `/my-bookings`
- [ ] Route is protected by `AuthGuard`

---

## Technical Notes

### Reading route params

```typescript
export class BookingComponent {
  private route = inject(ActivatedRoute);
  
  trainId = this.route.snapshot.paramMap.get('trainId');
  date = this.route.snapshot.queryParamMap.get('date');
}
```

### TrainService — add getTrainById

```typescript
getTrainById(id: number): Observable<Train> {
  return this.http.get<Train>(`${environment.apiUrl}/trains/${id}`);
}
```

> **Note:** This requires a new backend endpoint `GET /api/trains/{id}`. Add it to the backend as part of this ticket (it is a small addition to `TrainController` and `TrainService`).

### BookingService

```typescript
// core/services/booking.service.ts
export class BookingService {
  createBooking(trainId: number, journeyDate: string, seatNumber: number): Observable<Booking> {
    return this.http.post<Booking>(`${environment.apiUrl}/bookings`, {
      trainId, journeyDate, seatNumber
    });
  }
}
```

### Booking Interface

```typescript
export interface Booking {
  id: number;
  trainId: number;
  trainName: string;
  trainNumber: string;
  source: string;
  destination: string;
  journeyDate: string;
  seatNumber: number;
  status: string;
  bookedAt: string;
}
```

### 409 Error Handling

```typescript
this.bookingService.createBooking(...).subscribe({
  next: (booking) => { this.confirmedBooking = booking; },
  error: (err) => {
    if (err.status === 409) {
      this.errorMessage = 'Seat already taken. Please try a different seat number.';
    } else {
      this.errorMessage = 'Something went wrong. Please try again.';
    }
  }
});
```

---

## Backend Addition Required

Add `GET /api/trains/{id}` to `TrainController` in the backend. This is a small change — a single method that returns one train by ID (404 if not found). Do this as part of this ticket.

---

## TDD Note

Write these tests FIRST:

**Unit test — `BookingService`:**
1. `createBooking()` makes a POST to `/api/bookings` with the correct body

**Component test — `BookingComponent`:**
1. Confirm button is disabled when seat number is empty
2. On successful booking, confirmation message is displayed with booking ID
3. On 409 error, "Seat already taken" message is shown and the form is still visible
4. On other error, generic error message is shown

---

## Dependencies

- TRAIN-007 (routing, auth)
- TRAIN-008 (search page — navigates to booking)
- TRAIN-005 (Create Booking API)

---

## Definition of Done

- [ ] Booking flow works end-to-end with the real backend
- [ ] 409 conflict error handled gracefully
- [ ] All component tests pass
- [ ] Backend `GET /api/trains/{id}` endpoint added and tested

---

## Validation Steps

```bash
# From search results, click "Book" on Rajdhani Express for 2026-11-01
# Expected: /book/1?date=2026-11-01 loads with train details pre-filled

# Enter seat number 15 → click Confirm Booking
# Expected: success confirmation, "Booking ID: #42, Seat: 15, CONFIRMED"

# Go back to search, click Book again for same train/date
# Enter seat number 15 again → click Confirm Booking
# Expected: "Seat already taken" error, form stays visible

# Enter seat number 16 → click Confirm Booking
# Expected: success for seat 16

# Click "View My Bookings"
# Expected: navigated to /my-bookings
```
