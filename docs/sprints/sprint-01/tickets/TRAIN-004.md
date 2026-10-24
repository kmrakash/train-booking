# TRAIN-004: User Registration & Basic Auth

## Summary
Implement user registration (`POST /api/auth/register`) and a login endpoint (`POST /api/auth/login`) using HTTP Basic Auth via Spring Security. Passwords are stored as BCrypt hashes — never plain text.

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

Before a user can book a train, they must have an account. This ticket adds the minimal auth layer for MVP:
- Register with name, email, password
- Login returns the authenticated user's details (no JWT yet — Sprint 2 adds JWT)
- All booking endpoints will require authentication (HTTP Basic Auth header)

> **MVP Note:** We are using HTTP Basic Auth (username:password in every request header) intentionally. It is simple, Spring Security supports it natively, and it lets us focus on the booking logic. Sprint 2 replaces this with JWT.

---

## API Contract

### Register

```
POST /api/auth/register
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "Akash Kumar",
  "email": "akash@example.com",
  "password": "secret123"
}
```

**Success — 201 Created:**
```json
{
  "id": 1,
  "name": "Akash Kumar",
  "email": "akash@example.com"
}
```

**Duplicate email — 409 Conflict:**
```json
{
  "error": "Email already registered"
}
```

**Validation error — 400 Bad Request:**
```json
{
  "error": "name, email, and password are required"
}
```

---

### Login

```
POST /api/auth/login
Authorization: Basic base64(email:password)
```

**Success — 200 OK:**
```json
{
  "id": 1,
  "name": "Akash Kumar",
  "email": "akash@example.com"
}
```

**Wrong credentials — 401 Unauthorized** (Spring Security handles this automatically)

---

## Acceptance Criteria

- [ ] `User` JPA entity maps to the `users` table from TRAIN-002
- [ ] `UserRepository` extends `JpaRepository<User, Long>` and has `findByEmail(String email)`
- [ ] `POST /api/auth/register` creates a user with a BCrypt-hashed password
- [ ] Registering with an existing email returns 409
- [ ] `POST /api/auth/login` with correct Basic Auth credentials returns 200 and user info
- [ ] `POST /api/auth/login` with wrong password returns 401
- [ ] Password is NEVER returned in any API response
- [ ] Spring Security is configured so that:
  - `/api/auth/register` and `/api/auth/login` are public (no auth required)
  - All other `/api/**` endpoints require authentication
  - CSRF is disabled (REST API, no browser sessions)
- [ ] `UserDetailsService` implementation loads users from the DB for Spring Security

---

## Technical Notes

### Spring Security Config (MVP version)

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .anyRequest().authenticated()
            )
            .httpBasic(Customizer.withDefaults());
        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### UserDetailsService

Spring Security calls this when it validates a Basic Auth header. Implement it as a service bean:

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {
    // loadUserByUsername receives the email
    // return a Spring Security User with the hashed password
}
```

### Never expose the password field

Use a response DTO:
```java
public record UserResponse(Long id, String name, String email) {}
```

---

## TDD Note

Write these tests FIRST:

**Unit test — `UserServiceTest`:**
1. `register()` saves a user with a BCrypt-hashed password (assert password does NOT equal original)
2. `register()` with duplicate email throws a specific exception

**Integration test — `AuthControllerTest`:**
1. `POST /api/auth/register` with valid body returns 201 and user JSON (no password field)
2. `POST /api/auth/register` with existing email returns 409
3. `POST /api/auth/register` with missing fields returns 400
4. `GET /api/trains/search?...` without auth returns 401
5. `GET /api/trains/search?...` with valid Basic Auth returns 200

---

## Dependencies

- TRAIN-002 must be complete (users table must exist)

---

## Definition of Done

- [ ] User entity, repository, service, controller all implemented
- [ ] Spring Security config in place
- [ ] All unit and integration tests pass
- [ ] Registration and login tested manually with curl
- [ ] Attempting any protected endpoint without auth returns 401

---

## Validation Steps

```bash
# Start the app
cd apps/backend && ./mvnw spring-boot:run

# 1. Register a new user
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"name":"Akash Kumar","email":"akash@example.com","password":"secret123"}'
# Expected: 201 {"id":1,"name":"Akash Kumar","email":"akash@example.com"}

# 2. Register with same email again
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"name":"Someone Else","email":"akash@example.com","password":"pass"}'
# Expected: 409 {"error":"Email already registered"}

# 3. Login (Basic Auth = base64 of "email:password")
curl -X POST http://localhost:8080/api/auth/login \
  -u "akash@example.com:secret123"
# Expected: 200 {"id":1,"name":"Akash Kumar","email":"akash@example.com"}

# 4. Wrong password
curl -X POST http://localhost:8080/api/auth/login \
  -u "akash@example.com:wrongpass"
# Expected: 401 Unauthorized

# 5. Access protected endpoint without auth
curl "http://localhost:8080/api/trains/search?source=Delhi&destination=Mumbai&date=2026-11-01"
# Expected: 401 Unauthorized

# 6. Access protected endpoint WITH auth
curl "http://localhost:8080/api/trains/search?source=Delhi&destination=Mumbai&date=2026-11-01" \
  -u "akash@example.com:secret123"
# Expected: 200 with train list

# 7. Run tests
./mvnw test
# Expected: BUILD SUCCESS
```
