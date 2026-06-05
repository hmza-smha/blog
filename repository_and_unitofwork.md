[Back](./README.md)


# Repository and Unit of Work in ASP.NET Core: Do You Really Need Them?

When learning ASP.NET Core and Entity Framework Core, you'll frequently encounter two patterns:

* Repository Pattern
* Unit of Work (UoW) Pattern

Many tutorials present them as mandatory architectural components. However, modern ASP.NET Core applications often use Entity Framework Core directly without implementing either pattern.

This article explains what Repository and Unit of Work are, why they were created, how Entity Framework Core already implements much of their functionality, and when you should—or should not—use them.

---

# The Problem These Patterns Were Designed to Solve

Imagine an e-commerce application where a customer places an order.

The checkout process might involve:

1. Creating an order.
2. Creating a payment record.
3. Updating inventory.
4. Applying a discount coupon.

All of these operations must either succeed together or fail together.

Additionally, developers wanted a way to separate database access logic from business logic.

This led to the Repository and Unit of Work patterns.

---

# What Is the Repository Pattern?

A Repository acts as a collection-like abstraction over a data source.

Instead of writing database access code throughout the application, data operations are centralized inside repositories.

Example:

```csharp
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(int id);
    void Add(Order order);
}
```

Implementation:

```csharp
public class OrderRepository : IOrderRepository
{
    private readonly AppDbContext _context;

    public OrderRepository(AppDbContext context)
    {
        _context = context;
    }

    public Task<Order?> GetByIdAsync(int id)
    {
        return _context.Orders.FindAsync(id).AsTask();
    }

    public void Add(Order order)
    {
        _context.Orders.Add(order);
    }
}
```

The repository hides the details of Entity Framework from the rest of the application.

---

# Benefits of Repositories

Repositories can provide:

* Centralized query logic
* Consistent data access
* Easier switching of data sources
* Architectural boundaries between layers

For example, instead of repeating this query everywhere:

```csharp
_context.Orders
    .Where(o => o.Status == OrderStatus.Pending)
```

you could expose:

```csharp
GetPendingOrdersAsync()
```

inside a repository.

---

# What Is the Unit of Work Pattern?

A Unit of Work coordinates multiple repositories and commits changes as a single operation.

Its responsibility is:

> Track all changes during a business operation and save them together.

Typical interface:

```csharp
public interface IUnitOfWork
{
    IOrderRepository Orders { get; }
    IPaymentRepository Payments { get; }

    Task<int> SaveChangesAsync();
}
```

Usage:

```csharp
_unitOfWork.Orders.Add(order);
_unitOfWork.Payments.Add(payment);

await _unitOfWork.SaveChangesAsync();
```

Instead of each repository saving independently, the Unit of Work controls when changes are committed.

---

# Repository vs Unit of Work

These patterns have different responsibilities.

Repository:

* Performs entity-specific data operations
* Encapsulates queries
* Encapsulates CRUD operations

Unit of Work:

* Coordinates multiple repositories
* Defines transaction boundaries
* Saves all changes together

Think of it this way:

```text
OrderRepository
PaymentRepository
InventoryRepository
       |
       v
   UnitOfWork
       |
       v
  SaveChanges()
```

Repositories perform operations.

Unit of Work decides when to commit them.

---

# Unit of Work Is Not the Same as a Transaction

This is one of the most common misconceptions.

A transaction ensures:

* Atomicity
* Commit
* Rollback

Example:

```csharp
using var transaction =
    await context.Database.BeginTransactionAsync();

try
{
    await context.SaveChangesAsync();

    await transaction.CommitAsync();
}
catch
{
    await transaction.RollbackAsync();
}
```

A Unit of Work has a different responsibility.

It tracks changes and decides when they should be saved.

A transaction guarantees that the save operation is atomic.

In practice:

```text
Unit of Work = What should be saved

Transaction = How it is safely committed
```

---

# Entity Framework Core Already Implements Unit of Work

This is the most important thing modern developers should understand.

Entity Framework Core's DbContext already behaves as a Unit of Work.

It:

* Tracks changes
* Tracks new entities
* Tracks modified entities
* Tracks deleted entities
* Saves everything with SaveChanges()

Example:

```csharp
_context.Orders.Add(order);
_context.Payments.Add(payment);

await _context.SaveChangesAsync();
```

What happens internally?

