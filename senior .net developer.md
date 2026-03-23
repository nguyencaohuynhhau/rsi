# Câu hỏi phỏng vấn Senior .NET Developer & Đáp án

---

## 1. DI container trong .NET Core hoạt động như thế nào, và khi nào nên sử dụng transient, scoped và singleton?

DI container tích hợp sẵn trong .NET Core tuân theo nguyên tắc **Inversion of Control (Đảo ngược điều khiển)**. Các service được đăng ký trong `Program.cs` (hoặc `Startup.cs`) và được resolve tự động thông qua constructor injection.

- **Transient** (`AddTransient`): Một instance mới được tạo mỗi lần được yêu cầu. Sử dụng cho các service nhẹ, không giữ trạng thái (stateless).
- **Scoped** (`AddScoped`): Một instance cho mỗi HTTP request (hoặc mỗi `IServiceScope`). Sử dụng cho các service giữ trạng thái ở cấp request, chẳng hạn như `DbContext`.
- **Singleton** (`AddSingleton`): Một instance duy nhất cho toàn bộ vòng đời ứng dụng. Sử dụng cho các service stateless, cache, hoặc nơi lưu trữ cấu hình.

**Lỗi thường gặp**: Inject một scoped service vào singleton gây ra **captive dependency (phụ thuộc bị giam giữ)** — instance scoped sẽ tồn tại lâu như singleton, dẫn đến dữ liệu cũ và lỗi đồng thời (concurrency bugs). Framework sẽ ném `InvalidOperationException` khi `ValidateScopes` được bật.

---

## 2. Giải thích middleware pipeline trong ASP.NET Core. Luồng xử lý request hoạt động như thế nào?

Middleware pipeline là một chuỗi các delegate được thực thi theo thứ tự cho mọi HTTP request. Mỗi middleware component có thể:

1. Xử lý request và chuyển tiếp cho middleware tiếp theo thông qua `next(context)`.
2. Ngắt pipeline (short-circuit) bằng cách không gọi `next()`.
3. Thực thi logic cả trước và sau middleware tiếp theo (tạo mô hình "sandwich").

Thứ tự đăng ký trong `Program.cs` rất quan trọng. Một pipeline điển hình:

```
UseExceptionHandler → UseHsts → UseHttpsRedirection → UseStaticFiles
→ UseRouting → UseCors → UseAuthentication → UseAuthorization
→ UseEndpoints
```

Custom middleware được tạo bằng cách implement `IMiddleware` hoặc viết inline delegate với `app.Use()`. Đối với các cross-cutting concern (logging, correlation ID, rate limiting), middleware là tầng kiến trúc phù hợp.

---

## 3. Sự khác biệt giữa `async/await` và sử dụng `Task` trực tiếp là gì? Làm thế nào để tránh các lỗi async phổ biến?

`async/await` là cú pháp đường (syntactic sugar) trên Task-based Asynchronous Pattern (TAP). Trình biên dịch tạo ra một state machine tạm dừng thực thi tại `await` và tiếp tục khi task hoàn thành — mà không chặn thread.

**Các lỗi cần tránh:**

- **`.Result` hoặc `.Wait()` trên code async** — gây deadlock trong môi trường có `SynchronizationContext` (ASP.NET classic, ứng dụng UI). Trong ASP.NET Core không có sync context, nhưng blocking vẫn lãng phí thread pool thread.
- **`async void`** — exception không thể được bắt bởi caller. Chỉ sử dụng cho event handler. Luôn trả về `Task` hoặc `Task<T>`.
- **Không sử dụng `ConfigureAwait(false)` trong thư viện** — trong code thư viện, thêm `.ConfigureAwait(false)` để tránh capture synchronization context của caller.
- **Tạo state machine không cần thiết** — nếu method chỉ đơn giản trả về một lời gọi async khác mà không có logic bổ sung, hãy trả về `Task` trực tiếp thay vì đánh dấu method là `async`.

---

