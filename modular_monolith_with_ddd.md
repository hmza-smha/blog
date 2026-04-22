[Back](./README.md)

# Production Softwre System

- Software architecture should always be created to resolve **specific business problems**. 
- A lot of other factors influence your software architecture - your team, opinions, preferences, experiences, technical constraints, time, budget, etc.


## Modular Monolith with DDD


Because of the above, **Modular Monolith with DDD** is one of the many ways to solve some problems. Remember to always pick the best solution which is appropriate to the problem class you have.




## Terminologies

| Concept                        | Simple Definition                                                       | Focus                    | Scope                             | Example                                                      |
| ------------------------------ | ----------------------------------------------------------------------- | ------------------------ | --------------------------------- | ------------------------------------------------------------ |
| **Monolith**                   | A single application where all parts are combined and deployed together | Deployment style         | Whole system                      | One backend handling users, orders, payments in one codebase |
| **Module**                     | A self-contained part of the system that handles a specific feature (or domain)     | Code organization        | Part of the system                | User module, Order module                                    |
| **Modular Monolith**           | A monolith structured into well-separated modules with clear boundaries | Architecture + structure | Whole system (internally modular) | One app with independent User, Order, Payment modules        |
| **DDD (Domain-Driven Design)** | A design approach that models software based on business concepts       | Design methodology       | Applies across system design      | Using entities like Order, Customer based on real business   |
| **Domain**                     | The real-world business problem the software is solving                 | Business understanding   | Problem space                     | E-commerce domain, Banking domain



## C4 Model

> The **C4 Model** is a simple, structured way to **visualize and document software architecture** at different levels of detail.


> The C4 Model helps you explain a system from **big picture → deep technical details**, step by step.

👉 “C4” stands for:

* **C**ontext: Shows the system as a whole and how it interacts with external users and systems.
* **C**ontainers: Breaks the system into major deployable parts (apps, services, databases) and how they communicate.
* **C**omponents: Details the internal structure of a container, showing its main building blocks and interactions.
* **C**ode: Represents the actual implementation level (classes, methods, functions) inside a component.

## Key assumptions:
* API contains no application logic
* API communicates with Modules using a small interface to send **Queries** and **Commands**
* Each Module has its own interface which is used by API
* Modules communicate each other only **asynchronously** using - Events Bus - **direct method calls are not allowed**
* Each Module has it's own data in a separate schema - shared data is not allowed
    * Module data could be moved into separate databases if desired
* Modules can only have a dependency on the integration events assembly of other Module (see Module level view)
* Each Module has its own **Composition Root**, which implies that each Module has its own **Inversion-of-Control** container
* API as a host needs to initialize each module and each module has an initialization method
* Each Module is **highly encapsulated** - only required types and members are public, the rest are internal or private

Each Module has Clean Architecture and consists of the following submodules (assemblies):

* **Application** - the application logic submodule which is responsible for requests processing: use cases, domain events, integration events, internal commands.
* **Domain** - Domain Model in Domain-Driven Design terms implements the applicable Bounded Context
* **Infrastructure** - infrastructural code responsible for module initialization, background processing, data access, communication with Events Bus and other external components or systems
* **IntegrationEvents** - Contracts published to the Events Bus; only this assembly can be called by other modules

## API and Module Communication

The API only communicates with Modules in two ways: during module initialization and request processing.

### Module initialization
Each module has a static `Initialize` method which is invoked in the API `Startup` class. All configuration needed by this module should be provided as arguments to this method. All services are configured during initialization and the Composition Root is created using the Inversion-of-Control Container.

```c#
public static void Initialize(
    string connectionString,
    IExecutionContextAccessor executionContextAccessor,
    ILogger logger,
    EmailsConfiguration emailsConfiguration)
{
    var moduleLogger = logger.ForContext("Module", "Meetings");

    ConfigureCompositionRoot(connectionString, executionContextAccessor, moduleLogger, emailsConfiguration);

    QuartzStartup.Initialize(moduleLogger);

    EventsBusStartup.Initialize(moduleLogger);
}
```


