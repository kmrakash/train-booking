# TRAIN-010: My Bookings UI (View & Cancel)

## Summary
Build the My Bookings page where a logged-in user can see all their bookings (CONFIRMED and CANCELLED) and cancel a CONFIRMED booking.

## Type
Story

## Priority
Medium

## Story Points
3

## Epic
Sprint 1 — MVP

## Sprint
Sprint 1

---

## Description

Users need to manage their bookings after making them. This screen lists all bookings with their status and provides a Cancel button for CONFIRMED bookings.

---

## UI Layout

```
┌─────────────────────────────────────────────┐
│  [Header]                                    │
├─────────────────────────────────────────────┤
│                                             │
│  My Bookings                                │
│                                             │
│  ┌────────────────────────────────────┐     │
│  │ Booking #42                  CONFIRMED   │
│  │ Rajdhani Express (12301)           │     │
│  │ Delhi → Mumbai                     │     │
│  │ Date: 2026-11-01  |  Seat: 15      │     │
│  │ Booked at: 24 Oct 2026, 10:30 AM   │     │
│  │                      [ Cancel ]    │     │
│  └────────────────────────────────────┘     │
│                                             │
│  ┌────────────────────────────────────┐     │
│  │ Booking #41                 CANCELLED    │
│  │ Shatabdi Express (12302)           │     │
│  │ Chennai → Bangalore                │     │
│  │ Date: 2026-10-15  |  Seat: 8       │     │
│  │ Booked at: 20 Oct 2026, 09:00 AM   │     │
│  │            (no cancel button)      │     │
│  └────────────────────────────────────┘     │
│                                             │
│  (empty state)                              │
│  You have no bookings yet.                  │
│  [ Search Trains ]                          │
└─────────────────────────────────────────────┘
```

States to handle:
- **Loading** — fetching bookings
- **Has bookings** — list of booking cards
- **Empty** — "No bookings yet" with a link to search
- **Cancel loading** — specific card shows "Cancelling..." while request is in flight
- **Cancel success** — card's status badge updates to CANCELLED, Cancel button disappears
- **Cancel error** — error message shown inline on the card

---

## Acceptance Criteria

- [ ] `MyBookingsComponent` loads on `/my-bookings` (protected by AuthGuard)
- [ ] On load, calls `BookingService.getMyBookings()` which calls `GET /api/bookings`
- [ ] Each booking is shown as a card with: booking ID, train name/number, route, date, seat, status badge, booked-at datetime
- [ ] CONFIRMED bookings show a "Cancel" button; CANCELLED bookings do not
- [ ] Clicking "Cancel" on a booking calls `BookingService.cancelBooking(id)` which calls `DELETE /api/bookings/{id}`
- [ ] During cancellation, the "Cancel" button shows "Cancelling..." and is disabled to prevent double-clicks
- [ ] On successful cancellation, the card's status badge updates to CANCELLED and the Cancel button disappears (no full page reload)
- [ ] On cancellation error, an error message is shown on the card
- [ ] Empty state shows a friendly message and a link to `/search`
- [ ] Route is protected by AuthGuard

---

## Technical Notes

### BookingService — add getMyBookings and cancelBooking

```typescript
getMyBookings(): Observable<Booking[]> {
  return this.http.get<Booking[]>(`${environment.apiUrl}/bookings`);
}

cancelBooking(id: number): Observable<Booking> {
  return this.http.delete<Booking>(`${environment.apiUrl}/bookings/${id}`);
}
```

### Optimistic vs Pessimistic UI Update

For this sprint, use a **pessimistic update**: wait for the API response, then update the card's status in the component state. This is simpler and correct. Sprint 6 can introduce optimistic updates.

```typescript
// In the component, after successful cancel:
const booking = this.bookings.find(b => b.id === id);
if (booking) booking.status = 'CANCELLED';
```

### Status Badge Styling

Use a simple CSS class based on status:
```html
<span [class]="'status-badge status-' + booking.status.toLowerCase()">
  {{ booking.status }}
</span>
```
```css
.status-confirmed { color: green; }
.status-cancelled { color: red; text-decoration: line-through; }
```

---

## TDD Note

Write these tests FIRST:

**Unit test — `BookingService` (add to existing):**
1. `getMyBookings()` makes GET to `/api/bookings`
2. `cancelBooking(42)` makes DELETE to `/api/bookings/42`

**Component test — `MyBookingsComponent`:**
1. On load, renders a card for each booking returned by the service
2. CONFIRMED booking shows Cancel button; CANCELLED booking does not
3. After successful cancel, the card's status changes to CANCELLED and Cancel button disappears
4. Empty bookings array shows the empty state message and Search link

---

## Dependencies

- TRAIN-007 (routing, AuthGuard)
- TRAIN-006 (View & Cancel Booking API)
- TRAIN-009 (navigates to My Bookings from booking confirmation)

---

## Definition of Done

- [ ] My Bookings page works end-to-end with the real backend
- [ ] Cancel flow works without a page reload
- [ ] Empty state and error states are handled
- [ ] All component and service tests pass

---

## Validation Steps

```bash
# Navigate to http://localhost:4200/my-bookings (after making a booking in TRAIN-009)

# 1. Bookings load
# Expected: list of booking cards with correct info

# 2. Click Cancel on a CONFIRMED booking
# Expected: button shows "Cancelling...", then card updates to CANCELLED, button disappears

# 3. Navigate to /my-bookings when no bookings exist (new user)
# Expected: "You have no bookings yet" + "Search Trains" link

# 4. Click "Search Trains" from empty state
# Expected: navigates to /search

# 5. Run tests
cd apps/frontend && ng test --watch=false
# Expected: all tests pass
```