## 4. Bạn sẽ thiết kế một REST API có khả năng mở rộng với versioning, xử lý lỗi và phân trang như thế nào?

**Versioning (Quản lý phiên bản)**: Sử dụng versioning qua URL segment (`/api/v1/orders`) để rõ ràng, hoặc versioning qua header (`Api-Version: 2`) để URL gọn hơn. Package `Asp.Versioning.Http` cung cấp hỗ trợ sẵn.

**Xử lý lỗi**: Trả về thông tin lỗi nhất quán sử dụng **RFC 7807** (`ProblemDetails`). Dùng global exception handler middleware để bắt các exception chưa xử lý, ghi log đầy đủ ngữ cảnh, và trả về response lỗi đã được làm sạch. Không bao giờ để lộ stack trace trong production.

**Phân trang**: Sử dụng phân trang dựa trên cursor cho tập dữ liệu lớn (hiệu suất tốt hơn phân trang offset). Trả về response envelope:

```json
{
  "data": [...],
  "nextCursor": "abc123",
  "hasMore": true
}
```

**Các nguyên tắc thiết kế bổ sung:**
- Sử dụng đúng HTTP status code (201 cho tạo mới, 204 cho xóa, 409 cho xung đột).
- Triển khai idempotency key cho các thao tác không idempotent (POST).
- Áp dụng rate limiting thông qua `System.Threading.RateLimiting`.
- Sử dụng `CancellationToken` trong tất cả controller action để xử lý khi client ngắt kết nối.

---

## 5. Entity Framework Core xử lý migration, change tracking và tối ưu hiệu suất như thế nào?

**Migration**: EF Core so sánh model snapshot hiện tại với cấu hình `DbContext` và tạo ra các file migration. Áp dụng qua `dotnet ef database update` hoặc `context.Database.Migrate()` khi khởi động (không khuyến khích cho production — hãy sử dụng CI/CD script thay thế).

**Change tracking (Theo dõi thay đổi)**: `ChangeTracker` giám sát các entity được lấy từ database. Khi gọi `SaveChanges()`, nó so sánh giá trị property hiện tại với snapshot được chụp tại thời điểm query và tạo ra SQL tương ứng (INSERT/UPDATE/DELETE). Đối với query chỉ đọc, sử dụng `.AsNoTracking()` để bỏ qua chi phí tracking.

**Tối ưu hiệu suất:**

- **Split query** (`.AsSplitQuery()`) cho các query có nhiều collection include để tránh bùng nổ tích Descartes (cartesian explosion).
- **Compiled query** (`EF.CompileAsyncQuery`) cho các query hot-path được thực thi lặp đi lặp lại.
- **Projection** (`Select`) thay vì load toàn bộ entity khi chỉ cần một tập con các cột.
- **Batch operation** — EF Core 7+ hỗ trợ `ExecuteUpdate` và `ExecuteDelete` cho các thao tác hàng loạt mà không cần load entity.
- **Connection resiliency (Khả năng phục hồi kết nối)** — cấu hình chiến lược retry qua `EnableRetryOnFailure` cho các lỗi database tạm thời.

---

## 6. Bạn sử dụng pattern nào để xử lý distributed transaction và đảm bảo tính nhất quán dữ liệu giữa các microservice?

Distributed two-phase commit (2PC) thường được tránh trong kiến trúc microservice do coupling chặt và đặc tính khả dụng kém. Thay vào đó:

**Saga Pattern**: Điều phối một chuỗi các transaction cục bộ xuyên suốt các service. Mỗi bước có một hành động bù trừ (compensating action) để rollback. Hai cách tiếp cận:
- **Choreography (Biên đạo)**: Các service phát hành domain event; các service khác phản ứng. Đơn giản nhưng khó theo dõi luồng tổng thể.
- **Orchestration (Điều phối)**: Một orchestrator trung tâm điều khiển các bước saga. Dễ suy luận và debug hơn.

