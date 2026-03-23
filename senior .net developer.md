# Senior .NET Developer Interview Questions & Answers

---

## 1. How does the .NET Core dependency injection container work, and when would you use transient, scoped, and singleton lifetimes?

The built-in DI container in .NET Core follows the **Inversion of Control** principle. Services are registered in `Program.cs` (or `Startup.cs`) and resolved automatically via constructor injection.

- **Transient** (`AddTransient`): A new instance is created every time it is requested. Use for lightweight, stateless services.
- **Scoped** (`AddScoped`): One instance per HTTP request (or per `IServiceScope`). Use for services that hold request-level state, such as `DbContext`.
- **Singleton** (`AddSingleton`): One instance for the entire application lifetime. Use for stateless services, caches, or configuration holders.

**Common pitfall**: Injecting a scoped service into a singleton causes a **captive dependency** — the scoped instance lives as long as the singleton, which leads to stale data and concurrency bugs. The framework throws an `InvalidOperationException` when `ValidateScopes` is enabled.

---

## 2. Explain the middleware pipeline in ASP.NET Core. How does request processing flow?

The middleware pipeline is a chain of delegates executed in order for every HTTP request. Each middleware component can:

1. Process the request and pass it to the next middleware via `next(context)`.
2. Short-circuit the pipeline by not calling `next()`.
3. Execute logic both before and after the next middleware (creating a "sandwich" pattern).

The order of registration in `Program.cs` matters. A typical pipeline:

```
UseExceptionHandler → UseHsts → UseHttpsRedirection → UseStaticFiles
→ UseRouting → UseCors → UseAuthentication → UseAuthorization
→ UseEndpoints
```

Custom middleware is created by implementing `IMiddleware` or writing an inline delegate with `app.Use()`. For cross-cutting concerns (logging, correlation IDs, rate limiting), middleware is the correct architectural layer.

---

## 3. What is the difference between `async/await` and raw `Task` usage? How do you avoid common async pitfalls?

`async/await` is syntactic sugar over the Task-based Asynchronous Pattern (TAP). The compiler generates a state machine that suspends execution at `await` and resumes when the awaited task completes — without blocking a thread.

**Key pitfalls to avoid:**

- **`.Result` or `.Wait()` on async code** — causes deadlocks in environments with a `SynchronizationContext` (ASP.NET classic, UI apps). In ASP.NET Core there is no sync context, but blocking still wastes a thread pool thread.
- **`async void`** — exceptions cannot be caught by the caller. Only use for event handlers. Always return `Task` or `Task<T>`.
- **Not using `ConfigureAwait(false)` in libraries** — in library code, append `.ConfigureAwait(false)` to avoid capturing the caller's synchronization context.
- **Creating unnecessary state machines** — if a method simply returns another async call with no additional logic, return the `Task` directly instead of marking the method `async`.

---

## 4. How would you design a scalable REST API with proper versioning, error handling, and pagination?

**Versioning**: Use URL segment versioning (`/api/v1/orders`) for clarity, or header-based versioning (`Api-Version: 2`) for cleaner URLs. The `Asp.Versioning.Http` package provides built-in support.

**Error handling**: Return consistent problem details using **RFC 7807** (`ProblemDetails`). Use a global exception handler middleware that catches unhandled exceptions, logs the full context, and returns a sanitized error response. Never leak stack traces in production.

**Pagination**: Use cursor-based pagination for large datasets (better performance than offset-based). Return a response envelope:

```json
{
  "data": [...],
  "nextCursor": "abc123",
  "hasMore": true
}
```

**Additional design principles:**
- Use proper HTTP status codes (201 for creation, 204 for deletion, 409 for conflicts).
- Implement idempotency keys for non-idempotent operations (POST).
- Apply rate limiting via `System.Threading.RateLimiting`.
- Use `CancellationToken` in all controller actions to handle client disconnects.

---

## 5. How does Entity Framework Core handle migrations, change tracking, and performance optimization?

**Migrations**: EF Core compares the current model snapshot with the `DbContext` configuration and generates migration files. Apply via `dotnet ef database update` or `context.Database.Migrate()` at startup (not recommended for production — use CI/CD scripts instead).

**Change tracking**: The `ChangeTracker` monitors entities retrieved from the database. On `SaveChanges()`, it compares current property values against the snapshot taken at query time and generates the appropriate SQL (INSERT/UPDATE/DELETE). For read-only queries, use `.AsNoTracking()` to skip tracking overhead.

**Performance optimizations:**