### Request processing

Each module has the same interface signature exposed to the API. It contains 3 methods: command with result, command without result and query.
```c#
public interface IMeetingsModule
{
    Task<TResult> ExecuteCommandAsync<TResult>(ICommand<TResult> command);

    Task ExecuteCommandAsync(ICommand command);

    Task<TResult> ExecuteQueryAsync<TResult>(IQuery<TResult> query);
}
```


## Modules Communication

### Domain Events & Integration Events


| Aspect                  | 🟢 Domain Events                                       | 🔵 Integration Events                             |
| ----------------------- | ------------------------------------------------------ | ------------------------------------------------- |
| **Definition**          | Represents something that happened *inside the domain* | Represents something to notify *external systems* |
| **Scope**               | Internal (same application)                            | External (between services/microservices)         |
| **Purpose**             | Enforce business rules and trigger domain logic        | Communicate with other systems                    |
| **Execution**           | Immediate (in-process)                                 | Asynchronous (via message broker)                 |
| **Timing**              | Synchronous (When a domain event is raised, its handlers run immediately in the same execution flow (same request, same thread, same transaction).)                        | Eventually consistent                             |
| **Transaction**         | Same database transaction                              | Separate transaction                              |
| **Failure Impact**      | Rolls back entire operation                            | Can retry independently                           |
| **Coupling**            | Loosely coupled inside domain                          | Loosely coupled across systems                    |
| **Transport Mechanism** | In-memory (e.g., MediatR)                              | Message bus (RabbitMQ, Kafka, etc.)               |
| **Performance Impact**  | Affects request time                                   | Doesn’t block user request                        |
| **Data Shape**          | Rich domain objects                                    | Flattened DTOs (contracts)                        |
| **Reliability Concern** | Lower (same process)                                   | High (must ensure delivery)                       |
| **Persistence Needed**  | No                                                     | Yes (Outbox pattern recommended)                  |
| **Audience**            | Internal handlers                                      | External services                                 |
| **Examples**            | `OrderCreatedDomainEvent`                              | `OrderCreatedIntegrationEvent`                    |

---

#### How They Work Together (Typical Flow)

```text
Command → Domain Event (🟢) → Internal Handling
                           ↓
                   Integration Event (🔵)
                           ↓
                     Event Bus → Other Services
```

---

#### Quick Mental Model

* 🟢 **Domain Event** = “Something happened, handle it internally”
* 🔵 **Integration Event** = “Something happened, tell the outside world”

---

##### 🎯 Example

##### Domain Event

* Triggered when order is created
* Updates inventory, applies discounts

##### Integration Event

* Sent after order is created
* Notifies:

  * Email service
  * Shipping service



### Module Requests Processing via CQRS

Processing of Commands and Queries is separated by applying the architectural style/pattern **Command Query Responsibility Segregation (CQRS)**.



* **Commands** are processed using the **Write Model**, which is implemented using DDD (Domain-Driven Design) tactical patterns.
* **Queries** are processed using the **Read Model**, which is implemented by executing direct queries on the database.



#### Key Advantages

* **Solution is appropriate to the problem** – Reading and writing needs are usually different; CQRS allows each to evolve independently.
* **Supports Single Responsibility Principle (SRP)** – Each handler is dedicated to performing exactly one specific task.
* **Supports Interface Segregation Principle (ISP)** – Each handler implements an interface with exactly one method.
* **Supports Parameter Object pattern** – Commands and Queries are objects that are easy to serialize/deserialize.
* **Decorator Pattern** – Provides an easy way to handle cross-cutting concerns (like logging, validation, or transactions) by wrapping handlers.
* **Loose Coupling via Mediator Pattern** – Separates the invoker of a request from the actual handler, promoting a decoupled architecture.

#### Disadvantage

* **Indirection** – The Mediator pattern introduces an extra layer of indirection, which can make it harder to trace or reason about which specific class is handling a request compared to direct method calls.




