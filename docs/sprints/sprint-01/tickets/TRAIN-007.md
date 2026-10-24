# TRAIN-007: Angular App Setup & Routing

## Summary
Configure the Angular 17+ application with a routing structure, shared layout (header/nav), an HTTP interceptor for Basic Auth, and a service base for communicating with the backend API.

## Type
Task

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

Before building individual screens, the Angular app needs a solid foundation:
- **Routing** — navigate between search, booking, and my-bookings pages
- **Shared layout** — a header with nav links present on every page
- **Auth service** — stores the logged-in user's credentials in memory and attaches them to every API request via an HTTP interceptor
- **HTTP interceptor** — adds the `Authorization: Basic ...` header to every outbound request automatically, so individual services don't have to worry about it
- **API base URL** — one place to configure the backend URL

---

## Route Structure

| Route | Component | Auth Required |
|-------|-----------|--------------|
| `/` | Redirects to `/search` | No |
| `/login` | `LoginComponent` | No |
| `/register` | `RegisterComponent` | No |
| `/search` | `TrainSearchComponent` | Yes |
| `/book/:trainId` | `BookingComponent` | Yes |
| `/my-bookings` | `MyBookingsComponent` | Yes |

Auth-required routes redirect to `/login` if no user is in the auth service.

---

## Acceptance Criteria

- [ ] `app.routes.ts` defines all routes listed above
- [ ] Routes marked "Auth Required" redirect to `/login` when user is not authenticated (use an `AuthGuard`)
- [ ] `AuthService` stores `{ id, name, email, password }` in memory (no localStorage for now)
- [ ] `AuthService.isLoggedIn()` returns `true` when a user is stored
- [ ] `AuthService.logout()` clears the stored user
- [ ] `BasicAuthInterceptor` adds `Authorization: Basic base64(email:password)` to every request when a user is logged in
- [ ] `BasicAuthInterceptor` is registered as a functional interceptor in `app.config.ts`
- [ ] `AppComponent` renders a `<app-header>` above the `<router-outlet>`
- [ ] `HeaderComponent` shows navigation links: Search | My Bookings | Logout (or Login/Register when not authenticated)
- [ ] `environment.ts` has `apiUrl: 'http://localhost:8080/api'` (and `environment.prod.ts` placeholder)
- [ ] Angular app starts with `ng serve` and routes work in the browser

---

## Technical Notes

### Functional HTTP Interceptor (Angular 17+ style)

```typescript
// auth.interceptor.ts
export const basicAuthInterceptor: HttpInterceptorFn = (req, next) => {
  const auth = inject(AuthService);
  if (auth.isLoggedIn()) {
    const credentials = btoa(`${auth.currentUser()!.email}:${auth.currentUser()!.password}`);
    const authReq = req.clone({
      headers: req.headers.set('Authorization', `Basic ${credentials}`)
    });
    return next(authReq);
  }
  return next(req);
};
```

Register in `app.config.ts`:
```typescript
provideHttpClient(withInterceptors([basicAuthInterceptor]))
```

### AuthGuard (functional guard, Angular 17+ style)

```typescript
// auth.guard.ts
export const authGuard: CanActivateFn = () => {
  const auth = inject(AuthService);
  const router = inject(Router);
  if (auth.isLoggedIn()) return true;
  return router.createUrlTree(['/login']);
};
```

### Folder Structure

```
src/app/
  core/
    services/
      auth.service.ts
    interceptors/
      auth.interceptor.ts
    guards/
      auth.guard.ts
  shared/
    components/
      header/
        header.component.ts
  features/
    auth/
      login/login.component.ts
      register/register.component.ts
    search/
      train-search/train-search.component.ts
    booking/
      booking/booking.component.ts
    my-bookings/
      my-bookings/my-bookings.component.ts
```

---

## TDD Note

Write these tests FIRST:

**Unit test — `AuthService`:**
1. `isLoggedIn()` returns false initially
2. After `setUser(...)`, `isLoggedIn()` returns true
3. After `logout()`, `isLoggedIn()` returns false again

**Unit test — `AuthGuard`:**
1. Returns `true` when user is logged in
2. Returns a `UrlTree` to `/login` when user is not logged in

---

## Dependencies

- TRAIN-001 (Angular app must exist)

---

## Definition of Done

- [ ] `ng serve` starts with no errors
- [ ] Navigating to `/search` without login redirects to `/login`
- [ ] After logging in, navigating to `/search` shows the (empty) search page
- [ ] Header shows correct nav links based on auth state
- [ ] All unit tests pass (`ng test`)

---

## Validation Steps

```bash
cd apps/frontend

# 1. App starts
ng serve
# Open http://localhost:4200

# 2. Navigate to http://localhost:4200/search
# Expected: redirected to /login (auth guard working)

# 3. Navigate to http://localhost:4200/login
# Expected: login page loads (even if empty/placeholder for now)

# 4. In browser console — no errors on startup

# 5. Run tests
ng test --watch=false
# Expected: all tests pass
```
