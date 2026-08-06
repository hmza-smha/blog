
[Back](./README.md)


# Senior Software Developer

## Intro
1. SOLID Principles
2. What are Design Patterns? 
    1. Dependency Injection
    2. Factory Pattern
    3. Repository Pattern
    4. Unit of Work
    5. CQRS

3. What software architectures have you worked with?
    1. Monolith
    2. Modular Monolith
    3. Clean Architectures

---

## How you design an API?

---

## How would you secure an API?

* OAuth 2.0
* JWT
* API Keys
* Rate Limiting
* Input Validation - Never Trust Client Input
* SQL Injection Prevention
* XSS Prevention
* CSRF Protection

---

## What is the difference between JWT and Session Authentication?

* Stateless vs. Stateful
* Storage
* Performance

---

## How do you revoke JWTs?

* Blacklist/Denylist
* Short-Lived Tokens
* Refresh Token Rotation
* Token Versioning
* Key Rotation

---

## How do you design API pagination?

* Offset Pagination
* Cursor Pagination
* Performance

---

## How do you prevent duplicate API requests?

* Idempotency Keys
* Locking

---
## How do you handle long-running API requests?

* Async Processing
* Background Jobs
* Message Queues
* 202 Accepted
* Polling
* Webhooks
* Job Status Endpoint
* Timeouts
* Retries
* Idempotency



# Scenarios

# Scenario 1: Slow API

> A GET endpoint that usually responds in 150 ms now takes 8 seconds after yesterday's deployment.

* Git history
* Logs

---


# Scenario 2: External API Failure

> Your application depends on a third-party payment provider. Today, their API is timing out.

---

# Scenario 3: Caching

> Your dashboard makes 12 database queries and receives 3 million visits per day.

Would you:

* Add Redis?
* Cache in memory?
* Cache at the CDN?
* Optimize SQL?


---
[Back](./README.md)
