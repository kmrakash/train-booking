# TRAIN-008: Train Search UI

## Summary
Build the train search page where a user enters source, destination, and date, submits the form, and sees a list of matching trains with available seats.

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

This is the first screen a logged-in user sees. The flow is:
1. User fills in source, destination, and date
2. Clicks Search
3. The app calls `GET /api/trains/search?source=&destination=&date=`
4. Results display as cards — each card shows train info, available seats, and a "Book" button
5. Clicking "Book" navigates to `/book/:trainId` passing the date as a query param

---

## UI Layout

```
┌─────────────────────────────────────────────┐
│  [Header]                                    │
├─────────────────────────────────────────────┤
│                                             │
│  Search Trains                              │
│  ┌─────────┐  ┌─────────────┐  ┌────────┐  │
│  │ Source  │  │ Destination │  │  Date  │  │
│  └─────────┘  └─────────────┘  └────────┘  │
│                          [ Search ]         │
│                                             │
│  ── Results ──                              │
│                                             │
│  ┌────────────────────────────────────┐     │
│  │ Rajdhani Express  (12301)          │     │
│  │ Delhi → Mumbai                     │     │
│  │ Dep: 16:00   Arr: 08:00            │     │
│  │ Available: 97 seats                │     │
│  │                        [ Book ]    │     │
│  └────────────────────────────────────┘     │
│                                             │
│  (repeat for each result)                  │
└─────────────────────────────────────────────┘
```

States to handle:
- **Initial** — empty results, no message
- **Loading** — show a "Searching..." indicator
- **Results** — show train cards
- **No results** — show "No trains found on this route"
- **Error** — show "Something went wrong, please try again"

---

## Acceptance Criteria

- [ ] `TrainSearchComponent` has a form with three fields: source (text), destination (text), date (date picker)
- [ ] All three fields are required — form is invalid and button is disabled if any field is empty
- [ ] On submit, calls `TrainService.searchTrains(source, destination, date)`
- [ ] `TrainService` makes `GET /api/trains/search?source=&destination=&date=` and returns `Observable<Train[]>`
- [ ] While the request is in flight, a loading indicator is shown and the Search button is disabled
- [ ] Results are displayed as a list of cards with: train name, number, source→destination, departure/arrival times, available seats
- [ ] Each card has a "Book" button that navigates to `/book/:trainId?date=2026-11-01`
- [ ] "Book" button is disabled if `availableSeats === 0` and shows "Full" text instead
- [ ] Empty results show a friendly message
- [ ] HTTP errors show an error message (do not crash the app)
- [ ] Route is protected by `AuthGuard`

---

## Technical Notes

### TrainService

```typescript
// core/services/train.service.ts
export class TrainService {
  private apiUrl = inject(environment).apiUrl; // or inject directly

  searchTrains(source: string, destination: string, date: string): Observable<Train[]> {
    return this.http.get<Train[]>(`${environment.apiUrl}/trains/search`, {
      params: { source, destination, date }
    });
  }
}
```

### Train Interface

```typescript
// core/models/train.model.ts
export interface Train {
  id: number;
  trainNumber: string;
  trainName: string;
  source: string;
  destination: string;
  departureTime: string;
  arrivalTime: string;
  totalSeats: number;
  availableSeats: number;
}
```

### Reactive Form vs Template Form

Use Angular Reactive Forms (`FormGroup`, `FormControl`) — they make validation and state management cleaner. This is a learning project so learn the right way.

### Navigation to Booking

```typescript
this.router.navigate(['/book', train.id], { queryParams: { date: this.searchForm.value.date } });
```

---

## TDD Note

Write these tests FIRST:

**Unit test — `TrainService`:**
1. `searchTrains()` makes a GET request to the correct URL with the correct query params
2. Returns an `Observable<Train[]>`

**Component test — `TrainSearchComponent`:**
1. Search button is disabled when any field is empty
2. Submitting the form calls `TrainService.searchTrains()` with correct arguments
3. On successful response, train cards are rendered
4. On empty response, "No trains found" message is shown
5. On HTTP error, error message is shown

---

## Dependencies

- TRAIN-007 (Angular routing, AuthGuard, AuthService, interceptor)
- TRAIN-003 (Search API must be running to test end-to-end)

---

## Definition of Done

- [ ] Search form works end-to-end with the real backend
- [ ] All component and service tests pass
- [ ] All three UI states (loading, results, no results) are tested manually
- [ ] "Book" button navigates to `/book/:trainId?date=...`

---

## Validation Steps

```bash
# Prerequisites: backend running, user registered and logged in

# 1. Start frontend
cd apps/frontend && ng serve

# Open http://localhost:4200
# Expected: redirected to /login → log in → redirected to /search

# 2. Enter: Source=Delhi, Destination=Mumbai, Date=2026-11-01 → click Search
# Expected: loading indicator, then list of trains

# 3. Enter: Source=Delhi, Destination=Chennai → click Search
# Expected: "No trains found on this route"

# 4. Click Book on a train result
# Expected: navigated to /book/1?date=2026-11-01

# 5. Leave Source empty → try to click Search
# Expected: button is disabled or form validation error shown

# 6. Run tests
ng test --watch=false
# Expected: all tests pass
```