### Domain Model Principles and Attributes

The **Domain Model** is the central and most critical part of the system. It is designed with special attention to ensure business logic is protected and accurately represented.



#### Key Principles

* **High Level of Encapsulation**
    * All members are `private` by default, then `internal`. Only the bare minimum is marked as `public` at the very edge to maintain strict control over state transitions.
* **High Level of Persistence Ignorance (PI)**
    * The model has no dependencies on infrastructure, databases, or external frameworks. All classes are **POCOs** (Plain Old CLR Objects).
* **Rich in Behavior**
    * All business logic resides within the Domain Model. There are no "leaks" to the application layer; this avoids **Anemic Domain Models**.
* **Low Level of Primitive Obsession**
    * Instead of using raw strings or integers, attributes are grouped into meaningful **Value Objects** (e.g., `MeetingGroupId` instead of `Guid`).
* **Business Language (Ubiquitous Language)**
    * Classes and methods are named using the specific business terminology of the **Bounded Context**.
* **Testable Design**
    * Since the Domain Model is the core of the system, it is designed to be easily testable without requiring database connections or complex setups.


### Cross-Cutting Concerns

To support **Single Responsibility Principle** and Don't **Repeat Yourself principles**, the implementation of cross-cutting concerns is done using the **Decorator Pattern**. Each Command processor is decorated by 3 decorators: logging, validation and unit of work.


```txt
                ┌─────────────────────────────────────┐
                │  LoggingCommandHandlerDecorator      │  ← outermost (logs start/end/error)
                │  ┌───────────────────────────────┐  │
                │  │ ValidationCommandHandlerDeco.. │  │  ← validates with FluentValidation
                │  │  ┌─────────────────────────┐  │  │
                │  │  │ UnitOfWorkCommandHandler │  │  │  ← commits DB transaction after
                │  │  │  ┌───────────────────┐  │  │  │
                │  │  │  │  Actual Handler   │  │  │  │  ← your business logic
                │  │  │  │  (e.g. Assign...) │  │  │  │
                │  │  │  └───────────────────┘  │  │  │
                │  │  └─────────────────────────┘  │  │
                │  └───────────────────────────────┘  │
                └─────────────────────────────────────┘
                

```

#### Logging
The Logging decorator logs execution, arguments and processing of each Command. This way each log inside a processor has the log context of the processing command.

#### Validation
The Validation decorator performs Command data validation. It checks rules against Command arguments using the FluentValidation library.

#### Unit Of Work
All Command processing has side effects. To avoid calling commit on every handler, `UnitOfWorkCommandHandlerDecorator` is used. It additionally marks `InternalCommand` as processed (if it is Internal Command) and dispatches all Domain Events (as part of Unit Of Work).



## Event Sourcing

**The problem Event Sourcing solves**

In a normal database, you only store the *current state*. A bank account row says "balance: $350." You don't know how it got there — was it one deposit? Ten transactions? A reversal? That history is gone.

Event Sourcing flips this completely: instead of storing the current state, you store every *event that ever happened*. The current state is not saved anywhere — you *calculate* it by replaying all the events from the beginning.

Here's the intuition diagram:The event store is essentially an **append-only log**. You never update or delete a row — you only ever add new events at the end. The current state of anything is the result of "folding" all past events together.

Now, the natural question is: *how does a change actually happen?* This is where Event Sourcing connects directly to what you were reading about Commands and Aggregates:This is exactly what section 3.8 was describing — state only changes via a Command, and that Command produces a Domain Event. In Event Sourcing, that event is not just a notification — it *is* the persisted record of what happened.



## What is an Aggregate Root?

> An **Aggregate Root** is the main entity that controls access to a group of related objects (the *aggregate*), ensuring consistency and enforcing business rules.

---

### 🧩 Think of it like this

👉 You don’t interact with everything directly —
you go through a **single entry point**.

That entry point = **Aggregate Root**

---

### Example: Order System

### Aggregate = Order + its items

* Order (root)
* OrderItems (children)