1. DbContext tracks changes.
2. EF generates SQL.
3. EF opens a transaction.
4. EF executes commands.
5. EF commits or rolls back.

This is already Unit of Work behavior.

---

# Entity Framework Core Already Implements Repository-Like Behavior

DbSet<T> behaves similarly to a repository.

Example:

```csharp
_context.Orders.Add(order);

var order =
    await _context.Orders
        .FirstOrDefaultAsync(x => x.Id == id);
```

DbSet already provides:

* Add
* Update
* Remove
* Query

which are the same operations many repositories expose.

---

# The Traditional Repository + Unit of Work Approach

Many older applications use this structure:

```text
Controller
    |
    v
Service
    |
    v
Repositories
    |
    v
UnitOfWork
    |
    v
DbContext
```

Example:

```csharp
_orderRepository.Add(order);
_paymentRepository.Add(payment);

await _unitOfWork.SaveChangesAsync();
```

This was especially common before modern versions of Entity Framework.

---

# The Modern ASP.NET Core Approach

Today many teams use:

```text
Controller
    |
    v
Service
    |
    v
DbContext
```

Example:

```csharp
_context.Orders.Add(order);
_context.Payments.Add(payment);

await _context.SaveChangesAsync();
```

This eliminates unnecessary abstraction layers.

The code becomes simpler and easier to maintain.

---

# Example Where Unit of Work Makes Sense

Imagine a checkout process:

```csharp
_orderRepository.Add(order);

_paymentRepository.Add(payment);

_inventoryRepository.DecreaseStock(
    productId,
    quantity);

_couponRepository.MarkAsUsed(couponId);

await _unitOfWork.SaveChangesAsync();
```

Benefits:

* Repositories do not save independently.
* Business logic controls when changes are committed.
* All modifications are saved together.

This is one of the strongest arguments for a Unit of Work abstraction.

---

# Example Where Unit of Work Adds No Value

Consider this implementation:

```csharp
public interface IUnitOfWork
{
    Task<int> SaveChangesAsync();
}
```

```csharp
public class UnitOfWork : IUnitOfWork
{
    private readonly AppDbContext _context;

    public Task<int> SaveChangesAsync()
    {
        return _context.SaveChangesAsync();
    }
}
```

This adds:

* Another interface
* Another class
* More dependency injection registrations

without adding any new behavior.

It simply wraps DbContext.

Many developers consider this unnecessary complexity.

---

# When Should You Use Repositories?

Repositories can be useful when:

## 1. You Have Complex Queries

```csharp
GetPendingOrdersAsync()

GetOrdersByCustomerAsync()

GetTopSellingProductsAsync()
```

Instead of scattering query logic across the application.

## 2. You Want Clear Architectural Boundaries

For example:

```text
Application Layer
      |
      v
Repository Interfaces
      |
      v
Infrastructure Layer
```

This is common in Clean Architecture and Domain-Driven Design.

## 3. Multiple Data Sources Exist

Example:

* SQL Server
* MongoDB
* External APIs

Repositories can provide a unified abstraction.

---

# When Should You Use Unit of Work?

A Unit of Work may be useful when:

## 1. Multiple Repositories Must Share One Commit

```csharp
Orders.Add(order);
Payments.Add(payment);
Inventory.Update(stock);

await _unitOfWork.SaveChangesAsync();
```

## 2. You Follow Strict Clean Architecture

The application layer depends on interfaces rather than EF Core.

## 3. You Need a Consistent Transaction Boundary

The Unit of Work becomes the single place where commits occur.

---

# When Should You Avoid Repository and Unit of Work?

You may not need them when:

* You are already using EF Core.
* DbContext is directly available.
* Queries are simple.
* The abstractions merely wrap EF methods.
* No additional business value is provided.

In these cases:

```csharp
_context.Orders.Add(order);

await _context.SaveChangesAsync();
```

is often the cleanest solution.

---

# Final Recommendation

For most modern ASP.NET Core applications:

```text
Service
   |
   v
DbContext
```

is perfectly sufficient.

Use custom Repositories when they encapsulate meaningful query logic or architectural boundaries.

Use a custom Unit of Work when multiple repositories must coordinate a shared commit.

Avoid creating Repository and Unit of Work layers that simply wrap Entity Framework Core without adding real value.

The key principle is simple:

> Entity Framework Core already provides Repository-like and Unit of Work-like functionality. Only add additional abstractions when they solve a real problem, not because a pattern says you should.

---
[Back](./README.md)