**Outbox Pattern**: Ghi domain event vào bảng `Outbox` trong cùng một database transaction với dữ liệu nghiệp vụ. Một tiến trình nền (hoặc công cụ CDC như Debezium) phát hành các event lên message broker. Điều này đảm bảo giao hàng ít nhất một lần (at-least-once delivery) mà không cần distributed transaction.

**Idempotent consumer (Consumer bất biến)**: Vì at-least-once delivery có nghĩa là có thể có bản sao, mọi consumer phải xử lý việc tái xử lý một cách an toàn — thường thông qua bảng `IdempotencyKey` hoặc `ProcessedMessages`.

**Triển khai trong .NET**: Sử dụng MassTransit hoặc NServiceBus để điều phối saga, hỗ trợ outbox, và các chính sách retry có sẵn.

---

## 7. Bạn triển khai authentication và authorization trong ASP.NET Core như thế nào? So sánh JWT và cookie-based.

**Authentication (Xác thực)** xác minh danh tính; **Authorization (Phân quyền)** xác định quyền truy cập.

**JWT (Bearer token):**
- Stateless — server xác thực chữ ký token mà không cần session store.
- Lý tưởng cho API được sử dụng bởi SPA, ứng dụng mobile, hoặc các service khác.
- Lưu trữ token an toàn (HttpOnly cookie hoặc secure storage, không bao giờ dùng `localStorage`).
- Sử dụng access token ngắn hạn (5–15 phút) kết hợp refresh token để xoay vòng.

**Cookie-based:**
- Server tạo session; trình duyệt tự động gửi cookie.
- Bảo vệ CSRF tích hợp sẵn với anti-forgery token.
- Phù hợp hơn cho ứng dụng render phía server (Razor Pages, MVC).

**Các tầng authorization trong ASP.NET Core:**
- **Role-based (Dựa trên vai trò)**: `[Authorize(Roles = "Admin")]` — đơn giản nhưng thô.
- **Policy-based (Dựa trên chính sách)**: `[Authorize(Policy = "CanEditOrders")]` — linh hoạt, có thể kết hợp các requirement.
- **Resource-based (Dựa trên tài nguyên)**: Inject `IAuthorizationService` và đánh giá policy đối với một instance tài nguyên cụ thể (ví dụ: "người dùng này có thể sửa *đơn hàng này* không?").

Đối với hệ thống multi-tenant, triển khai custom `IAuthorizationHandler` để xác thực ngữ cảnh tenant trên mọi request.

---

## 8. CQRS pattern là gì và khi nào thì phù hợp? Bạn sẽ triển khai nó trong .NET như thế nào?

**CQRS (Command Query Responsibility Segregation — Phân tách trách nhiệm Command và Query)** tách biệt read model khỏi write model. Command thay đổi trạng thái; query đọc trạng thái — thông qua các model khác nhau, và có thể là các data store khác nhau.

**Khi nào nên sử dụng:**
- Workload đọc và ghi có nhu cầu mở rộng hoặc tối ưu rất khác nhau.
- Domain phức tạp và hưởng lợi từ write model phong phú (DDD aggregate) trong khi read cần các projection phẳng, đã denormalize.
- Khi sử dụng event sourcing.

**Khi nào KHÔNG nên sử dụng:**
- Ứng dụng CRUD đơn giản — CQRS thêm độ phức tạp không cần thiết.
- Team nhỏ không có đủ nguồn lực để duy trì hai model.

**Triển khai trong .NET:**

```csharp
// Phía Command — sử dụng MediatR
public record CreateOrderCommand(string CustomerId, List<OrderItem> Items) : IRequest<Guid>;

public class CreateOrderHandler : IRequestHandler<CreateOrderCommand, Guid>
{
    public async Task<Guid> Handle(CreateOrderCommand cmd, CancellationToken ct)
    {
        // Logic domain, validation, lưu trữ qua EF Core
    }
}

// Phía Query — read model mỏng, có thể dùng Dapper để tăng hiệu suất
public record GetOrderQuery(Guid OrderId) : IRequest<OrderDto>;
```