### 🔒 Key Rule

❌ Don’t do this:

```csharp
order.Items.Add(new OrderItem(...)); // BAD
```

✔ Do this:

```csharp
order.AddItem(productId, quantity); // GOOD
```

👉 Why?
Because the **Aggregate Root protects invariants**.

---

### 🎯 Responsibilities of Aggregate Root

* Enforce business rules
* Maintain consistency
* Control access to child entities
* Raise domain events


### What is an Entity Version?

## Definition

> An **Entity Version** is a number (or timestamp) used to track changes to an entity, usually for **optimistic concurrency control**.

---

### Example

```csharp
public class Order
{
    public int Id { get; set; }
    public int Version { get; set; }
}
```

---

### How it works

1. Read entity:

```text
Order Id = 1, Version = 5
```

2. Update it:

```text
WHERE Id = 1 AND Version = 5
```

3. If success:

```text
Version → 6
```

4. If someone else updated it:

```text
Version mismatch ❌ → concurrency conflict
```

---

### Why we need it?

👉 To prevent:

> ❌ Lost updates (two users overwrite each other)

---

### Example Scenario

Two users edit the same order:

```text
User A reads Version = 1
User B reads Version = 1
```

* User A updates → Version becomes 2 ✅
* User B updates → fails ❌ (still using Version = 1)

---


## FULL PROJECT STRUCTURE

