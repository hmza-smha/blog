
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
    4. Layered Architectures

4. How you design an API?


# Scenario 1: Slow API

> A GET endpoint that usually responds in 150 ms now takes 8 seconds after yesterday's deployment.

### Follow-up

* What data do you collect first?
* How do you determine whether it's the application, database, or infrastructure?
* Would you roll back?

**Look for**

* Metrics before assumptions
* Query analysis
* Deployment comparison
* Distributed tracing
* Profiling

---


# Scenario 2: External API Failure

> Your application depends on a third-party payment provider. Today, their API is timing out.

### Follow-up

* What happens to users?
* Would you retry?
* How many retries?
* Would you use a circuit breaker?
* Would you queue requests?
* How do you avoid duplicate payments?

---


# Scenario 3: Scaling

> Your application currently handles 500 requests per minute. Marketing expects 100,000 requests per minute next month.

### Follow-up

* What would break first?
* What would you optimize first?
* Would you cache?
* Would you scale the database?
* Would you introduce queues?

---


# Scenario 4: Race Condition

> Occasionally, two users purchase the last available product at the same time.

### Follow-up

* Why is this happening?
* How would you reproduce it?
* How would you fix it?
* Database lock?
* Optimistic concurrency?
* Distributed lock?

---



# Scenario 5: Caching

> Your dashboard makes 12 database queries and receives 3 million visits per day.

Would you:

* Add Redis?
* Cache in memory?
* Cache at the CDN?
* Optimize SQL?

Why?

### Follow-up

What if the data changes every minute?

---




# Scenario 6: Production Is Down

> The system is down and customers cannot place orders. You are the senior developer on call. What do you do?

### Follow-up

* What is the first thing you check?
* Do you roll back immediately?
* How do you communicate with stakeholders?
* How do you identify the root cause?
* How do you prevent it from happening again?

**Look for**

* Calm, structured approach
* Incident management
* Logs, metrics, monitoring
* Rollback strategy
* Communication

---
[Back](./README.md)