- **Split queries** (`.AsSplitQuery()`) for queries with multiple collection includes to avoid cartesian explosion.
- **Compiled queries** (`EF.CompileAsyncQuery`) for hot-path queries executed repeatedly.
- **Projection** (`Select`) instead of loading full entities when only a subset of columns is needed.
- **Batch operations** — EF Core 7+ supports `ExecuteUpdate` and `ExecuteDelete` for bulk operations without loading entities.
- **Connection resiliency** — configure retry strategies via `EnableRetryOnFailure` for transient database errors.

---

## 6. What patterns do you use to handle distributed transactions and data consistency across microservices?

Distributed two-phase commit (2PC) is generally avoided in microservice architectures due to tight coupling and poor availability characteristics. Instead:

**Saga Pattern**: Orchestrate a sequence of local transactions across services. Each step has a compensating action for rollback. Two approaches:
- **Choreography**: Services publish domain events; other services react. Simple but harder to track overall flow.
- **Orchestration**: A central orchestrator directs the saga steps. Easier to reason about and debug.

**Outbox Pattern**: Write the domain event to an `Outbox` table in the same database transaction as the business data. A background process (or CDC tool like Debezium) publishes the events to the message broker. This guarantees at-least-once delivery without distributed transactions.

**Idempotent consumers**: Since at-least-once delivery means duplicates are possible, every consumer must handle reprocessing safely — typically via an `IdempotencyKey` or `ProcessedMessages` table.

**Implementation in .NET**: Use MassTransit or NServiceBus for saga orchestration, outbox support, and retry policies out of the box.

---

## 7. How do you implement authentication and authorization in ASP.NET Core? Compare JWT and cookie-based approaches.

**Authentication** verifies identity; **authorization** determines access.

**JWT (Bearer tokens):**
- Stateless — the server validates the token signature without a session store.
- Ideal for APIs consumed by SPAs, mobile apps, or other services.
- Store tokens securely (HttpOnly cookies or secure storage, never `localStorage`).
- Use short-lived access tokens (5–15 min) with refresh tokens for rotation.

**Cookie-based:**
- Server creates a session; the browser sends the cookie automatically.
- Built-in CSRF protection with anti-forgery tokens.
- Better for server-rendered apps (Razor Pages, MVC).

**Authorization layers in ASP.NET Core:**
- **Role-based**: `[Authorize(Roles = "Admin")]` — simple but coarse.
- **Policy-based**: `[Authorize(Policy = "CanEditOrders")]` — flexible, composable requirements.
- **Resource-based**: Inject `IAuthorizationService` and evaluate policies against a specific resource instance (e.g., "can this user edit *this* order?").

For multi-tenant systems, implement a custom `IAuthorizationHandler` that validates tenant context on every request.

---

## 8. What is the CQRS pattern and when is it appropriate? How would you implement it in .NET?

**CQRS (Command Query Responsibility Segregation)** separates the read model from the write model. Commands mutate state; queries read state — through different models, and potentially different data stores.

**When to use:**
- Read and write workloads have very different scaling or optimization needs.
- The domain is complex and benefits from a rich write model (DDD aggregates) while reads need flat, denormalized projections.
- Event sourcing is in play.

**When NOT to use:**
- Simple CRUD applications — CQRS adds unnecessary complexity.
- Small teams without the bandwidth to maintain two models.

**Implementation in .NET:**

```csharp
// Command side — uses MediatR
public record CreateOrderCommand(string CustomerId, List<OrderItem> Items) : IRequest<Guid>;

public class CreateOrderHandler : IRequestHandler<CreateOrderCommand, Guid>
{
    public async Task<Guid> Handle(CreateOrderCommand cmd, CancellationToken ct)
    {
        // Domain logic, validation, persist via EF Core
    }
}

// Query side — thin read model, possibly Dapper for performance
public record GetOrderQuery(Guid OrderId) : IRequest<OrderDto>;
```

Use MediatR for in-process command/query dispatch. For full CQRS with separate read stores, project domain events into read-optimized tables or a search index (Elasticsearch).

---

## 9. How do you approach logging, observability, and health checks in a production .NET application?

**Structured logging**: Use `ILogger<T>` with Serilog or NLog as the provider. Log structured data, not interpolated strings:

```csharp
// Good — structured, searchable
logger.LogInformation("Order {OrderId} placed by {CustomerId}", orderId, customerId);

// Bad — unstructured, not searchable
logger.LogInformation($"Order {orderId} placed by {customerId}");
```

**Distributed tracing**: Use `System.Diagnostics.Activity` and OpenTelemetry SDK to propagate trace context across services. Export to Jaeger, Zipkin, or an OTLP-compatible backend. Correlation IDs should flow through HTTP headers, message bus headers, and log scopes.

**Metrics**: Expose counters, histograms, and gauges via `System.Diagnostics.Metrics`. Scrape with Prometheus or push to OTLP.

