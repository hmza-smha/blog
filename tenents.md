[Back](./README.md)

---

## 📑 Table of Contents

| Section | Topic |
|---------|-------|
| [1](#what-is-a-tenant) | What is a Tenant? |
| [2](#key-characteristics) | Key Characteristics |
| [3](#simple-architecture-example) | Architecture Example |
| [4](#common-multi-tenancy-models) | Multi-Tenancy Models |
| [5](#benefits) | Benefits |
| [6](#challenges) | Challenges |
| [7](#multi-tenant-vs-single-tenant) | vs Single-Tenant |

---

# Tenants

A **multi-tenant software application** is a system where **one shared application instance serves multiple customers (tenants)**, while **keeping each tenant’s data and configuration isolated**.

Think of it like an **apartment building** 🏢

* One building (application)
* Many apartments (tenants)
* Shared infrastructure (elevators, electricity)
* Private spaces (each tenant’s data & settings)

---

## What is a *Tenant*?

A **tenant** is usually:

* A company
* An organization
* Sometimes a single customer

Each tenant:

* Uses the same app
* Cannot see other tenants’ data
* May have its own users, roles, and configuration

---

## Key Characteristics

### 1. Shared Application

* One codebase
* One deployment
* One infrastructure (servers, services)

### 2. Data Isolation

Data is **logically separated**, not necessarily physically.
Common approaches:

* TenantId column in tables
* Separate schemas
* Separate databases

### 3. Customization per Tenant

Each tenant may have:

* Different users & roles
* Different permissions
* Branding (logo, colors)
* Feature toggles

---

## Simple Architecture Example

```
Users → Web App → API → Database
                    |
                    └── Data filtered by TenantId
```

---

## Common Multi-Tenancy Models

### 1. **Shared Database, Shared Schema**

* All tenants share tables
* Data separated by `TenantId`

**Example table**

```sql
Orders
-----------------
Id
TenantId
OrderDate
Amount
```

✅ Cheapest
❌ Harder security & scaling

---

### 2. **Shared Database, Separate Schemas**

* One database
* Each tenant has its own schema

```
db
 ├── tenant1.Orders
 ├── tenant2.Orders
```

✅ Better isolation
❌ More complex migrations

---

### 3. **Separate Database per Tenant**

* Each tenant has its own database

```
Tenant A → DB_A
Tenant B → DB_B
```

✅ Strong isolation
❌ Higher cost & operational overhead

---

## Benefits

✅ Lower cost
✅ Easier updates (one deployment)
✅ Better resource utilization
✅ Faster onboarding of new tenants

---

## Challenges

⚠ Data isolation bugs
⚠ Complex authorization logic
⚠ Tenant-aware caching
⚠ Performance issues (noisy neighbor problem)

---

## Multi-Tenant vs Single-Tenant

| Feature       | Multi-Tenant | Single-Tenant     |
| ------------- | ------------ | ----------------- |
| App instance  | Shared       | One per customer  |
| Cost          | Low          | High              |
| Customization | Limited      | Full              |
| Isolation     | Logical      | Physical          |
| Scaling       | Harder       | Easier per tenant |

---


There are **two main dimensions** when talking about *approaches to implement multi-tenancy*:

1. **How you isolate tenant data**
2. **How your application becomes “tenant-aware”**

---

# 1️⃣ Data Isolation Approaches (MOST IMPORTANT)

This is the core of multi-tenancy.

---

## **Approach 1: Shared Database, Shared Schema**

**(TenantId per row)**

### How it works

* One database
* One set of tables
* Every table has `TenantId`

```sql
Users
----------------
Id
TenantId
Email
Name
```

### How tenant isolation works

* Every query filters by `TenantId`

```sql
SELECT * FROM Orders WHERE TenantId = @TenantId
```

### Pros

✅ Cheapest
✅ Simple infrastructure
✅ Easy onboarding

### Cons

❌ Highest risk if you forget a filter
❌ Harder performance tuning
❌ Noisy neighbor risk

### Best for

* Startups
* Many small tenants
* Early-stage SaaS

---

## **Approach 2: Shared Database, Separate Schemas**

### How it works

* One database
* Each tenant has its own schema

```text
tenant1.Users
tenant2.Users
```

### How tenant isolation works

* Connection points to tenant schema
* Same code, different schema

### Pros

✅ Better isolation
✅ Easier per-tenant backups
✅ Less risk of data leakage

### Cons

❌ Schema migrations are harder
❌ More operational complexity

### Best for

* Medium SaaS
* Tenants with moderate data volume

---

## **Approach 3: Separate Database per Tenant**

### How it works

* Each tenant has its own database
* App selects DB based on tenant

```text
Tenant A → DB_A
Tenant B → DB_B
```

### Pros

✅ Strong isolation
✅ Easier compliance (GDPR, HIPAA)
✅ Per-tenant scaling

### Cons

❌ Higher cost
❌ Complex operations
❌ Connection management complexity

### Best for

* Enterprise SaaS
* Regulated industries
* High-value customers

---

## Summary Table (Data Level)

| Approach                  | Isolation | Cost   | Complexity |
| ------------------------- | --------- | ------ | ---------- |
| Shared DB + Shared Schema | Low       | Low    | Low        |
| Shared DB + Schemas       | Medium    | Medium | Medium     |
| DB per Tenant             | High      | High   | High       |

---

[Back](./README.md)