```
src/                                                # Root of the entire application
│
├── API/                                            # Presentation layer (entry point, no business logic)
│   └── App.API/
│       │
│       ├── Controllers/                            # HTTP endpoints → map requests to Commands/Queries
│       │   ├── OrdersController.cs                 # Handles order-related HTTP requests
│       │   ├── CatalogController.cs                # Handles catalog-related HTTP requests
│       │   └── CustomersController.cs              # Handles customer-related HTTP requests
│       │
│       ├── Modules/                                # Responsible for initializing each module
│       │   ├── OrdersStartup.cs                    # Calls OrdersModule.Initialize(...)
│       │   ├── CatalogStartup.cs                   # Calls CatalogModule.Initialize(...)
│       │   └── CustomersStartup.cs                 # Calls CustomersModule.Initialize(...)
│       │
│       ├── Configuration/                          # API-level configurations (not business logic)
│       │   ├── SwaggerConfiguration.cs             # Swagger/OpenAPI setup
│       │   ├── AuthenticationConfiguration.cs      # Auth (JWT, Identity, etc.)
│       │   └── ApiConfiguration.cs                 # General API configs (CORS, etc.)
│       │
│       ├── Middlewares/                            # Cross-cutting concerns at HTTP level
│       │   ├── ExceptionHandlingMiddleware.cs      # Global exception handling
│       │   └── RequestLoggingMiddleware.cs         # Logs incoming HTTP requests
│       │
│       ├── CompositionRoot/                        # API dependency injection setup
│       │   └── ApiCompositionRoot.cs               # Registers API services in DI container
│       │
│       └── Program.cs                              # Application bootstrap (starts the app)
│
├── Modules/                                        # All business modules (bounded contexts)
│   │
│   ├── Orders/                                     # Orders bounded context
│   │   │
│   │   ├── Orders.API/                             # Public interface exposed to API
│   │   │   ├── IOrdersModule.cs                    # Contract used by API (ExecuteCommand/Query)
│   │   │   └── OrdersModule.cs                     # Implementation + entry point into module
│   │   │
│   │   ├── Orders.Application/                     # Application layer (CQRS, use cases)
│   │   │   │
│   │   │   ├── Commands/                           # Write operations (state-changing)
│   │   │   │   └── CreateOrder/
│   │   │   │       ├── CreateOrderCommand.cs       # Data for creating order
│   │   │   │       ├── CreateOrderCommandHandler.cs# Handles business flow
│   │   │   │       └── CreateOrderValidator.cs     # Validates command input
│   │   │   │
│   │   │   ├── Queries/                            # Read operations (no side effects)
│   │   │   │   └── GetOrderDetails/
│   │   │   │       ├── GetOrderDetailsQuery.cs     # Query definition
│   │   │   │       ├── GetOrderDetailsHandler.cs   # Executes DB read
│   │   │   │       └── OrderDto.cs                 # Read model (flattened data)
│   │   │   │
│   │   │   ├── DomainEvents/                       # Handles domain events internally
│   │   │   │   └── OrderCreatedDomainEventHandler.cs # Reacts to domain event
│   │   │   │
│   │   │   ├── IntegrationEvents/                  # Converts domain → integration events
│   │   │   │   └── OrderCreatedPublishHandler.cs   # Publishes event to EventBus
│   │   │   │
│   │   │   ├── InternalCommands/                   # Async internal processing
│   │   │   │   └── ProcessOrderInternalCommand.cs  # Background task inside module
│   │   │   │
│   │   │   ├── Behaviors/                          # Decorators (cross-cutting)
│   │   │   │   ├── LoggingCommandHandlerDecorator.cs    # Logs command execution
│   │   │   │   ├── ValidationCommandHandlerDecorator.cs # Validates commands
│   │   │   │   └── UnitOfWorkCommandHandlerDecorator.cs # Handles transactions + events
│   │   │   │
│   │   │   └── Contracts/                          # Interfaces for infrastructure
│   │   │       └── IOrdersRepository.cs            # Repository abstraction
│   │   │
│   │   ├── Orders.Domain/                          # Core domain model (DDD)
│   │   │   │
│   │   │   ├── Aggregates/                         # Aggregate roots
│   │   │   │   └── Order/
│   │   │   │       ├── Order.cs                    # Aggregate root (business rules)
│   │   │   │       ├── OrderItem.cs                # Child entity
│   │   │   │       └── OrderVersion.cs             # Optimistic concurrency version
│   │   │   │
│   │   │   ├── Entities/                           # Supporting entities
│   │   │   │   └── CustomerSnapshot.cs             # Snapshot of customer at order time
│   │   │   │
│   │   │   ├── ValueObjects/                       # Immutable domain values
│   │   │   │   ├── OrderId.cs                      # Strongly-typed ID
│   │   │   │   ├── ProductId.cs                    # Product identifier
│   │   │   │   └── MoneyValue.cs                   # Money abstraction
│   │   │   │
│   │   │   ├── DomainEvents/                       # Domain-level events
│   │   │   │   └── OrderCreatedDomainEvent.cs      # Raised inside aggregate
│   │   │   │
│   │   │   ├── BusinessRules/                      # Domain invariants
│   │   │   │   └── OrderMustHaveAtLeastOneItemRule.cs # Business validation rule
│   │   │   │
│   │   │   └── Services/                           # Domain services (complex logic)
│   │   │       └── PricingDomainService.cs         # Pricing calculations
│   │   │
│   │   ├── Orders.Infrastructure/                  # Technical implementation
│   │   │   │
│   │   │   ├── Configuration/                      # Module-specific settings
│   │   │   │   └── OrdersSettings.cs
│   │   │   │
│   │   │   ├── DataAccess/                         # Database access
│   │   │   │   ├── OrdersDbContext.cs              # ORM context
│   │   │   │   ├── OrdersRepository.cs             # Repository implementation
│   │   │   │   └── Queries/
│   │   │   │       └── OrderQueries.cs             # Raw SQL / read model queries
│   │   │   │
│   │   │   ├── Domain/                             # Domain-related infrastructure
│   │   │   │   └── DomainEventsDispatcher.cs       # Dispatches domain events
│   │   │   │
│   │   │   ├── Processing/                         # CQRS execution engine
│   │   │   │   ├── Commands/                       # Command execution pipeline
│   │   │   │   ├── Queries/                        # Query execution pipeline
│   │   │   │   └── InternalCommandsProcessor.cs    # Processes async commands
│   │   │   │
│   │   │   ├── EventBus/                           # Integration with Event Bus
│   │   │   │   └── IntegrationEventPublisher.cs    # Publishes integration events
│   │   │   │
│   │   │   └── CompositionRoot/                    # Module DI container
│   │   │       ├── OrdersCompositionRoot.cs        # Registers module dependencies
│   │   │       └── OrdersStartup.cs                # Module initialization entry
│   │   │
│   │   └── Orders.IntegrationEvents/               # Public contracts for other modules
│   │       └── Events/
│   │           ├── OrderCreatedIntegrationEvent.cs # Event published externally
│   │           └── OrderCancelledIntegrationEvent.cs
│   │
│   ├── Catalog/                                    # Catalog module (same structure as Orders)
│   │   └── ...                                     # Products, categories, pricing, etc.
│   │
│   └── Customers/                                  # Customers module (same structure)
│       └── ...                                     # Users, profiles, etc.
│
├── BuildingBlocks/                                 # Shared infrastructure (NOT business logic)
│   │
│   ├── Domain/                                     # Base DDD abstractions
│   │   ├── Entity.cs                               # Base entity
│   │   ├── AggregateRoot.cs                        # Base aggregate root
│   │   ├── ValueObject.cs                          # Base value object
│   │   ├── IDomainEvent.cs                         # Domain event contract
│   │   ├── DomainEventDispatcher.cs                # Dispatches domain events
│   │   └── EntityVersion.cs                        # Optimistic concurrency support
│   │
│   ├── Application/                                # CQRS abstractions
│   │   ├── ICommand.cs                             # Command marker
│   │   ├── ICommandHandler.cs                      # Command handler interface
│   │   ├── IQuery.cs                               # Query marker
│   │   ├── IQueryHandler.cs                        # Query handler interface
│   │   ├── ICommandScheduler.cs                    # Schedules async commands
│   │   ├── InternalCommand.cs                      # Internal async command base
│   │   └── Result.cs                               # Result wrapper
│   │
│   ├── Infrastructure/                             # Shared technical components
│   │   ├── UnitOfWork/
│   │   │   ├── IUnitOfWork.cs                      # Transaction abstraction
│   │   │   └── UnitOfWork.cs                       # Transaction implementation
│   │   │
│   │   ├── DataAccess/
│   │   │   ├── IDbConnectionFactory.cs             # DB connection abstraction
│   │   │   └── DbConnectionFactory.cs              # DB connection implementation
│   │   │
│   │   ├── Logging/
│   │   │   └── LoggerAdapter.cs                    # Logging wrapper
│   │   │
│   │   └── Validation/
│   │       └── ValidationException.cs              # Validation errors
│   │
│   ├── EventBus/                                   # Async messaging system
│   │   ├── IEventBus.cs                            # Event bus abstraction
│   │   ├── IntegrationEvent.cs                     # Base integration event
│   │   ├── EventBusImplementation.cs               # RabbitMQ/Kafka/etc.
│   │   │
│   │   ├── Outbox/                                 # Reliable publishing (Outbox pattern)
│   │   │   ├── OutboxMessage.cs                    # Stored event message
│   │   │   ├── OutboxProcessor.cs                  # Sends events to bus
│   │   │   └── OutboxRepository.cs                 # Stores events in DB
│   │   │
│   │   └── Inbox/                                  # Reliable consumption
│   │       ├── InboxMessage.cs                     # Received event tracking
│   │       └── InboxProcessor.cs                   # Processes incoming events
│   │
│   └── SharedKernel/                               # Shared utilities
│       ├── ExecutionContext/
│       │   ├── IExecutionContextAccessor.cs        # Current user/context abstraction
│       │   └── ExecutionContextAccessor.cs
│       │
│       ├── Clock/
│       │   └── SystemClock.cs                      # Time provider
│       │
│       └── Exceptions/
│           └── BusinessRuleValidationException.cs  # Domain rule violation
│
└── Bootstrap/                                      # System-level initialization
    └── CompositionRoot/
        └── RootCompositionRoot.cs                  # Wires everything together
```

---
[Back](./README.md)