Sử dụng MediatR để dispatch command/query trong process. Với CQRS đầy đủ có read store riêng biệt, project domain event vào các bảng tối ưu cho đọc hoặc search index (Elasticsearch).

---

## 9. Bạn tiếp cận logging, observability và health check trong ứng dụng .NET production như thế nào?

**Structured logging (Log có cấu trúc)**: Sử dụng `ILogger<T>` với Serilog hoặc NLog làm provider. Log dữ liệu có cấu trúc, không dùng chuỗi nội suy:

```csharp
// Tốt — có cấu trúc, có thể tìm kiếm
logger.LogInformation("Order {OrderId} placed by {CustomerId}", orderId, customerId);

// Xấu — không có cấu trúc, không thể tìm kiếm
logger.LogInformation($"Order {orderId} placed by {customerId}");
```

**Distributed tracing (Theo dõi phân tán)**: Sử dụng `System.Diagnostics.Activity` và OpenTelemetry SDK để truyền trace context xuyên suốt các service. Export sang Jaeger, Zipkin, hoặc backend tương thích OTLP. Correlation ID cần được truyền qua HTTP header, message bus header, và log scope.

**Metrics (Số liệu)**: Expose counter, histogram, và gauge thông qua `System.Diagnostics.Metrics`. Thu thập bằng Prometheus hoặc đẩy lên OTLP.

**Health check (Kiểm tra sức khỏe)**: Đăng ký qua `builder.Services.AddHealthChecks()`:
- **Liveness** (`/health/live`): Tiến trình có đang chạy không? Được orchestrator sử dụng để restart container.
- **Readiness** (`/health/ready`): Có thể phục vụ traffic không? Kiểm tra kết nối database, các dependency downstream.
- Custom health check implement `IHealthCheck` và trả về `Healthy`, `Degraded`, hoặc `Unhealthy`.

---

## 10. Bạn thiết kế và tối ưu chiến lược caching trong .NET backend như thế nào?

**Caching đa tầng:**

1. **In-memory cache** (`IMemoryCache`): Nhanh nhất, mỗi instance một bản. Tốt cho dữ liệu tham chiếu ít thay đổi.
2. **Distributed cache** (`IDistributedCache` backed bởi Redis hoặc SQL Server): Chia sẻ giữa các instance. Thiết yếu trong triển khai load-balanced hoặc container hóa.
3. **Response caching / Output caching**: ASP.NET Core 7+ output caching middleware cache toàn bộ response với invalidation dựa trên tag.

**Chiến lược invalidation cache:**
- **Dựa trên thời gian (TTL)**: Đơn giản, eventual consistency. Sử dụng TTL ngắn cho dữ liệu biến động.
- **Dựa trên sự kiện (Event-driven)**: Phát hành cache invalidation event khi dữ liệu thay đổi (qua Redis Pub/Sub hoặc message broker).
- **Cache-aside pattern**: Ứng dụng kiểm tra cache trước, load từ DB khi cache miss, sau đó đưa vào cache.

**Các lỗi thường gặp:**
- **Cache stampede (Đổ xô cache)**: Khi một cache key phổ biến hết hạn, nhiều request đồng thời đánh vào DB cùng lúc. Giảm thiểu bằng `SemaphoreSlim` hoặc thư viện như `FusionCache` hỗ trợ chống stampede.
- **Dữ liệu cũ (Stale data)**: Luôn xem xét liệu nghiệp vụ có chấp nhận eventual consistency cho entity được cache hay không.
- **Chi phí serialization**: Với distributed cache, sử dụng serializer hiệu quả (MessagePack, protobuf) thay vì JSON.

---

## 11. Các nguyên tắc chính của Domain-Driven Design là gì và bạn áp dụng chúng trong giải pháp .NET như thế nào?

