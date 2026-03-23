# 🧑‍💻 Senior Fullstack Developer — Interview Q&A
> Stack: .NET Core (C#) · ReactJS · TypeScript  
> Góc nhìn: Backend Architect · Software Architect · Frontend Developer  
> Ngôn ngữ: Tiếng Việt · Thuật ngữ kỹ thuật giữ nguyên

---

## 1. Strong Proficiency in C#, JavaScript/TypeScript & Modern Web Technologies

---

**Q1. Sự khác biệt cốt lõi giữa `async/await` trong C# và trong JavaScript/TypeScript là gì?**

**A:**  
Cả hai đều là syntactic sugar trên nền tảng Promise/Task, nhưng runtime khác nhau căn bản:

- **C#**: `async/await` dựa trên `Task`/`ValueTask`. Thread pool thực sự được giải phóng khi `await` một I/O operation. Có `ConfigureAwait(false)` để tránh deadlock trong context synchronization (ví dụ: ASP.NET classic).
- **JavaScript**: Single-threaded, event loop. `await` không giải phóng thread (vì chỉ có một), nó chỉ yield lại control cho event loop. Không có deadlock theo nghĩa của C#.

Trade-off: C# có thể tận dụng true parallelism qua `Task.WhenAll`, còn JS thực chất là concurrent, không parallel.

---

**Q2. Trong C#, `ValueTask` khác `Task` như thế nào và khi nào nên dùng `ValueTask`?**

**A:**  
- `Task` là reference type, luôn allocate trên heap dù operation hoàn thành synchronously.
- `ValueTask` là struct, tránh allocation khi operation hoàn thành ngay (cache hit, hot path).

Dùng `ValueTask` khi:
1. Method thường xuyên return synchronously (ví dụ: đọc từ in-memory cache).
2. High-frequency call path, nơi allocation pressure ảnh hưởng GC.

**Không** dùng `ValueTask` nếu: await nhiều lần, hoặc convert sang `Task` — vì `ValueTask` không hỗ trợ multiple awaits.

---

**Q3. TypeScript `unknown` vs `any` — khi nào dùng cái nào?**

**A:**  
- `any`: bypass hoàn toàn type checker. Dùng khi migrate JS cũ hoặc prototype nhanh — đây là technical debt.
- `unknown`: type-safe counterpart của `any`. Phải narrow type trước khi sử dụng (`typeof`, `instanceof`, type guard).

```typescript
function parse(input: unknown) {
  if (typeof input === 'string') {
    return input.toUpperCase(); // OK
  }
  throw new Error('Invalid input');
}
```

Rule of thumb: `any` là tắt type system, `unknown` là dùng type system đúng cách cho dữ liệu không rõ kiểu.

---

**Q4. Generics trong C# và TypeScript hoạt động khác nhau ở điểm gì quan trọng nhất?**

**A:**  
- **C# Generics**: reified — type parameter tồn tại ở runtime. `typeof(T)` cho ra real type, có thể dùng `T` với reflection.
- **TypeScript Generics**: erased — chỉ tồn tại compile-time. Sau khi compile ra JS, `T` biến mất hoàn toàn.

Hệ quả thực tế: trong C# có thể viết `new T()` (với constraint `where T : new()`), còn TypeScript không thể instantiate generic type trực tiếp mà phải truyền factory function.

---

**Q5. Giải thích `record` type trong C# và khi nào ưu tiên dùng thay `class`?**

**A:**  
`record` (C# 9+) là reference type với value-based equality, immutability mặc định, và auto-generated `ToString`, `Deconstruct`, `with` expression.

```csharp
record ProductDto(string Name, decimal Price);
var updated = product with { Price = 99.9m };
```

Dùng `record` khi:
- DTO, Value Object trong DDD.
- Immutable data transfer giữa layers.
- Cần structural equality (so sánh theo giá trị, không theo reference).

Dùng `class` khi: cần mutable state, identity semantics, hoặc complex behavior.

---

**Q6. React Concurrent Mode mang lại gì so với legacy rendering?**

**A:**  
Concurrent Mode cho phép React "pause, interrupt, resume" rendering. Key features:

- **`useTransition`**: mark state update là non-urgent, UI không bị block.
- **`useDeferredValue`**: defer re-render của slow subtree.
- **Automatic Batching** (React 18): batch updates trong async callbacks, không chỉ event handlers.
- **Suspense for data fetching**: tích hợp native với server components.

Thực tế: input field typing mượt mà dù list 10k items đang re-render, vì React ưu tiên user input.

---

**Q7. `IEnumerable<T>` vs `IQueryable<T>` trong C# — sai lầm phổ biến nhất là gì?**

**A:**  
- `IEnumerable<T>`: execute in-memory (LINQ to Objects).
- `IQueryable<T>`: expression tree, translate sang SQL (LINQ to EF/SQL).

Sai lầm phổ biến: gọi `.ToList()` hoặc cast sang `IEnumerable` **trước** khi filter/sort → load toàn bộ table vào memory, rồi mới filter ở tầng application.

```csharp
// ❌ BAD: load all, then filter in memory
IEnumerable<Product> products = _dbContext.Products;
var result = products.Where(p => p.Price > 100).ToList();

// ✅ GOOD: translate to SQL WHERE clause
IQueryable<Product> products = _dbContext.Products;
var result = products.Where(p => p.Price > 100).ToList();
```

---

**Q8. Trong Next.js 14+, khi nào dùng Server Component vs Client Component?**

**A:**  
- **Server Component** (default): render trên server, zero JS bundle, có thể access database/API trực tiếp. Dùng cho: static content, data fetching, SEO-critical pages.
- **Client Component** (`"use client"`): cần interactivity, browser APIs, hooks như `useState`, `useEffect`.

Chiến lược tối ưu: "push client down" — giữ component tree là Server Component tối đa, chỉ tạo Client Component ở leaf nodes cần tương tác. Tránh đưa data fetching vào Client Component không cần thiết.

---

## 2. Software Design Patterns, Data Structures & Algorithms

---

**Q1. Giải thích Repository Pattern và Unit of Work — tại sao nhiều dự án implement sai?**

**A:**  
- **Repository**: abstraction over data access cho một aggregate root.
- **Unit of Work**: đảm bảo nhiều repository operations trong một transaction.

Implement sai phổ biến:
1. **Generic Repository wrapping DbContext**: `IRepository<T>` chỉ wrap lại EF Core mà không thêm value, vì `DbContext` đã là Unit of Work + Repository.
2. **Leaky abstraction**: return `IQueryable<T>` từ Repository — buộc caller biết về EF internals.
3. **Over-engineering**: project nhỏ dùng pattern này chỉ để "đúng pattern".

Đúng hơn: Repository nên ẩn query complexity, return domain objects, không expose `IQueryable`.

---

**Q2. CQRS pattern giải quyết vấn đề gì và trade-off là gì?**

**A:**  
CQRS tách biệt read model (Query) và write model (Command):

- **Command**: thay đổi state, không return data (hoặc chỉ return ID).
- **Query**: read-only, có thể optimize riêng (denormalized view, read replica).

**Lợi ích**: scale read và write independently, simplify domain model (write model focus vào business logic).

**Trade-off**:
- Eventual consistency nếu dùng event sourcing.
- Complexity tăng đáng kể — overkill cho CRUD apps đơn giản.
- Cần infrastructure hỗ trợ (MediatR, message bus).

Áp dụng khi: read/write workload chênh lệch lớn, hoặc domain logic phức tạp cần tách bạch.

---

**Q3. Khi nào dùng Mediator Pattern và tại sao MediatR phổ biến trong .NET?**

**A:**  
Mediator giảm coupling giữa components bằng cách định tuyến request qua một central handler.

MediatR phổ biến vì:
1. Tích hợp tự nhiên với CQRS — mỗi Command/Query có handler riêng.
2. Pipeline behaviors: cross-cutting concerns (logging, validation, transaction) implement một lần.
3. Vertical slice architecture: mỗi feature là một folder độc lập (request + handler + validator).

```csharp
// Pipeline behavior for validation
public class ValidationBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
{
    public async Task<TResponse> Handle(TRequest request, RequestHandlerDelegate<TResponse> next, ...)
    {
        // validate before handler
        await _validator.ValidateAndThrowAsync(request);
        return await next();
    }
}
```

---

**Q4. Giải thích Dependency Injection Lifetime: Singleton, Scoped, Transient — sai lầm nghiêm trọng nhất?**

**A:**  
- **Singleton**: một instance cho toàn bộ app lifetime.
- **Scoped**: một instance per HTTP request.
- **Transient**: mỗi lần resolve là một instance mới.

**Captive dependency** — sai lầm nghiêm trọng nhất: inject Scoped/Transient service vào Singleton. Singleton "giữ" instance của Scoped service, dẫn đến shared state across requests, data leak, race condition.

```csharp
// ❌ Captive dependency
public class MySingleton // Singleton
{
    public MySingleton(IScopedService scoped) { } // BUG: scoped captured forever
}
```

ASP.NET Core detect được lỗi này và throw exception ở startup (scope validation).

---

**Q5. B-Tree Index hoạt động như thế nào và tại sao nó phù hợp cho database?**

**A:**  
B-Tree là balanced tree, mỗi node chứa nhiều keys (page-aligned với disk block size, thường 8–16KB).

Ưu điểm cho database:
- **O(log n)** cho point query và range query.
- **Sequential I/O** cho range scan — keys ở leaf nodes được liên kết (B+ Tree variant).
- Self-balancing sau insert/delete.

Khi nào index kém hiệu quả:
- Low cardinality column (boolean, gender) — full scan nhanh hơn.
- Frequent write: insert/delete trigger re-balancing, tốn I/O.
- Leading column rule: composite index `(A, B, C)` không dùng được cho query chỉ có `B` hoặc `C`.

---

**Q6. Giải thích Big O của các operation phổ biến — `Dictionary<K,V>` trong C# dùng cấu trúc gì?**

**A:**  
`Dictionary<K,V>` dùng **hash table** với separate chaining (linked list hoặc tree khi collision nhiều).

| Operation | Average | Worst Case |
|-----------|---------|------------|
| Lookup    | O(1)    | O(n) — hash collision |
| Insert    | O(1) amortized | O(n) — resize |
| Delete    | O(1)    | O(n) |

Resize xảy ra khi load factor vượt ngưỡng (~0.72), allocate bucket array mới và rehash toàn bộ.

Thực tế: `SortedDictionary<K,V>` dùng Red-Black Tree, mọi operation là O(log n) nhưng key luôn sorted — dùng khi cần ordered iteration.

---

**Q7. Khi nào dùng Queue vs Stack và ví dụ thực tế trong backend?**

**A:**  
- **Queue (FIFO)**: xử lý tuần tự theo thứ tự đến. Dùng cho: background job queue (Hangfire, RabbitMQ), rate limiting request, BFS graph traversal.
- **Stack (LIFO)**: undo/redo, call stack, DFS, expression parsing, nested tag validation.

Thực tế trong .NET backend:
```csharp
// Validate nested HTML tags
var stack = new Stack<string>();
// ... push opening tags, pop and verify on closing tags
```

Message broker (RabbitMQ/Kafka) về bản chất là distributed queue — đảm bảo message ordering và at-least-once delivery.

---

**Q8. Strategy Pattern vs Policy Pattern (Polly) trong .NET — khác nhau ở điểm nào?**

**A:**  
- **Strategy Pattern** (GoF): encapsulate algorithm/behavior, swap at runtime. Ví dụ: payment processor — `MoMoStrategy`, `VNPayStrategy` implement `IPaymentStrategy`.
- **Polly Resilience Policy**: không phải design pattern theo nghĩa GoF, mà là resilience library implement retry, circuit breaker, timeout, fallback.

Kết hợp: Strategy cho business logic, Polly cho infrastructure resilience:

```csharp
// Strategy cho business
IPaymentStrategy strategy = paymentFactory.GetStrategy(method);
// Polly cho resilience
var result = await _retryPolicy.ExecuteAsync(() => strategy.ProcessAsync(order));
```

Circuit Breaker quan trọng: sau N failures, "mở mạch" (fast fail) để tránh cascade failure.

---

## 3. RESTful APIs, JSON & Asynchronous Programming

---

**Q1. REST constraints là gì và hầu hết API gọi là "REST" thực ra vi phạm ở đâu?**

**A:**  
REST gồm 6 constraints của Fielding: Client-Server, Stateless, Cacheable, Uniform Interface, Layered System, Code on Demand (optional).

**Uniform Interface** là constraint bị vi phạm nhiều nhất, cụ thể **HATEOAS** (Hypermedia As The Engine Of Application State) — response phải chứa links để client tự discover next actions.

Thực tế: 99% API được gọi là "REST" thực ra là **HTTP API** hoặc **RPC over HTTP**. Điều này không sai, chỉ cần honest: đừng tự claim "RESTful" nếu không implement HATEOAS.

Richardson Maturity Model: Level 0 (one endpoint) → Level 1 (resources) → Level 2 (HTTP verbs) → Level 3 (HATEOAS). Hầu hết production API dừng ở Level 2.

---

**Q2. API Versioning — các chiến lược và trade-off?**

**A:**  
| Chiến lược | Ví dụ | Pros | Cons |
|------------|-------|------|------|
| URL segment | `/api/v1/products` | Explicit, cacheable, easy routing | URL không "clean" |
| Query string | `?version=1` | Backward compat | Dễ bỏ qua, không cacheable |
| Header | `Api-Version: 1` | Clean URL | Hidden, khó test trực tiếp |
| Media type | `Accept: application/vnd.api+json;v=1` | Truly RESTful | Complex client implementation |

**Khuyến nghị production**: URL segment cho public API (developer-friendly), header versioning cho internal/partner API.

Quan trọng: deprecation policy — thông báo trước ít nhất 6 tháng, return `Sunset` header.

---

**Q3. Thiết kế pagination API — Cursor-based vs Offset-based?**

**A:**  
- **Offset (`?page=2&limit=20`)**: đơn giản, support random access. Vấn đề: `OFFSET 10000` phải skip 10000 rows → chậm. Data shift nếu có insert/delete giữa pages.
- **Cursor (`?after=eyJpZCI6MTIzfQ`)**: encode position (thường là ID + timestamp), O(1) seek bằng index. Không skip rows. Nhược điểm: không random access, opaque cursor.

```json
{
  "data": [...],
  "pagination": {
    "next_cursor": "eyJpZCI6MTIzfQ",
    "has_more": true
  }
}
```

**Rule of thumb**: dùng cursor-based cho real-time feed (social, notification), offset cho admin dashboard cần "jump to page".

---

**Q4. Xử lý idempotency trong REST API — tại sao critical cho payment API?**

**A:**  
Idempotency: gọi nhiều lần có kết quả như gọi một lần.

- `GET`, `PUT`, `DELETE`: idempotent by definition.
- `POST`: không idempotent — duplicate request = duplicate resource.

Implement idempotency key:
```
POST /api/payments
Idempotency-Key: uuid-v4-client-generated

# Server lưu key + result vào cache (Redis, TTL 24h)
# Request thứ 2 cùng key → return cached result, không process lại
```

Critical cho payment: network timeout sau khi server process nhưng trước khi response về client. Client retry → duplicate charge nếu không có idempotency.

MoMo và VNPay đều yêu cầu `requestId` / `vnp_TxnRef` unique per transaction — đây chính là idempotency key.

---

**Q5. JSON Serialization gotchas trong .NET — `System.Text.Json` vs `Newtonsoft.Json`?**

**A:**  
`System.Text.Json` (built-in, high performance) vs `Newtonsoft.Json` (feature-rich, mature):

| Issue | System.Text.Json | Newtonsoft |
|-------|-----------------|------------|
| Camel case | Cần configure | Auto |
| Circular reference | Throw exception | Handle |
| Dynamic/anonymous | Hạn chế | Tốt |
| Custom converter | Verbose | Flexible |
| Performance | ~2x nhanh hơn | Baseline |

Gotchas thực tế với `System.Text.Json`:
1. `[JsonIgnore]` không ignore khi deserializing theo default.
2. `readonly struct` fields không serialize.
3. Polymorphic serialization cần `[JsonDerivedType]` (C# 7+).

---

**Q6. Xử lý long-running API calls — WebSocket vs SSE vs Polling?**

**A:**  
| Kỹ thuật | Use case | Pros | Cons |
|----------|----------|------|------|
| Short Polling | Simple status check | Easy implement | Wasteful (empty responses) |
| Long Polling | Notification, chat | Simpler than WS | Connection overhead |
| SSE | Server-push, one-way stream | Simple, HTTP/2 multiplexing | One-way only |
| WebSocket | Bidirectional real-time | Full-duplex | Complex, stateful |

**Pattern tốt nhất cho job API**:
1. `POST /jobs` → return `202 Accepted` + `jobId`.
2. `GET /jobs/{id}/status` → polling hoặc SSE để nhận progress.
3. Webhook callback khi hoàn thành.

---

**Q7. Rate Limiting trong ASP.NET Core — các algorithm và implementation?**

**A:**  
.NET 7+ có built-in `RateLimiter`:

- **Fixed Window**: đếm request trong window cố định (vd: 100 req/phút). Burst issue: 100 request trong giây cuối của window 1 + 100 giây đầu window 2 = 200 request liền tiếp.
- **Sliding Window**: smooth hơn, track requests theo rolling window.
- **Token Bucket**: refill token theo rate, allow burst trong giới hạn.
- **Concurrency Limiter**: giới hạn concurrent request, không theo time.

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddSlidingWindowLimiter("api", opt =>
    {
        opt.Window = TimeSpan.FromMinutes(1);
        opt.PermitLimit = 100;
        opt.SegmentsPerWindow = 4;
    });
});
```

Production: kết hợp Redis để distributed rate limiting across multiple instances.

---

**Q8. Deadlock trong async code — tại sao `.Result` và `.Wait()` nguy hiểm trong ASP.NET?**

**A:**  
Classic deadlock:

```csharp
// ❌ DEADLOCK trong ASP.NET classic / WinForms
public string GetData()
{
    return _httpClient.GetStringAsync(url).Result; // Blocks current thread
}
```

Nguyên nhân: `SynchronizationContext` trong ASP.NET classic/WinForms yêu cầu continuation chạy trên original thread. Nhưng thread đó đang bị block bởi `.Result` → deadlock.

**Fix**: `async` all the way, hoặc dùng `ConfigureAwait(false)` nếu không cần context.

ASP.NET Core **không có** `SynchronizationContext`, nên deadlock ít xảy ra hơn — nhưng `.Result` vẫn tệ vì nó block ThreadPool thread, giảm throughput.

---

## 4. Relational Databases (SQL Server, PostgreSQL) & ORM Frameworks

---

**Q1. Giải thích Execution Plan và cách đọc khi query chậm?**

**A:**  
Execution Plan là kế hoạch của query optimizer để thực thi SQL. Đọc từ **right to left, bottom to top**.

Các operator cần chú ý:
- **Table Scan / Clustered Index Scan**: đọc toàn bộ table — red flag nếu table lớn.
- **Index Seek**: tìm kiếm theo index — tốt.
- **Key Lookup (RID Lookup)**: sau index seek, quay lại clustered index để lấy thêm columns — tốn cost, cân nhắc covering index.
- **Hash Match / Merge Join**: join strategy. Nested Loop tốt cho small dataset, Hash Match cho large.
- **Sort**: tốn memory, có thể trigger spill to disk.

Công cụ: `SET STATISTICS IO ON` (SQL Server), `EXPLAIN ANALYZE` (PostgreSQL), `Query Store` (SQL Server 2016+).

---

**Q2. Index Covering và Composite Index — thứ tự columns quan trọng như thế nào?**

**A:**  
**Covering Index**: index chứa đủ columns mà query cần → không cần quay lại table.

```sql
-- Query
SELECT name, email FROM users WHERE status = 'active' ORDER BY created_at DESC;

-- Covering index
CREATE INDEX idx_users_covering ON users(status, created_at DESC) INCLUDE (name, email);
```

**Composite Index — thứ tự columns**:
- Equality conditions trước, range conditions sau.
- Most selective column (high cardinality) thường đứng trước cho point queries.
- Rule: index `(A, B, C)` dùng được cho predicate `A`, `A+B`, `A+B+C` — không dùng được cho chỉ `B` hoặc `C` (leading column rule).

---

**Q3. Transaction Isolation Levels — khi nào phantom read, dirty read xảy ra?**

**A:**  

| Level | Dirty Read | Non-repeatable Read | Phantom Read |
|-------|-----------|-------------------|--------------|
| Read Uncommitted | ✅ Có | ✅ Có | ✅ Có |
| Read Committed | ❌ Không | ✅ Có | ✅ Có |
| Repeatable Read | ❌ | ❌ | ✅ Có (MySQL InnoDB: không) |
| Serializable | ❌ | ❌ | ❌ |

SQL Server default: **Read Committed**.  
PostgreSQL default: **Read Committed** nhưng dùng MVCC — không block reader.

**Snapshot Isolation** (SQL Server): readers không block writers, writers không block readers — dùng row versioning. Trade-off: tốn tempdb storage.

---

**Q4. N+1 Query Problem trong EF Core — phát hiện và fix như thế nào?**

**A:**  
N+1: load 1 list → với mỗi item, query thêm related data = 1 + N queries.

```csharp
// ❌ N+1
var orders = await _context.Orders.ToListAsync();
foreach (var order in orders) {
    var items = order.Items; // Lazy load → N queries
}

// ✅ Eager loading
var orders = await _context.Orders
    .Include(o => o.Items)
    .ThenInclude(i => i.Product)
    .ToListAsync();

// ✅ Projection (tốt hơn nếu không cần toàn bộ entity)
var result = await _context.Orders
    .Select(o => new OrderDto {
        Id = o.Id,
        ItemCount = o.Items.Count
    }).ToListAsync();
```

Phát hiện: bật `LogTo` của EF Core hoặc dùng MiniProfiler để đếm queries per request.

---

**Q5. Stored Procedure vs Application-side logic — khi nào dùng SP trong 2024?**

**A:**  
**Stored Procedure phù hợp**:
- Bulk operations (ETL, batch update hàng triệu rows).
- Cần atomic multi-step operation cực kỳ performance-sensitive.
- Legacy system, DBA team quản lý database logic.

**Application-side (ORM/Raw SQL) phù hợp**:
- Logic phức tạp, thay đổi thường xuyên.
- Unit testable.
- Version control rõ ràng (SP trong DB rất khó diff/review).
- Horizontal scaling — SP gắn với database instance.

**Quan điểm thực tế**: với modern microservices + cloud database, ưu tiên application-side. SP dùng cho batch job và data migration, không nên dùng cho business logic.

---

**Q6. Database Sharding vs Partitioning — khác nhau và khi nào cần?**

**A:**  
- **Partitioning**: chia data trong một database (horizontal partition = nhiều physical files). Database engine tự manage, transparent với application.
- **Sharding**: chia data across nhiều database instances. Application phải biết routing.

**Partition**: dùng cho time-series data (partition by month), archive old data, improve query performance cho large tables.

**Sharding cần khi**:
- Single database instance đã đạt giới hạn vertical scaling.
- Write throughput quá lớn.

**Shard key selection** là critical: chọn key có high cardinality, uniform distribution, và align với query pattern. Hot shard là failure mode phổ biến.

---

**Q7. EF Core migrations trong team environment — quản lý conflict như thế nào?**

**A:**  
Migration conflict xảy ra khi 2 developer tạo migration cùng lúc — cả hai build on cùng snapshot, migration ID collision.

**Quy trình tránh conflict**:
1. Mỗi PR chỉ có 1 migration, merge tuần tự.
2. Sau khi merge main, developer khác phải `Remove-Migration` → merge → `Add-Migration` lại.
3. Đặt migration name có semantic: `AddProductPriceIndex` thay vì timestamp-only.

**Production deployment**:
```bash
dotnet ef database update --connection "..." # Sai với multiple instances
```

Dùng `MigrateDatabaseAtStartup` hoặc dedicated migration job, không chạy migration trong nhiều app instances đồng thời (race condition).

---

**Q8. Connection Pooling — cấu hình sai gây ra vấn đề gì trong production?**

**A:**  
Connection Pool tái sử dụng physical database connections. Vấn đề phổ biến:

**Pool exhaustion**: mọi connection trong pool đang bận → request mới timeout. Nguyên nhân:
- Không dispose `SqlConnection`/`DbContext` (thiếu `using`).
- Long-running transaction giữ connection.
- Pool size quá nhỏ so với concurrency.

```csharp
// ❌ Connection leak
var conn = new SqlConnection(connStr);
conn.Open();
// ... quên Dispose

// ✅ Always dispose
await using var conn = new SqlConnection(connStr);
```

**Cấu hình**:
```
Server=...;Max Pool Size=100;Min Pool Size=10;Connection Timeout=30;
```

Monitoring: `sys.dm_exec_sessions` (SQL Server), `pg_stat_activity` (PostgreSQL) để track active connections.

---

## 5. Security Principles, Data Protection & Compliance

---

**Q1. Giải thích OAuth2 flow và tại sao không nên tự implement authentication?**

**A:**  
OAuth2 là authorization framework, không phải authentication. OpenID Connect (OIDC) bổ sung authentication layer trên OAuth2.

**Authorization Code Flow + PKCE** (recommended cho web app):
1. Client redirect user đến Authorization Server kèm `code_challenge`.
2. User authenticate → AS redirect về với `authorization_code`.
3. Client exchange code (kèm `code_verifier`) lấy `access_token` + `id_token`.

**Tại sao không tự implement**:
- Token storage, rotation, revocation là extremely complex.
- Timing attack trong cryptographic comparison.
- Session fixation, CSRF, token leakage qua referrer.

Dùng: **Keycloak**, **Auth0**, **Azure AD B2C**, hoặc ASP.NET Core Identity với proven library.

---

**Q2. JWT — điểm yếu bảo mật và best practices?**

**A:**  
JWT gồm header.payload.signature — payload **không encrypted** (chỉ base64), ai có token đều đọc được.

**Điểm yếu phổ biến**:
1. `alg: "none"` attack — verify signature bị bypass nếu server accept algorithm "none".
2. Lưu JWT trong `localStorage` → XSS attack steal token.
3. JWT không thể revoke trước khi expire → stolen token vẫn valid.

**Best practices**:
- Lưu trong `HttpOnly Secure Cookie` → không accessible qua JS.
- Short expiry `access_token` (15 phút) + `refresh_token` với rotation.
- Blacklist revoked JWTs trong Redis (đánh đổi: cần lookup per request).
- Validate `iss`, `aud`, `exp`, `nbf` claims.
- Dùng RS256 (asymmetric) thay HS256 cho distributed systems.

---

**Q3. SQL Injection vẫn xảy ra năm 2024 — tại sao và cách phòng chống đúng?**

**A:**  
Vẫn xảy ra vì:
- Dynamic SQL concatenation trong legacy code / raw SQL.
- ORM được bypass để "optimize" query.
- Stored procedure dùng `EXEC` với string concatenation.

**Phòng chống**:
```csharp
// ❌ VULNERABLE
var query = $"SELECT * FROM users WHERE name = '{userInput}'";

// ✅ Parameterized query
var users = await _context.Users
    .Where(u => u.Name == userInput) // EF Core parameterized
    .ToListAsync();

// ✅ Raw SQL với parameters
var users = await _context.Users
    .FromSqlRaw("SELECT * FROM users WHERE name = {0}", userInput)
    .ToListAsync();
```

**Không đủ**: input sanitization / escaping — vẫn có edge cases. Parameterized query là defense duy nhất đáng tin cậy.

---

**Q4. CORS — misconfiguration phổ biến nhất trong production?**

**A:**  
CORS là browser security feature — server declare ai được phép call API từ browser.

**Misconfiguration nguy hiểm**:

```csharp
// ❌ Wildcard + credentials = không work + security risk
app.UseCors(x => x
    .AllowAnyOrigin()       // Không thể kết hợp với AllowCredentials
    .AllowCredentials()
    .AllowAnyMethod());

// ❌ Reflect Origin không validate (trust any origin)
if (request.Headers["Origin"] != null)
    response.Headers["Access-Control-Allow-Origin"] = request.Headers["Origin"];

// ✅ Explicit whitelist
app.UseCors(x => x
    .WithOrigins("https://app.savvysale.com", "https://admin.savvysale.com")
    .AllowCredentials()
    .AllowAnyMethod()
    .WithHeaders("Authorization", "Content-Type"));
```

CORS không protect server-to-server calls — chỉ là browser enforcement.

---

**Q5. Lưu trữ password — tại sao MD5/SHA1 không đủ và bcrypt/Argon2 hoạt động như thế nào?**

**A:**  
MD5/SHA1: fast hash — brute force 10 tỷ hashes/giây trên GPU hiện đại.

**bcrypt/Argon2**: adaptive hash — designed to be slow:
- **Work factor (cost)**: tăng iteration count khi hardware mạnh hơn.
- **Salt built-in**: mỗi password hash unique, rainbow table vô dụng.
- **Memory-hard (Argon2)**: yêu cầu large memory → GPU attack kém hiệu quả.

```csharp
// .NET Identity dùng PBKDF2 by default
// Hoặc dùng BCrypt.Net
var hash = BCrypt.Net.BCrypt.HashPassword(password, workFactor: 12);
var valid = BCrypt.Net.BCrypt.Verify(password, hash);
```

**Rule of thumb**: hash time ~250–500ms là acceptable cho login, không ảnh hưởng UX nhưng làm brute force tốn kém.

---

**Q6. Data Encryption at Rest vs in Transit — implement trong .NET như thế nào?**

**A:**  
- **In Transit**: TLS 1.2+ (enforce, disable TLS 1.0/1.1). HSTS header.
- **At Rest**: mã hóa sensitive columns, backup encryption, disk encryption.

**Column-level encryption** trong .NET:
```csharp
// ASP.NET Core Data Protection API
var protector = _dataProtectionProvider.CreateProtector("PersonalData");
var encrypted = protector.Protect(plainText);      // Store this
var decrypted = protector.Unprotect(encrypted);
```

**Envelope encryption** (pattern AWS/GCP dùng):
1. Data Encryption Key (DEK) mã hóa data.
2. Key Encryption Key (KEK) trong KMS mã hóa DEK.
3. Chỉ lưu encrypted DEK cùng data.

Rotation: thay KEK mới chỉ cần re-encrypt DEK, không cần re-encrypt toàn bộ data.

---

**Q7. OWASP Top 10 — developer cần focus vào những gì nhất?**

**A:**  
Top issues developer gây ra trực tiếp:

1. **Injection** (SQL, NoSQL, Command): parameterized queries, input validation.
2. **Broken Authentication**: weak passwords, no MFA, session fixation.
3. **Sensitive Data Exposure**: log PII, return sensitive fields trong API response.
4. **Security Misconfiguration**: default credentials, verbose error messages in production, debug endpoints.
5. **Insecure Direct Object Reference (IDOR)**: `GET /orders/12345` — verify ownership, không chỉ authentication.

```csharp
// ❌ IDOR vulnerability
var order = await _context.Orders.FindAsync(orderId);

// ✅ Verify ownership
var order = await _context.Orders
    .FirstOrDefaultAsync(o => o.Id == orderId && o.UserId == currentUserId);
```

---

**Q8. GDPR / Data Compliance — developer phải làm gì cụ thể trong code?**

**A:**  
Developer chịu trách nhiệm implement:

1. **Right to Access**: API export toàn bộ data của user theo request.
2. **Right to Erasure (Right to be Forgotten)**: soft delete không đủ — cần anonymize PII hoặc hard delete, cascade qua tất cả services.
3. **Data Minimization**: không collect data không cần thiết, không log PII (IP, email trong application logs).
4. **Consent tracking**: lưu timestamp + version của consent, not just boolean.
5. **Data Breach Notification**: phải notify trong 72 giờ → cần audit log, monitoring.

```csharp
// Anonymization thay vì delete (nếu cần giữ transaction history)
user.Email = $"deleted_{user.Id}@anonymized.invalid";
user.Name = "Deleted User";
user.PhoneNumber = null;
```

---

## 6. Problem-Solving, Debugging & Troubleshooting

---

**Q1. Hệ thống production bị chậm đột ngột — quy trình debug như thế nào?**

**A:**  
**Systematic approach — không guess, measure first**:

1. **Define**: chậm ở đâu? API latency? DB? External service? → Check APM (Application Insights, Datadog).
2. **Scope**: toàn bộ endpoints hay specific? Sau deployment? Sau data growth?
3. **Database first**: check slow query log, `pg_stat_statements`, execution plans.
4. **Application**: memory leak? GC pressure? Thread pool starvation? → memory dump, CPU profiler.
5. **Infrastructure**: CPU, RAM, disk I/O, network latency → CloudWatch/Azure Monitor.
6. **Correlate với changes**: deployment, config change, traffic spike, data volume.

**Không** restart server ngay mà không collect evidence — mất trace.

---

**Q2. Memory leak trong .NET — phát hiện và phân tích dump như thế nào?**

**A:**  
Dấu hiệu: memory tăng dần, GC chạy nhiều nhưng memory không giảm, eventually `OutOfMemoryException`.

**Nguyên nhân phổ biến**:
1. Event handler không unsubscribe (publisher giữ reference đến subscriber).
2. Static collection grow indefinitely.
3. `IDisposable` không dispose.
4. Captured variable trong closure/async lambda.

**Debug**:
```bash
# Collect dump
dotnet-dump collect --process-id <pid>

# Analyze
dotnet-dump analyze <dump>
> dumpheap -stat          # Top types by memory
> dumpheap -type MyClass  # Find instances
> gcroot <address>        # Who's holding reference
```

Tools: **dotMemory**, **PerfView**, **Visual Studio Diagnostic Tools**.

---

**Q3. Deadlock trong database — phát hiện và resolve?**

**A:**  
Deadlock: Transaction A giữ Lock X, chờ Lock Y. Transaction B giữ Lock Y, chờ Lock X.

**Phát hiện**:
```sql
-- SQL Server
SELECT * FROM sys.dm_exec_requests WHERE blocking_session_id > 0;
-- Enable trace flag 1222 để log deadlock detail
DBCC TRACEON(1222, -1);

-- PostgreSQL
SELECT * FROM pg_locks JOIN pg_stat_activity USING (pid)
WHERE NOT granted;
```

**Resolve strategies**:
1. **Consistent lock ordering**: luôn lock resources theo thứ tự cố định (A trước B).
2. **Keep transactions short**: không làm user interaction trong transaction.
3. **Lower isolation level**: dùng snapshot isolation nếu business cho phép.
4. **Retry on deadlock**: detect `SqlException` error 1205, retry với backoff.

```csharp
catch (SqlException ex) when (ex.Number == 1205) // Deadlock
{
    await Task.Delay(TimeSpan.FromMilliseconds(random.Next(100, 500)));
    // retry
}
```

---

**Q4. Distributed tracing — tại sao cần và implement trong microservices như thế nào?**

**A:**  
Monolith: stack trace đủ để debug. Microservices: request đi qua 5–10 services → không biết chậm ở đâu.

**Distributed tracing**: propagate `TraceId` + `SpanId` qua HTTP headers (`traceparent` — W3C standard).

```csharp
// ASP.NET Core + OpenTelemetry
builder.Services.AddOpenTelemetry()
    .WithTracing(tracing => tracing
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .AddEntityFrameworkCoreInstrumentation()
        .AddOtlpExporter()); // Export sang Jaeger/Tempo/Datadog
```

Kết quả: Gantt chart của toàn bộ request journey. Identify bottleneck chính xác — `ProductService.GetById` tốn 800ms vì N+1 query.

---

**Q5. Giải thích cách debug race condition trong concurrent code?**

**A:**  
Race condition: kết quả phụ thuộc vào timing/ordering của concurrent operations — không reproduce consistently.

**Phát hiện**:
1. Code review: shared mutable state + multiple threads = suspect.
2. Stress test: `Task.WhenAll(Enumerable.Range(0, 1000).Select(_ => DoOperation()))`.
3. Thread Sanitizer (Linux), Helgrind (Valgrind).

**Common patterns**:
```csharp
// ❌ Race condition: check-then-act
if (!_cache.ContainsKey(key))
    _cache[key] = ComputeValue(); // Two threads can enter here

// ✅ Thread-safe với GetOrAdd
var value = _cache.GetOrAdd(key, k => ComputeValue());

// ✅ Lock khi cần atomicity
lock (_lock) {
    if (!_dict.ContainsKey(key))
        _dict[key] = ComputeValue();
}
```

Dùng `Interlocked` cho counter operations, `ReaderWriterLockSlim` cho read-heavy scenarios.

---

**Q6. Frontend performance debugging — khi React app bị lag, quy trình như thế nào?**

**A:**  
**Tools**: Chrome DevTools → Performance tab, React DevTools Profiler.

**Quy trình**:
1. **React Profiler**: identify component render nhiều nhất, thời gian render.
2. **Why re-render?**: `React.memo` bị bypass vì prop mới mỗi lần? Object/array literal inline?
3. **Chrome Performance**: check Long Tasks (>50ms), Layout Thrashing, Memory.

**Common fixes**:
```tsx
// ❌ New object reference every render
<Child config={{ timeout: 3000 }} />

// ✅ useMemo / hoist constant
const config = useMemo(() => ({ timeout: 3000 }), []);
<Child config={config} />

// ❌ Inline callback = new function every render
<Child onClick={() => handleClick(id)} />

// ✅ useCallback
const handleClick = useCallback((id) => { ... }, []);
```

---

**Q7. Production hotfix — quy trình deploy an toàn khi system đang có traffic?**

**A:**  
**Không deploy directly to production không có rollback plan**.

**Quy trình**:
1. **Hotfix branch** từ production tag, không từ main (main có thể có unreleased features).
2. **Feature flag**: bật/tắt fix mà không cần deploy.
3. **Blue-Green hoặc Canary deployment**:
   - Canary: route 5% traffic → 20% → 50% → 100%, monitor error rate.
   - Blue-Green: switch load balancer, rollback trong 30 giây.
4. **Database migration phải backward-compatible**: không drop column ngay — add new column, migrate data, update code, sau đó drop trong sprint sau.
5. **Smoke test ngay sau deploy**: kiểm tra critical path.

**Rollback criteria**: error rate tăng >0.1%, p95 latency tăng >20% so với baseline.

---

**Q8. Bạn nhận được bug report mơ hồ: "thỉnh thoảng không checkout được" — approach như thế nào?**

**A:**  
**Không reproduce ngay, gather information first**:

1. **Clarify**: "thỉnh thoảng" = bao nhiêu % failure? User cụ thể? Browser? Giờ nào?
2. **Logs**: tìm error logs trong khoảng thời gian reported, correlate với `userId` hoặc `sessionId`.
3. **Reproduce**: tái tạo với cùng điều kiện (payment method, product combo, user state).
4. **Isolate**: frontend error hay backend 500? Network timeout? Payment gateway error?

```
Hypothetical diagnosis path:
Logs → "VNPay signature mismatch" intermittently
→ Check: amount formatting (dot vs comma, floating point precision)
→ Root cause: `price * quantity` = 99.9000000001 → signature mismatch
→ Fix: Math.round() trước khi tạo signature
```

**Không bao giờ đóng bug "cannot reproduce"** mà không có evidence. Document investigation, add logging cho next occurrence.

---

*— End of Document —*

> 📌 **Ghi chú**: Tài liệu này tổng hợp 48 câu hỏi phỏng vấn Senior Fullstack Developer (.NET Core + ReactJS), phân theo 6 nhóm kỹ năng cốt lõi. Mỗi câu trả lời phản ánh góc nhìn production-first, có trade-off rõ ràng — không chỉ lý thuyết.