**Health checks**: Register via `builder.Services.AddHealthChecks()`:
- **Liveness** (`/health/live`): Is the process running? Used by the orchestrator to restart the container.
- **Readiness** (`/health/ready`): Can it serve traffic? Checks database connectivity, downstream dependencies.
- Custom health checks implement `IHealthCheck` and return `Healthy`, `Degraded`, or `Unhealthy`.

---

## 10. How do you design and optimize a caching strategy in a .NET backend?

**Multi-layer caching:**

1. **In-memory cache** (`IMemoryCache`): Fastest, per-instance. Good for reference data that rarely changes.
2. **Distributed cache** (`IDistributedCache` backed by Redis or SQL Server): Shared across instances. Essential in load-balanced or containerized deployments.
3. **Response caching / Output caching**: ASP.NET Core 7+ output caching middleware caches entire responses with tag-based invalidation.

**Cache invalidation strategies:**
- **Time-based (TTL)**: Simple, eventual consistency. Use short TTLs for volatile data.
- **Event-driven**: Publish a cache invalidation event when data changes (via Redis Pub/Sub or a message broker).
- **Cache-aside pattern**: Application checks cache first, loads from DB on miss, then populates cache.

**Common pitfalls:**
- **Cache stampede**: When a popular cache key expires, many concurrent requests hit the DB simultaneously. Mitigate with `SemaphoreSlim` or libraries like `FusionCache` that support stampede protection.
- **Stale data**: Always consider whether the business can tolerate eventual consistency for the cached entity.
- **Serialization overhead**: For distributed caches, use efficient serializers (MessagePack, protobuf) instead of JSON.

---

## 11. What are the key principles of Domain-Driven Design and how do you apply them in a .NET solution?

**Strategic patterns:**
- **Bounded Context**: Each microservice or module owns a specific subdomain with its own ubiquitous language and models. An "Order" in the Sales context is different from an "Order" in Shipping.
- **Context Mapping**: Define relationships between bounded contexts — Anti-Corruption Layer, Shared Kernel, Conformist, etc.

**Tactical patterns:**
- **Aggregates**: A cluster of entities and value objects with a single root entity that enforces invariants. All writes go through the aggregate root.
- **Value Objects**: Immutable types defined by their attributes, not identity (e.g., `Money`, `Address`). Implement as `record` in C#.
- **Domain Events**: Signal that something meaningful happened (`OrderPlaced`, `PaymentReceived`). Raised by aggregates, handled by application services or other bounded contexts.
- **Repositories**: Abstractions over data access, one per aggregate root. The repository returns and persists whole aggregates.

**Project structure:**

```
src/
  Sales.Domain/          # Entities, value objects, domain events, repository interfaces
  Sales.Application/     # Use cases, commands, queries, DTOs
  Sales.Infrastructure/  # EF Core, external service clients, repository implementations
  Sales.API/             # Controllers, middleware, composition root
```

The Domain layer has zero dependencies on infrastructure — the Dependency Inversion Principle ensures infrastructure depends on domain abstractions.

---

## 12. How do you approach performance profiling, bottleneck identification, and optimization in a .NET application?

**Profiling workflow:**

1. **Measure first** — never optimize based on assumptions. Use BenchmarkDotNet for micro-benchmarks and Application Insights / Datadog APM for production profiling.
2. **Identify the bottleneck** — CPU, memory, I/O, or network? Use `dotnet-counters`, `dotnet-trace`, and `dotnet-dump` for runtime diagnostics.
3. **Optimize the critical path** — focus on the 20% of code that causes 80% of latency.

**Common optimization techniques:**

- **Reduce allocations**: Use `Span<T>`, `stackalloc`, `ArrayPool<T>`, and `string.Create` to minimize GC pressure. Profile with `dotnet-gcdump`.
- **Async I/O everywhere**: Never block threads on I/O. Use `async/await` for database calls, HTTP calls, and file operations.
- **Connection pooling**: Reuse `HttpClient` via `IHttpClientFactory`. Configure EF Core connection pool size.
- **Query optimization**: Use SQL profiler or EF Core logging to identify N+1 queries, missing indexes, and unnecessary data loading.
- **Response compression**: Enable Brotli/gzip compression for API responses.
- **Minimize serialization**: Use `System.Text.Json` source generators for AOT-friendly, allocation-free serialization.

**Load testing**: Use tools like k6, NBomber, or Apache JMeter to simulate production traffic patterns and identify breaking points before deployment.

---

*These questions cover the breadth expected of a Senior .NET Developer — from framework internals and async patterns to distributed systems architecture and production operational excellence.*