**Các pattern chiến lược (Strategic patterns):**
- **Bounded Context (Ngữ cảnh giới hạn)**: Mỗi microservice hoặc module sở hữu một subdomain cụ thể với ngôn ngữ chung (ubiquitous language) và model riêng. Một "Order" trong ngữ cảnh Sales khác với "Order" trong Shipping.
- **Context Mapping**: Xác định mối quan hệ giữa các bounded context — Anti-Corruption Layer, Shared Kernel, Conformist, v.v.

**Các pattern chiến thuật (Tactical patterns):**
- **Aggregate**: Một cụm entity và value object với một root entity duy nhất thực thi các ràng buộc bất biến (invariant). Mọi thao tác ghi đều đi qua aggregate root.
- **Value Object**: Kiểu bất biến (immutable) được định nghĩa bởi thuộc tính, không phải định danh (ví dụ: `Money`, `Address`). Triển khai bằng `record` trong C#.
- **Domain Event**: Tín hiệu cho biết điều gì đó có ý nghĩa đã xảy ra (`OrderPlaced`, `PaymentReceived`). Được phát ra bởi aggregate, xử lý bởi application service hoặc các bounded context khác.
- **Repository**: Trừu tượng hóa truy cập dữ liệu, mỗi aggregate root một repository. Repository trả về và lưu trữ toàn bộ aggregate.

**Cấu trúc project:**

```
src/
  Sales.Domain/          # Entity, value object, domain event, repository interface
  Sales.Application/     # Use case, command, query, DTO
  Sales.Infrastructure/  # EF Core, external service client, repository implementation
  Sales.API/             # Controller, middleware, composition root
```

Tầng Domain không có dependency nào vào infrastructure — Nguyên tắc Đảo ngược Phụ thuộc (Dependency Inversion Principle) đảm bảo infrastructure phụ thuộc vào các abstraction của domain.

---

## 12. Bạn tiếp cận việc profiling hiệu suất, xác định bottleneck và tối ưu hóa trong ứng dụng .NET như thế nào?

**Quy trình profiling:**

1. **Đo lường trước** — không bao giờ tối ưu dựa trên giả định. Sử dụng BenchmarkDotNet cho micro-benchmark và Application Insights / Datadog APM cho profiling production.
2. **Xác định bottleneck** — CPU, bộ nhớ, I/O, hay mạng? Sử dụng `dotnet-counters`, `dotnet-trace`, và `dotnet-dump` để chẩn đoán runtime.
3. **Tối ưu đường dẫn quan trọng (critical path)** — tập trung vào 20% code gây ra 80% độ trễ.

**Các kỹ thuật tối ưu phổ biến:**

- **Giảm allocation**: Sử dụng `Span<T>`, `stackalloc`, `ArrayPool<T>`, và `string.Create` để giảm áp lực GC. Profile bằng `dotnet-gcdump`.
- **Async I/O ở mọi nơi**: Không bao giờ chặn thread trên I/O. Sử dụng `async/await` cho database call, HTTP call, và file operation.
- **Connection pooling**: Tái sử dụng `HttpClient` qua `IHttpClientFactory`. Cấu hình kích thước connection pool của EF Core.
- **Tối ưu query**: Sử dụng SQL profiler hoặc EF Core logging để xác định N+1 query, thiếu index, và load dữ liệu không cần thiết.
- **Nén response**: Bật nén Brotli/gzip cho API response.
- **Giảm thiểu serialization**: Sử dụng `System.Text.Json` source generator cho serialization thân thiện AOT, không allocation.

**Load testing (Kiểm thử tải)**: Sử dụng các công cụ như k6, NBomber, hoặc Apache JMeter để mô phỏng traffic pattern production và xác định điểm giới hạn trước khi triển khai.

---

*Các câu hỏi này bao quát toàn bộ phạm vi kiến thức kỳ vọng của một Senior .NET Developer — từ nội bộ framework và async pattern đến kiến trúc hệ thống phân tán và vận hành production.*
