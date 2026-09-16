# Senior Full-Stack Developer Interview (.NET Core + ReactJS)

> **Position:** Senior Full-Stack Developer (4+ years)
> **Stack:** .NET Core (C#) + ReactJS (TypeScript)
> **Format:** 48 questions (6 sections x 8 questions), senior-level answers with trade-offs and real-world context

---

## SECTION 1: C#, JavaScript/TypeScript & Modern Web Technologies (Q1-Q8)

---

### Q1. So sánh các tính năng nâng cao của C# (records, pattern matching, nullable reference types) với các tính năng tương đương trong TypeScript. Khi nào bạn sử dụng từng tính năng trong dự án full-stack?

**Answer:**

Records trong C# và `type`/`interface` trong TypeScript đều dùng để định nghĩa data model, nhưng C# records cung cấp value equality, immutability và `with` expression có sẵn ở mức ngôn ngữ, trong khi TypeScript cần thư viện bên ngoài hoặc convention thủ công.

```csharp
// C# (.NET 8) - Record với pattern matching
public record OrderDto(string Id, decimal Total, OrderStatus Status);

public string GetStatusMessage(OrderDto order) => order switch
{
    { Status: OrderStatus.Pending, Total: > 1000 } => "Đơn lớn chờ duyệt",
    { Status: OrderStatus.Shipped } => "Đã giao",
    _ => "Đang xử lý"
};

// Nullable reference types - bật trong .csproj
public string? GetCustomerName(int id) // compiler cảnh báo nếu dùng mà không check null
```

```typescript
// TypeScript 5+ - Discriminated union thay thế pattern matching
type Order =
  | { status: "pending"; total: number }
  | { status: "shipped"; trackingId: string };

function getStatusMessage(order: Order): string {
  switch (order.status) {
    case "pending": return order.total > 1000 ? "Đơn lớn chờ duyệt" : "Chờ xử lý";
    case "shipped": return `Đã giao: ${order.trackingId}`; // TypeScript tự narrowing type
  }
}
```

**Trade-off:** C# records mạnh hơn về immutability (init-only properties, positional records) nhưng TypeScript discriminated unions linh hoạt hơn cho UI state machine. Nullable reference types trong C# là compile-time check tương tự TypeScript strict mode -- cả hai đều nên bật từ đầu dự án, bật giữa chừng sẽ sinh ra hàng trăm warning khó xử lý dần.

---

### Q2. Giải thích sự khác biệt cốt lõi giữa async/await trong C# và JavaScript. Tại sao hiểu sai mô hình threading có thể gây bug nghiêm trọng trong ứng dụng full-stack?

**Answer:**

Điểm khác biệt quan trọng nhất: C# async/await chạy trên thread pool đa luồng thực sự, còn JavaScript async/await chạy trên single-threaded event loop. Trong C#, `await` giải phóng thread hiện tại về pool và continuation có thể chạy trên thread khác. Trong JavaScript, `await` chỉ nhường quyền lại cho event loop, mọi thứ vẫn trên một thread duy nhất.

```csharp
// C# - NGUY HIỂM: deadlock trong ASP.NET (legacy SynchronizationContext)
public string GetData()
{
    // KHÔNG BAO GIỜ LÀM THẾ NÀY - .Result block thread, gây deadlock
    var result = _httpClient.GetStringAsync("/api/data").Result;
    return result;
}

// ĐÚNG: async xuyên suốt (async all the way)
public async Task<string> GetDataAsync()
{
    var result = await _httpClient.GetStringAsync("/api/data");
    // Trong .NET 8 minimal API, không có SynchronizationContext
    // nên ConfigureAwait(false) không còn cần thiết ở tầng API
    return result;
}
```

```typescript
// JavaScript - Không có deadlock kiểu C# nhưng có pitfall riêng
// SAI: forEach không await được
async function processOrders(ids: string[]) {
  ids.forEach(async (id) => {
    await fetchOrder(id); // Fire-and-forget, không chờ thực sự!
  });
}

// ĐÚNG: dùng for...of (tuần tự) hoặc Promise.all (song song)
async function processOrders(ids: string[]) {
  await Promise.all(ids.map((id) => fetchOrder(id))); // song song
  // hoặc: for (const id of ids) { await fetchOrder(id); } // tuần tự
}
```

**Pitfall thực tế:** Trong C#, quên `await` một Task sẽ nuốt exception âm thầm (fire-and-forget). Trong JavaScript, unhandled promise rejection giờ crash process trong Node.js. Khi làm full-stack, lỗi phổ biến nhất là dev quen JavaScript nghĩ C# async cũng single-thread, dẫn đến race condition khi truy cập shared state mà không dùng lock hoặc ConcurrentDictionary.

---

### Q3. React 18+ giới thiệu Concurrent Rendering với useTransition và useDeferredValue. Giải thích cách hoạt động, so sánh với debounce truyền thống, và cho ví dụ tích hợp với .NET API.

**Answer:**

Concurrent Rendering cho phép React tạm dừng render không quan trọng để ưu tiên tương tác người dùng (như gõ phím). Khác với debounce (trì hoãn thực thi), useTransition đánh dấu state update là "low priority" -- React vẫn bắt đầu render ngay nhưng có thể interrupt nếu có update ưu tiên cao hơn.

```typescript
// useTransition - cho action chủ động (user trigger)
function SearchPage() {
  const [query, setQuery] = useState("");
  const [results, setResults] = useState<ProductDto[]>([]);
  const [isPending, startTransition] = useTransition();

  const handleSearch = (value: string) => {
    setQuery(value); // HIGH priority - input phản hồi ngay
    startTransition(async () => {
      // LOW priority - React có thể interrupt render này
      const data = await fetch(`/api/products?q=${value}`).then(r => r.json());
      setResults(data);
    });
  };

  return (
    <>
      <input value={query} onChange={(e) => handleSearch(e.target.value)} />
      {isPending && <Spinner />}
      <ProductList items={results} />
    </>
  );
}

// useDeferredValue - cho giá trị bị động (derived/prop)
function ProductList({ items }: { items: ProductDto[] }) {
  const deferredItems = useDeferredValue(items);
  const isStale = items !== deferredItems;

  return (
    <div style={{ opacity: isStale ? 0.7 : 1 }}>
      {deferredItems.map(item => <ProductCard key={item.id} item={item} />)}
    </div>
  );
}
```

```csharp
// .NET 8 API endpoint hỗ trợ search
app.MapGet("/api/products", async (string? q, AppDbContext db, CancellationToken ct) =>
{
    // CancellationToken quan trọng: khi React abort request cũ, server cũng dừng query
    var query = db.Products.AsNoTracking();
    if (!string.IsNullOrEmpty(q))
        query = query.Where(p => EF.Functions.ILike(p.Name, $"%{q}%"));
    return await query.Take(50).ToListAsync(ct);
});
```

**Trade-off:** useTransition phù hợp cho search/filter nặng, nhưng không thay thế debounce cho API call -- nên kết hợp cả hai: debounce (300ms) để giảm request đến server, useTransition để giữ UI responsive trong khi render kết quả. useDeferredValue phù hợp hơn khi data đến từ prop hoặc external store mà bạn không kiểm soát thời điểm update.

---

### Q4. So sánh hệ thống Dependency Injection của .NET Core với các giải pháp state/dependency management trong React (Context API, Zustand, Jotai). Khi nào dùng cái nào?

**Answer:**

.NET Core DI là IoC container hoàn chỉnh quản lý lifetime (Singleton, Scoped, Transient) ở tầng application, còn React Context/Zustand quản lý state và dependency ở tầng UI component tree. Chúng giải quyết bài toán khác nhau nhưng cùng nguyên lý: tách dependency ra khỏi nơi sử dụng.

```csharp
// .NET 8 DI - Keyed Services (mới trong .NET 8)
builder.Services.AddKeyedSingleton<ICache, RedisCache>("redis");
builder.Services.AddKeyedSingleton<ICache, MemoryCache>("memory");

builder.Services.AddScoped<IOrderService, OrderService>();
// Scoped = 1 instance per HTTP request, quan trọng cho DbContext

public class OrderService(
    [FromKeyedServices("redis")] ICache cache,  // Primary constructor DI (.NET 8)
    AppDbContext db) : IOrderService
{
    public async Task<Order?> GetAsync(int id) =>
        await cache.GetOrSetAsync($"order:{id}",
            () => db.Orders.FindAsync(id));
}
```

```typescript
// React - Context cho dependency thực sự (services, config)
// Zustand cho UI state (nhanh, ít boilerplate)

import { create } from "zustand";

interface OrderStore {
  orders: Order[];
  fetchOrders: () => Promise<void>;
}

const useOrderStore = create<OrderStore>((set) => ({
  orders: [],
  fetchOrders: async () => {
    const data = await api.get<Order[]>("/api/orders");
    set({ orders: data });  // Chỉ component subscribe orders mới re-render
  },
}));

// Context - cho DI pattern (API client, theme, auth)
const ApiContext = createContext<ApiClient>(null!);
```

**Khi nào dùng gì:** React Context phù hợp cho giá trị ít thay đổi (theme, auth, API client) vì mỗi lần value thay đổi sẽ re-render toàn bộ consumer. Zustand/Jotai cho state thay đổi thường xuyên (form, filter, list). Sai lầm phổ biến là nhét mọi thứ vào Context gây re-render cascade, hoặc dùng Redux cho project nhỏ khi Zustand chỉ cần 10 dòng code. Ở backend, pitfall là đăng ký DbContext làm Singleton thay vì Scoped -- gây memory leak và stale data.

---

### Q5. Đánh giá các giải pháp CSS trong React hiện đại: CSS Modules, Tailwind CSS, và CSS-in-JS. Dự án nào nên dùng giải pháp nào?

**Answer:**

Ba hướng tiếp cận chính khác nhau ở runtime cost, developer experience, và khả năng scale. CSS Modules là zero-runtime với scoped class names. Tailwind là utility-first với build-time purging. CSS-in-JS cho phép dynamic styling nhưng có runtime overhead.

```typescript
// 1. CSS Modules - zero runtime, scoped tu dong
import styles from "./Button.module.css";
const Button = () => <button className={styles.primary}>Click</button>;

// 2. Tailwind CSS - utility-first, nhanh prototype
import { clsx } from "clsx";
const Button = ({ variant }: { variant: "primary" | "danger" }) => (
  <button className={clsx(
    "px-4 py-2 rounded font-medium transition-colors",
    variant === "primary" && "bg-blue-600 hover:bg-blue-700 text-white",
    variant === "danger" && "bg-red-600 hover:bg-red-700 text-white",
  )}>
    Click
  </button>
);

// 3. CSS-in-JS (Emotion) - dynamic styling mạnh nhưng có runtime cost
import styled from "@emotion/styled";
const Button = styled.button<{ $size: number }>`
  padding: ${(p) => p.$size * 4}px ${(p) => p.$size * 8}px;
  background: ${(p) => p.theme.colors.primary};
`;
```

| Tiêu chí | CSS Modules | Tailwind | CSS-in-JS |
|---|---|---|---|
| Runtime cost | Zero | Zero | Có (parse + inject) |
| Bundle size | Nhỏ | Rất nhỏ (purged) | Lớn hơn |
| Dynamic styling | Khó | Cần workaround | Rất mạnh |
| Server Components | Tương thích | Tương thích | Hầu hết KHÔNG tương thích |

**Khuyến nghị:** Tailwind cho phần lớn dự án mới (fast iteration, nhỏ gọn, tương thích RSC). CSS Modules cho dự án cần tách biệt rõ ràng styling với logic. CSS-in-JS đang giảm xu hướng vì không tương thích React Server Components -- nếu cần dynamic styling, xem xét Panda CSS hoặc Vanilla Extract (zero-runtime CSS-in-JS).

---

### Q6. Thiết kế kiến trúc real-time feature sử dụng SignalR (.NET) và React. Nêu các gotcha phổ biến và cách xử lý.

**Answer:**

SignalR cung cấp abstraction trên WebSocket với fallback (Server-Sent Events, Long Polling) và tích hợp sẵn với .NET authentication/authorization. Kiến trúc tốt cần xử lý reconnection, message ordering, và cleanup đúng cách ở cả hai phía.

```csharp
// .NET 8 - Strongly-typed Hub
public interface IChatClient
{
    Task ReceiveMessage(ChatMessage message);
    Task UserJoined(string userName);
}

public class ChatHub : Hub<IChatClient>
{
    public override async Task OnConnectedAsync()
    {
        var user = Context.User?.Identity?.Name ?? "Anonymous";
        await Groups.AddToGroupAsync(Context.ConnectionId, "general");
        await Clients.Group("general").UserJoined(user);
    }

    public async Task SendMessage(string content)
    {
        var message = new ChatMessage(Context.User!.Identity!.Name!, content, DateTime.UtcNow);
        await _repository.SaveAsync(message);
        await Clients.Group("general").ReceiveMessage(message);
    }
}

// Scale-out: .AddStackExchangeRedis("connection-string") khi deploy nhieu instance
```

```typescript
// React - Custom hook voi reconnection handling
import { HubConnectionBuilder, HubConnectionState } from "@microsoft/signalr";

function useSignalR(url: string) {
  const [connection] = useState(() =>
    new HubConnectionBuilder()
      .withUrl(url, { accessTokenFactory: () => getToken() })
      .withAutomaticReconnect([0, 2000, 5000, 10000, 30000])
      .build()
  );

  useEffect(() => {
    connection.start();
    return () => { connection.stop(); }; // QUAN TRỌNG: cleanup khi unmount
  }, [connection]);

  return connection;
}

// Usage - QUAN TRỌNG: dùng functional update, tránh stale closure
useEffect(() => {
  connection.on("ReceiveMessage", (msg: ChatMessage) => {
    setMessages(prev => [...prev, msg]);
  });
  return () => { connection.off("ReceiveMessage"); };
}, [connection]);
```

**Gotcha phổ biến:** (1) Quên cleanup `connection.off()` gây memory leak và duplicate messages. (2) Stale closure -- handler bắt state cũ, phải dùng functional update hoặc useRef. (3) Khi scale nhiều server instance, phải dùng Redis backplane, không thì message chỉ đến client kết nối cùng instance. (4) Không xử lý reconnection -- user mất kết nối 30 giây sẽ miss messages, cần fetch lại history sau reconnect.

---

### Q7. Giải thích cách đạt full-stack type safety giữa .NET API và React frontend bằng OpenAPI code generation. So sánh NSwag vs openapi-typescript.

**Answer:**

Full-stack type safety nghĩa là khi thay đổi API contract ở backend, frontend sẽ báo lỗi compile-time thay vì crash runtime. Cách tiếp cận phổ biến nhất là generate OpenAPI spec từ .NET, sau đó generate TypeScript types/client từ spec đó.

```csharp
// .NET 8 - Tu dong generate OpenAPI spec
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

app.MapGet("/api/orders/{id}", async (int id, AppDbContext db) =>
{
    var order = await db.Orders.FindAsync(id);
    return order is null ? Results.NotFound() : Results.Ok(order);
})
.WithName("GetOrder")
.Produces<OrderDto>(200)
.Produces(404);
```

```typescript
// openapi-typescript + openapi-fetch (lightweight, type-only)
// npx openapi-typescript http://localhost:5000/swagger/v1/swagger.json -o ./src/api/schema.d.ts
import createClient from "openapi-fetch";
import type { paths } from "./api/schema";

const client = createClient<paths>({ baseUrl: "http://localhost:5000" });

// Fully typed - path, params, response deu co autocomplete
const { data, error } = await client.GET("/api/orders/{id}", {
  params: { path: { id: 123 } },
});
```

| | NSwag | openapi-typescript |
|---|---|---|
| Output | Full client class + types | Types only + thin fetch wrapper |
| Bundle size | Lớn (generated code) | Rất nhỏ (types erased khi compile) |
| Customization | Ít linh hoạt | Dùng fetch native, dễ customize |
| CI integration | Cần .NET runtime | Chỉ cần Node.js |

**Khuyến nghị:** openapi-typescript cho dự án mới vì lightweight, tree-shakeable. NSwag phù hợp nếu team đã quen hoặc cần generate C# client cho các ứng dụng bên thứ 3. Quan trọng nhất: tích hợp vào CI -- mỗi lần build sẽ re-generate types, nếu frontend không compile được thì pipeline fail sớm.

---

### Q8. Span<T> và Memory<T> trong C# giải quyết bài toán gì? So sánh với cách JavaScript xử lý binary data (ArrayBuffer, TypedArray). Cho ví dụ full-stack xử lý file upload.

**Answer:**

`Span<T>` cho phép làm việc với vùng nhớ liên tục (contiguous memory) mà không cần allocate array mới, giảm GC pressure đáng kể. Nó là stack-only (ref struct), không thể dùng trong async method -- khi cần async thì dùng `Memory<T>`. JavaScript có `ArrayBuffer` + `TypedArray` tương tự nhưng không có zero-allocation slicing như Span.

```csharp
// .NET 8 - Parse CSV header không allocate string mới
public static IEnumerable<Range> ParseCsvHeaders(ReadOnlySpan<char> line)
{
    var ranges = new List<Range>();
    int start = 0;
    for (int i = 0; i <= line.Length; i++)
    {
        if (i == line.Length || line[i] == ',')
        {
            ranges.Add(start..i);  // Zero allocation - chỉ lưu range
            start = i + 1;
        }
    }
    return ranges;
}

// File upload endpoint với streaming (không load toàn bộ file vào RAM)
app.MapPost("/api/upload", async (HttpRequest request, CancellationToken ct) =>
{
    await using var stream = File.Create($"/uploads/{Guid.NewGuid()}");
    await request.Body.CopyToAsync(stream, ct);
    return Results.Ok();
});
```

```typescript
// React - File upload với progress và chunk processing
async function uploadFile(file: File, onProgress: (pct: number) => void) {
  const CHUNK_SIZE = 1024 * 1024; // 1MB chunks
  const totalChunks = Math.ceil(file.size / CHUNK_SIZE);

  for (let i = 0; i < totalChunks; i++) {
    const chunk = file.slice(i * CHUNK_SIZE, (i + 1) * CHUNK_SIZE);
    await fetch("/api/upload/chunk", {
      method: "POST",
      headers: { "X-Chunk-Index": String(i), "X-Total-Chunks": String(totalChunks) },
      body: chunk,
    });
    onProgress(((i + 1) / totalChunks) * 100);
  }
}
```

**Điểm cốt lõi:** `Span<T>` giảm allocation trong hot path (parsing, serialization) -- benchmark cho thấy giảm 40-60% GC pause. JavaScript `TypedArray.subarray()` tương đương Span (shared buffer) còn `slice()` tương đương `ToArray()` (copy). Pitfall: Span không dùng được trong async/await, closure, hoặc LINQ -- khi cần thì chuyển sang `Memory<T>`.

---

## SECTION 2: Software Design Patterns, Data Structures & Algorithms (Q9-Q16)

---

### Q9. Hãy cho ví dụ thực tế về vi phạm nguyên tắc SOLID trong dự án .NET Core và cách khắc phục?

**Answer:**

Vi phạm phổ biến nhất trong thực tế là **Single Responsibility** và **Dependency Inversion**. Một controller "béo" vừa validate, vừa chứa business logic, vừa gọi thẳng DbContext là dấu hiệu rõ nhất.

```csharp
// VI PHẠM: Controller làm quá nhiều việc
public class OrderController : ControllerBase
{
    private readonly AppDbContext _db;
    public async Task<IActionResult> Create(OrderDto dto)
    {
        // validate, tính giá, apply discount, gọi payment, save DB, send email...
        // 200 dòng code ở đây
    }
}

// SỬA: Tách responsibility rõ ràng
public class OrderController : ControllerBase
{
    private readonly ISender _mediator;
    public async Task<IActionResult> Create(CreateOrderCommand cmd)
        => Ok(await _mediator.Send(cmd));
}
```

**Open/Closed** thường bị vi phạm khi dùng `if/else` hoặc `switch` để xử lý nhiều loại nghiệp vụ -- giải pháp là dùng Strategy pattern kết hợp DI. **Interface Segregation** bị vi phạm khi một interface có 15-20 method mà các class implement chỉ dùng 2-3 cái, buộc phải throw `NotImplementedException`.

Bẫy thực tế: đừng áp dụng SOLID một cách cuồng tín. Nếu một service chỉ có 30 dòng code và không có khả năng mở rộng, việc tách ra 5 interface/class chỉ tạo thêm complexity vô nghĩa. SOLID là hướng dẫn, không phải luật.

---

### Q10. Repository + Unit of Work pattern có còn cần thiết khi đã dùng EF Core không? Khi nào nên dùng, khi nào là thừa?

**Answer:**

Đây là câu hỏi gây tranh cãi nhiều nhất trong cộng đồng .NET. EF Core **đã là** một implementation của Repository (DbSet) + Unit of Work (DbContext). Việc wrap thêm một lớp Generic Repository lên trên thường chỉ tạo ra **leaky abstraction** và giấu đi sức mạnh của EF Core (như `Include`, `AsNoTracking`, projection).

**Khi KHÔNG nên dùng:** Dự án CRUD đơn giản, team nhỏ, chỉ dùng 1 loại database. Generic Repository kiểu `IRepository<T>` với `GetAll()`, `GetById()` chỉ tạo thêm một lớp trung gian vô ích.

**Khi NÊN dùng:** Khi bạn thực sự cần abstract data access layer -- ví dụ phải hỗ trợ nhiều loại DB (SQL Server + MongoDB), hoặc muốn tách biệt domain layer hoàn toàn khỏi infrastructure trong Clean Architecture.

```csharp
// THAY VÌ Generic Repository, dùng Specific Repository / Query Object
public interface IOrderRepository
{
    Task<Order?> GetPendingOrderWithItems(Guid id);  // rõ intent
    Task<PagedResult<OrderSummary>> SearchOrders(OrderFilter filter);
}

// Hoặc inject DbContext trực tiếp vào handler (Vertical Slice)
public class GetOrderHandler : IRequestHandler<GetOrderQuery, OrderDto>
{
    private readonly AppDbContext _db;
    public async Task<OrderDto> Handle(GetOrderQuery q, CancellationToken ct)
        => await _db.Orders.Where(o => o.Id == q.Id)
            .Select(o => new OrderDto { ... }).FirstAsync(ct);
}
```

Trade-off: inject thẳng DbContext thì đơn giản nhưng khó unit test (phải dùng in-memory DB hoặc Testcontainers). Repository thì dễ mock nhưng tạo thêm abstraction layer. Lời khuyên thực tế: **bắt đầu không có Repository**, chỉ thêm khi có nhu cầu thực sự.

---

### Q11. CQRS + MediatR mang lại lợi ích gì? Pipeline behaviors hoạt động ra sao? Khi nào dùng là overkill?

**Answer:**

CQRS tách model đọc và ghi, giúp optimize từng phía độc lập. MediatR là thư viện triển khai mediator pattern, **không phải CQRS** -- nhưng thường dùng chung. Lợi ích chính: decoupling handler khỏi controller, và đặc biệt là **pipeline behaviors** -- cross-cutting concerns dạng middleware.

```csharp
// Pipeline behavior: tự động validate mọi command/query
public class ValidationBehavior<TReq, TRes> : IPipelineBehavior<TReq, TRes>
{
    private readonly IEnumerable<IValidator<TReq>> _validators;
    public async Task<TRes> Handle(TReq req, RequestHandlerDelegate<TRes> next,
        CancellationToken ct)
    {
        var failures = _validators.SelectMany(v => v.Validate(req).Errors);
        if (failures.Any()) throw new ValidationException(failures);
        return await next();  // gọi handler tiếp theo trong pipeline
    }
}
```

Pipeline behaviors phổ biến: **Validation**, **Logging**, **Caching** (cho query), **Transaction** (wrap command trong `BeginTransaction/Commit`), **Performance monitoring**.

**Khi nào overkill:** Dự án CRUD đơn giản, ít business logic. Nếu handler chỉ là 3 dòng gọi DbContext rồi return, bạn đang tạo thêm complexity không cần thiết. Ngoài ra MediatR tạo **indirection** -- khi debug phải nhảy qua nhiều lớp, dev mới vào team sẽ khó follow flow. Cân nhắc dùng khi có 10+ use cases phức tạp với cross-cutting concerns rõ ràng, không dùng cho module chỉ có 3-4 endpoint.

---

### Q12. Giải thích cách áp dụng Strategy, Factory và Decorator pattern trong .NET Core DI container?

**Answer:**

Ba pattern này kết hợp với DI container của .NET Core rất tự nhiên, giúp tránh `if/else` chains và tuân thủ Open/Closed principle.

**Strategy** -- chọn algorithm tại runtime:
```csharp
// Đăng ký nhiều strategy (.NET 8 Keyed Services)
services.AddKeyedScoped<IPaymentProcessor, VnPayProcessor>("vnpay");
services.AddKeyedScoped<IPaymentProcessor, MomoProcessor>("momo");

// Resolve bằng key
public class PaymentService(IServiceProvider sp)
{
    public Task Pay(string method, decimal amount)
    {
        var processor = sp.GetRequiredKeyedService<IPaymentProcessor>(method);
        return processor.ProcessAsync(amount);
    }
}
```

**Decorator** -- wrap behavior lên service có sẵn. Thư viện Scrutor hỗ trợ rất tốt:
```csharp
services.AddScoped<IOrderService, OrderService>();
services.Decorate<IOrderService, CachedOrderService>();   // thêm caching
services.Decorate<IOrderService, LoggingOrderService>();  // thêm logging
// Chain: request đi qua Logging -> Caching -> OrderService thực
```

**Factory** -- khi cần tạo instance dựa trên runtime data mà DI không resolve được trực tiếp, dùng `Func<T>` hoặc factory class đăng ký trong DI.

Pitfall: đừng lạm dụng decorator quá sâu (4-5 lớp), debug sẽ rất khó trace.

---

### Q13. So sánh Clean Architecture và Vertical Slice Architecture -- khi nào chọn cái nào?

**Answer:**

**Clean Architecture** tổ chức code theo layer nằm ngang: Domain -> Application -> Infrastructure -> Presentation. Mỗi layer là một project riêng, dependency hướng vào trong.

**Vertical Slice Architecture** tổ chức code theo feature dọc: mỗi feature (ví dụ `CreateOrder`) chứa tất cả từ request/response model, handler, validation, DB query trong cùng một folder.

| Tiêu chí | Clean Architecture | Vertical Slice |
|---|---|---|
| Cấu trúc folder | Theo layer (`/Domain`, `/Application`) | Theo feature (`/Features/Orders/Create`) |
| Coupling | Loose giữa layers, shared abstractions | Loose giữa features, tight trong feature |
| Thêm feature mới | Sửa nhiều layer/project | Thêm 1 folder mới, không ảnh hưởng feature khác |
| Learning curve | Cao, nhiều abstraction | Thấp hơn, dễ hiểu từng feature độc lập |
| Phù hợp | Domain phức tạp, team lớn, cần enforce rule | Feature-driven, team nhỏ-trung, cần tốc độ |

Trade-off quan trọng: Clean Architecture dễ bị **over-engineering** với quá nhiều interface và mapping. Vertical Slice dễ bị **code duplication** giữa các feature.

Lời khuyên thực tế: bắt đầu với Vertical Slice cho MVP, refactor sang Clean Architecture khi domain đủ phức tạp và team đủ lớn. Nhiều team thành công kết hợp cả hai: Clean Architecture ở mức project structure, Vertical Slice ở mức feature organization trong Application layer.

---

### Q14. Giải thích Aggregate, Value Object và Domain Events trong DDD. Cho ví dụ thực tế?

**Answer:**

**Aggregate** là cluster các entity có chung business invariant, truy cập qua một Aggregate Root duy nhất. **Value Object** là object không có identity, so sánh bằng giá trị, immutable. **Domain Events** thông báo "điều gì đó đã xảy ra" trong domain, giúp decouple giữa các aggregate.

```csharp
// Value Object - C# record + validation
public record Money(decimal Amount, string Currency)
{
    public Money
    {
        if (Amount < 0) throw new DomainException("Amount must be >= 0");
        if (string.IsNullOrEmpty(Currency)) throw new DomainException("Currency required");
    }
    public static Money operator +(Money a, Money b)
    {
        if (a.Currency != b.Currency) throw new DomainException("Currency mismatch");
        return new Money(a.Amount + b.Amount, a.Currency);
    }
}

// Domain Event
public class Order : AggregateRoot
{
    public void Complete()
    {
        Status = OrderStatus.Completed;
        AddDomainEvent(new OrderCompletedEvent(Id, TotalAmount));
        // Handler khác sẽ: gửi email, cập nhật inventory, tạo invoice...
    }
}
```

Pitfall thực tế: aggregate quá lớn gây lock contention trong DB. Nguyên tắc là giữ aggregate nhỏ, dùng domain events để đồng bộ giữa các aggregate thay vì nhét tất cả vào một aggregate root. Một sai lầm phổ biến: dùng Entity thay cho Value Object cho `Address`, `Money`, `DateRange`.

---

### Q15. Khi nào dùng Dictionary, HashSet, List trong .NET? Phân tích Big-O trong các tình huống thực tế?

**Answer:**

Nguyên tắc chọn dựa trên **operation phổ biến nhất** trong use case:

| Collection | Lookup | Insert | Dùng khi |
|---|---|---|---|
| `List<T>` | O(n) | O(1) amortized | Dữ liệu ít, cần index, iterate nhiều |
| `Dictionary<K,V>` | O(1) avg | O(1) avg | Cần map key->value, lookup nhiều |
| `HashSet<T>` | O(1) avg | O(1) avg | Cần check tồn tại, loại bỏ trùng lặp |

```csharp
// SAI: dùng List rồi .Any() trong vòng lặp = O(n*m)
var existingIds = await _db.Products.Select(p => p.Id).ToListAsync();
foreach (var item in importData)
    if (existingIds.Any(id => id == item.Id)) { ... }  // O(n) mỗi lần

// ĐÚNG: dùng HashSet = O(n+m)
var existingIds = (await _db.Products.Select(p => p.Id).ToListAsync()).ToHashSet();
foreach (var item in importData)
    if (existingIds.Contains(item.Id)) { ... }  // O(1) mỗi lần
```

Lưu ý: với dữ liệu nhỏ (dưới ~100 phần tử), `List` có thể nhanh hơn `Dictionary`/`HashSet` nhờ cache locality. Với .NET 6+, `FrozenDictionary` / `FrozenSet` cho read-only scenarios nhanh hơn đáng kể vì optimize layout tại thời điểm tạo.

---

### Q16. So sánh offset-based pagination và keyset (cursor) pagination? Khi nào sorting nên thực hiện trong DB vs in-memory?

**Answer:**

**Offset pagination** (`OFFSET/FETCH` hoặc `.Skip().Take()`): đơn giản, cho phép nhảy đến page bất kỳ. Nhược điểm: page càng lớn, DB phải scan và bỏ qua càng nhiều row.

**Keyset pagination** (cursor): dùng giá trị của row cuối cùng làm mốc:

```csharp
// Offset: chậm dần ở page lớn
var page = await _db.Orders.OrderByDescending(o => o.CreatedAt)
    .Skip((pageNum - 1) * pageSize).Take(pageSize).ToListAsync();

// Keyset: tốc độ ổn định bất kể vị trí
var page = await _db.Orders
    .Where(o => o.CreatedAt < lastSeenDate
        || (o.CreatedAt == lastSeenDate && o.Id < lastSeenId))
    .OrderByDescending(o => o.CreatedAt).ThenByDescending(o => o.Id)
    .Take(pageSize).ToListAsync();
```

| Tiêu chí | Offset | Keyset |
|---|---|---|
| Performance page lớn | Kém (O(offset+limit)) | Ổn định (O(limit)) |
| Nhảy page ngẫu nhiên | Được | Không (chỉ next/prev) |
| Dữ liệu thay đổi liên tục | Bị trùng/mất row | Ổn định, không bị lệch |
| Phù hợp | Admin panel, report ít data | Feed, infinite scroll, API lớn |

**Sorting:** luôn ưu tiên sort trong DB vì DB đã có index. Sort in-memory chỉ hợp lý khi: (1) dữ liệu đã load hết vào memory rồi (cache), (2) sort logic phức tạp phụ thuộc business rule không thể express bằng SQL, (3) dữ liệu từ nhiều nguồn cần merge rồi sort. Pitfall: `.ToList()` rồi `.OrderBy()` trên 100K record là cách phổ biến nhất để giết performance.

---

## SECTION 3: RESTful APIs, JSON & Asynchronous Programming (Q17-Q24)

---

### Q17. What are the core RESTful API design principles and how do you implement them properly in ASP.NET Core?

**Answer:**

RESTful API xoay quanh các nguyên tắc: sử dụng HTTP verbs đúng ngữ nghĩa (GET không thay đổi state, PUT là idempotent, POST tạo mới), resource-based URI (danh từ số nhiều, không dùng động từ), stateless communication.

```csharp
[ApiController]
[Route("api/v1/orders")]
public class OrdersController : ControllerBase
{
    [HttpGet("{id:guid}")]
    [ProducesResponseType(typeof(OrderDto), 200)]
    [ProducesResponseType(404)]
    public async Task<IActionResult> Get(Guid id, CancellationToken ct)
    {
        var order = await _service.GetAsync(id, ct);
        return order is null ? NotFound() : Ok(order);
    }

    [HttpPost]
    [ProducesResponseType(typeof(OrderDto), 201)]
    public async Task<IActionResult> Create(CreateOrderRequest req, CancellationToken ct)
    {
        var order = await _service.CreateAsync(req, ct);
        return CreatedAtAction(nameof(Get), new { id = order.Id }, order);
    }
}
```

Pitfall phổ biến: trả 200 cho mọi thứ thay vì dùng đúng status code (201 Created, 204 No Content, 409 Conflict). Nên dùng `[ApiController]` attribute vì nó tự động enable model validation, binding source inference, và `ProblemDetails` response cho 400+. Trong thực tế, nên thiết kế API theo business capability chứ không phải map 1:1 với database table.

---

### Q18. Explain API versioning strategies in ASP.NET Core. Which approach do you prefer and why?

**Answer:**

Có 4 chiến lược chính: URL path (`/api/v2/orders`), query string (`?api-version=2`), header (`X-Api-Version`), và media type versioning.

```csharp
// .NET 7+ voi Asp.Versioning.Http
builder.Services.AddApiVersioning(opt =>
{
    opt.DefaultApiVersion = new ApiVersion(1, 0);
    opt.AssumeDefaultVersionWhenUnspecified = true;
    opt.ReportApiVersions = true;
    opt.ApiVersionReader = ApiVersionReader.Combine(
        new UrlSegmentApiVersionReader(),
        new HeaderApiVersionReader("X-Api-Version")
    );
}).AddApiExplorer(opt =>
{
    opt.GroupNameFormat = "'v'VVV";
    opt.SubstituteApiVersionInUrl = true;
});
```

URL path versioning là phổ biến nhất vì dễ hiểu, dễ debug, dễ cache. Trade-off là URL bị "xấu" hơn và vi phạm nguyên tắc URI nên identify resource chứ không phải version.

Chiến lược deprecation quan trọng không kém: dùng `[ApiVersion("1.0", Deprecated = true)]` kết hợp với `Sunset` header để báo client biết timeline. Nên maintain tối đa 2 version đồng thời.

---

### Q19. Compare System.Text.Json vs Newtonsoft.Json. When would you still choose Newtonsoft in a .NET 8 project?

**Answer:**

`System.Text.Json` (STJ) là default từ .NET Core 3.0, được thiết kế cho performance với allocation thấp hơn 2-5x so với Newtonsoft nhờ `Utf8JsonReader/Writer` làm việc trực tiếp trên byte UTF-8. Từ .NET 8, STJ đã support source generator tốt hơn giúp trim-friendly cho AOT.

```csharp
// Source generator - zero reflection, AOT-compatible
[JsonSerializable(typeof(List<OrderDto>))]
[JsonSourceGenerationOptions(PropertyNamingPolicy = JsonKnownNamingPolicy.CamelCase)]
public partial class AppJsonContext : JsonSerializerContext { }

builder.Services.ConfigureHttpJsonOptions(opt =>
    opt.SerializerOptions.TypeInfoResolverChain.Add(AppJsonContext.Default));
```

Chọn Newtonsoft khi: cần `$ref`/`$id` handling cho circular references phức tạp, cần `JsonPath` query, project đang dùng nhiều custom `JsonConverter` Newtonsoft mà migrate tốn effort lớn, hoặc cần serialize dynamic/ExpandoObject phức tạp.

Pitfall: STJ mặc định case-sensitive và không serialize field (chỉ property). Khi chuyển từ Newtonsoft sang, đây là hai lỗi phổ biến nhất gây mất data silent.

---

### Q20. Explain how async/await works internally — state machine, SynchronizationContext, and when to use ValueTask.

**Answer:**

Khi compiler gặp `async`, nó generate một struct implement `IAsyncStateMachine` với `MoveNext()` method. Mỗi `await` là một "checkpoint" -- nếu Task chưa complete, state machine lưu current state, đăng ký continuation, rồi return. Khi Task complete, continuation được schedule qua `SynchronizationContext` (nếu có) hoặc `ThreadPool`.

`SynchronizationContext` trong ASP.NET Core là **null** (không có như ASP.NET classic), nên `ConfigureAwait(false)` không cần thiết trong application code -- chỉ cần trong library code để tránh deadlock khi consumer có SynchronizationContext.

`ValueTask<T>` dùng khi: method thường xuyên return synchronously (cache hit, buffer đã có data). Nó tránh allocation Task object. Nhưng **không được** await `ValueTask` nhiều lần, không `WhenAll` được, và không được cache nó. Từ .NET 8, nên dùng `ValueTask` cho hot path performance-critical. Nếu không chắc -- dùng `Task<T>` cho an toàn vì nó ít ràng buộc hơn.

---

### Q21. How do you implement CancellationToken propagation and handle graceful shutdown in ASP.NET Core?

**Answer:**

`CancellationToken` trong ASP.NET Core tự động trigger khi client disconnect (request aborted). Nguyên tắc là propagate token qua **toàn bộ call chain** -- từ controller xuống service, repository, đến DbContext và HttpClient.

```csharp
[HttpGet("report")]
public async Task<IActionResult> GenerateReport(CancellationToken ct)
{
    var data = await _db.Orders
        .Where(o => o.Year == 2025)
        .ToListAsync(ct); // propagate xuong EF Core
    var pdf = await _reportService.GenerateAsync(data, ct);
    return File(pdf, "application/pdf");
}

// Graceful shutdown với IHostApplicationLifetime
public class CleanupService : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
            await ProcessQueueAsync(stoppingToken);
    }
}
```

Trong `Program.cs`, configure `HostOptions.ShutdownTimeout` (default 30s) để background service có đủ thời gian drain. Pitfall phổ biến: quên truyền token vào `HttpClient.SendAsync()` hoặc `Task.Delay()`, dẫn đến request đã bị cancel nhưng server vẫn xử lý tốn resource. Với long-running operation, nên dùng `CancellationTokenSource.CreateLinkedTokenSource()` để combine request cancellation với custom timeout.

---

### Q22. How does rate limiting work in .NET 7+ and what strategies are available out of the box?

**Answer:**

.NET 7 giới thiệu built-in rate limiting middleware với 4 algorithm: Fixed Window, Sliding Window, Token Bucket, và Concurrency Limiter.

```csharp
builder.Services.AddRateLimiter(opt =>
{
    opt.RejectionStatusCode = StatusCodes.Status429TooManyRequests;

    // Per-user partitioned limiter
    opt.AddPolicy("per-user", ctx =>
        RateLimitPartition.GetTokenBucketLimiter(
            ctx.User.Identity?.Name ?? ctx.Connection.RemoteIpAddress?.ToString(),
            _ => new TokenBucketRateLimiterOptions
            {
                TokenLimit = 50,
                ReplenishmentPeriod = TimeSpan.FromMinutes(1),
                TokensPerPeriod = 10
            }));
});

app.UseRateLimiter();
```

Fixed Window đơn giản nhưng có "burst at boundary" problem. Sliding Window giải quyết vấn đề này. Token Bucket phù hợp API cho phép burst ngắn. Concurrency Limiter giới hạn số request đồng thời (hữu ích cho heavy endpoint).

Trong production distributed, cần Redis-backed rate limiting (thư viện bên thứ ba) vì built-in chỉ hoạt động per-instance. Luôn trả `Retry-After` header trong 429 response.

---

### Q23. How do you approach API documentation with OpenAPI/Swagger and contract-first design?

**Answer:**

Từ .NET 9, ASP.NET Core tích hợp `Microsoft.AspNetCore.OpenApi` thay thế dần Swashbuckle. Contract-first nghĩa là viết OpenAPI spec trước rồi generate code -- giúp frontend và backend phát triển song song.

```csharp
// .NET 9 built-in OpenAPI
builder.Services.AddOpenApi(opt =>
{
    opt.AddDocumentTransformer((document, context, ct) =>
    {
        document.Info = new() { Title = "Order API", Version = "v1" };
        return Task.CompletedTask;
    });
});

app.MapOpenApi(); // endpoint: /openapi/v1.json
```

Dùng `[ProducesResponseType]`, `[EndpointSummary]`, `[EndpointDescription]` (Minimal API .NET 7+) để document rõ ràng. Best practice: tích hợp OpenAPI spec validation vào CI pipeline -- nếu spec thay đổi breaking (xóa field, đổi type) thì fail build. Dùng tool như `oasdiff` để detect breaking changes tự động. Đừng expose Swagger UI ở production -- chỉ bật ở development environment.

---

### Q24. Explain the ASP.NET Core middleware pipeline. How do you handle cross-cutting concerns efficiently?

**Answer:**

Middleware pipeline là chuỗi delegate xử lý request theo thứ tự FIFO và response theo thứ tự LIFO (Russian doll model). Mỗi middleware gọi `next()` để chuyển tiếp hoặc short-circuit pipeline.

```csharp
// Thứ tự middleware CỰC KỲ quan trọng
app.UseExceptionHandler("/error");  // 1. Bắt exception từ tất cả middleware phía sau
app.UseHsts();
app.UseHttpsRedirection();
app.UseCors("policy");              // 2. Trước auth
app.UseAuthentication();            // 3. Xác thực
app.UseAuthorization();             // 4. Phân quyền
app.UseRateLimiter();               // 5. Sau auth để rate limit per-user
app.MapControllers();

// Custom middleware cho request timing
public class TimingMiddleware(RequestDelegate next, ILogger<TimingMiddleware> logger)
{
    public async Task InvokeAsync(HttpContext ctx)
    {
        var sw = Stopwatch.StartNew();
        ctx.Response.OnStarting(() =>
        {
            ctx.Response.Headers["X-Response-Time"] = $"{sw.ElapsedMilliseconds}ms";
            return Task.CompletedTask;
        });
        await next(ctx);
    }
}
```

Pitfall kinh điển: đặt CORS sau Authorization -- browser preflight request sẽ bị 401 trước khi CORS header được thêm, gây lỗi khó debug. Dùng `IMiddleware` (transient) khi cần inject scoped service, dùng convention-based middleware (singleton implicitly) cho stateless logic vì performance tốt hơn.

---

## SECTION 4: Relational Databases & ORM Frameworks (Q25-Q32)

---

### Q25. Compare EF Core vs Dapper. When do you use each, and can they coexist?

**Answer:**

EF Core là full ORM với change tracking, migration, LINQ provider -- phù hợp CRUD-heavy domain. Dapper là micro-ORM, chỉ map query result sang object -- nhanh hơn 5-10x cho read-heavy, reporting.

```csharp
// Coexistence pattern: share DbConnection
public class OrderRepository(AppDbContext efContext, IDbConnection dapper)
{
    // EF Core cho write operations - leverage change tracking
    public async Task CreateAsync(Order order, CancellationToken ct)
    {
        efContext.Orders.Add(order);
        await efContext.SaveChangesAsync(ct);
    }

    // Dapper cho complex read - raw SQL, tối ưu performance
    public async Task<IEnumerable<OrderSummaryDto>> GetSummaryAsync(int year)
    {
        return await dapper.QueryAsync<OrderSummaryDto>(
            "SELECT o.Id, o.Total, c.Name FROM Orders o " +
            "JOIN Customers c ON o.CustomerId = c.Id " +
            "WHERE YEAR(o.CreatedAt) = @Year", new { Year = year });
    }
}
```

Nguyên tắc: EF Core cho write path (cần transaction, change tracking), Dapper cho read path phức tạp (report, dashboard). Có thể share cùng connection: `efContext.Database.GetDbConnection()`. Pitfall: không mix EF Core tracking với Dapper update trên cùng entity trong cùng request.

---

### Q26. Explain EF Core performance optimizations: AsNoTracking, Compiled Queries, and Split Queries.

**Answer:**

`AsNoTracking()` tắt change tracker -- tiết kiệm memory và CPU đáng kể (nhanh hơn 30-50%). `Compiled Queries` compile expression tree 1 lần, tái sử dụng. `Split Query` tránh cartesian explosion.

```csharp
// Compiled Query
private static readonly Func<AppDbContext, int, Task<Order?>> _getOrder =
    EF.CompileAsyncQuery((AppDbContext db, int id) =>
        db.Orders.AsNoTracking()
          .Include(o => o.Items)
          .FirstOrDefault(o => o.Id == id));

// Split Query - tránh cartesian explosion
var orders = await db.Orders
    .Include(o => o.Items)       // 1000 orders x 50 items = 50k rows nếu single query
    .Include(o => o.Payments)
    .AsSplitQuery()              // 3 query riêng, mỗi query nhỏ gọn
    .ToListAsync(ct);
```

Trade-off: `AsSplitQuery()` giảm data transfer nhưng tăng số roundtrip -- nếu latency đến DB cao thì single query có thể nhanh hơn. Compiled Query hữu ích nhất cho hot path được gọi hàng ngàn lần/giây. Nên set `AsNoTracking` ở DbContext level cho read-only context.

---

### Q27. How do you design an effective indexing strategy — composite, covering, and filtered indexes?

**Answer:**

Index strategy phải dựa trên actual query patterns, không phải đoán. Composite index thứ tự column cực kỳ quan trọng -- tuân theo quy tắc "equality first, range last, high selectivity trước".

```sql
-- Composite + Covering: tránh key lookup
CREATE INDEX IX_Orders_Status_CreatedAt
ON Orders (Status, CreatedAt DESC)
INCLUDE (Total, CustomerId);

-- Filtered: chỉ index subset data, nhỏ hơn và nhanh hơn
CREATE INDEX IX_Orders_Active
ON Orders (CreatedAt DESC)
WHERE Status = 'Active' AND IsDeleted = 0;
-- Chỉ hữu ích khi filter match < 30% total rows
```

Covering index chứa tất cả column cần thiết cho query, tránh key lookup về clustered index -- có thể cải thiện performance 10x cho analytical query. Filtered index nhỏ hơn, maintain nhanh hơn, nhưng chỉ được dùng khi query có WHERE clause match chính xác filter condition.

Pitfall: over-indexing làm chậm INSERT/UPDATE. Trong EF Core migration, dùng `HasIndex().HasFilter()` và `HasIndex().IncludeProperties()` (.NET 7+). Luôn kiểm tra index usage thực tế bằng `sys.dm_db_index_usage_stats` -- xóa index không được dùng.

---

### Q28. How do you detect and resolve the N+1 problem in EF Core?

**Answer:**

N+1 xảy ra khi load parent entity rồi lazy load child entity trong loop -- 1 query cho parent + N query cho mỗi child. Đây là "silent performance killer" phổ biến nhất.

```csharp
// N+1 Problem - 1 + N queries
var orders = await db.Orders.ToListAsync();
foreach (var order in orders)
    Console.WriteLine(order.Customer.Name); // Lazy load mỗi iteration!

// Fix 1: Eager loading
var orders = await db.Orders.Include(o => o.Customer).ToListAsync(ct);

// Fix 2: Projection - tốt nhất cho performance
var orders = await db.Orders
    .Select(o => new OrderDto
    {
        Id = o.Id,
        CustomerName = o.Customer.Name,
        ItemCount = o.Items.Count
    }).ToListAsync(ct);
```

Best practice: **tắt lazy loading hoàn toàn** bằng cách không install proxy package. Projection (Select) là giải pháp tốt nhất vì chỉ query đúng column cần thiết. Trong production, nên có automated test đếm số query cho critical endpoint.

---

### Q29. What strategies do you use for zero-downtime database migrations in production?

**Answer:**

Zero-downtime migration yêu cầu backward-compatible changes -- code cũ và code mới phải chạy đồng thời trên cùng schema (expand-contract pattern).

```
// Expand-Contract: đổi tên column "Name" -> "FullName"
// Phase 1 (Expand): Thêm column mới, deploy code đọc/ghi cả hai
ALTER TABLE Customers ADD FullName NVARCHAR(200);
UPDATE Customers SET FullName = Name;
-- Deploy code V2: write cả Name và FullName, read từ FullName

// Phase 2 (Contract): Sau khi 100% traffic dùng V2
ALTER TABLE Customers DROP COLUMN Name;
```

Nguyên tắc vàng: **không bao giờ** làm trong một migration: rename column, change type, drop column, add NOT NULL column without default. Mọi thao tác phải tách thành 2-3 migration riêng biệt.

Không dùng `dotnet ef database update` ở production. Export SQL script bằng `dotnet ef migrations script --idempotent`, review kỹ, rồi chạy qua CI/CD pipeline. Với database lớn, thêm column hoặc index phải dùng `ONLINE = ON` (SQL Server) để tránh lock table. Luôn có rollback script cho mỗi migration.

---

### Q30. Explain transaction isolation levels and how you handle concurrency conflicts in EF Core.

**Answer:**

SQL Server có 5 isolation level: Read Uncommitted, Read Committed (default), Repeatable Read, Serializable, và Snapshot. Mỗi level trade-off giữa consistency và concurrency/performance.

```csharp
// Optimistic concurrency với EF Core - phổ biến nhất cho web app
public class Order
{
    public int Id { get; set; }

    [Timestamp] // SQL Server rowversion - tự động
    public byte[] RowVersion { get; set; } = null!;
}

// Handling conflict
try { await db.SaveChangesAsync(ct); }
catch (DbUpdateConcurrencyException ex)
{
    var entry = ex.Entries.Single();
    var dbValues = await entry.GetDatabaseValuesAsync(ct);
    if (dbValues == null) { /* entity deleted */ return; }
    entry.OriginalValues.SetValues(dbValues);
    await db.SaveChangesAsync(ct); // retry với giá trị mới
}
```

Web application 99% dùng Optimistic Concurrency (`[Timestamp]`) vì request ngắn, conflict hiếm. Pessimistic locking (SELECT FOR UPDATE) chỉ dùng cho critical business logic như inventory deduction. Snapshot Isolation tốt cho reporting nhưng tốn thêm tempdb space. Pitfall: Serializable gây deadlock rất dễ trong high-concurrency.

---

### Q31. How do you approach query optimization — execution plans, parameter sniffing, and EF Core query analysis?

**Answer:**

Query optimization bắt đầu từ execution plan -- xem actual plan (không phải estimated) để biết query thực sự chạy thế nào. Tìm các operator tốn kém: Table Scan, Key Lookup, Sort (spill to tempdb).

```sql
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

-- Parameter sniffing problem:
-- SP compiled với @Status = 'Active' (90% rows) -> table scan plan
-- Khi gọi với @Status = 'VIP' (0.1% rows) -> vẫn dùng plan cũ!
SELECT * FROM Orders WHERE Status = @Status
OPTION (OPTIMIZE FOR (@Status UNKNOWN));
```

Trong EF Core, dùng `ToQueryString()` để xem generated SQL trước khi chạy. Dùng `Microsoft.EntityFrameworkCore.Diagnostics` để log slow query.

Checklist: (1) Kiểm tra logical reads trong STATISTICS IO, (2) Tránh SELECT * -- dùng projection, (3) Kiểm tra implicit conversion (nvarchar vs varchar gây index scan thay vì seek), (4) EF Core `HasConversion()` có thể gây implicit conversion SQL -- lỗi ẩn rất khó phát hiện.

---

### Q32. Explain connection pooling, DbContext lifetime management, and scaling strategies for database access.

**Answer:**

ADO.NET connection pooling mặc định bật -- pool size mặc định 100 per connection string. DbContext trong ASP.NET Core nên register Scoped (1 instance per request) -- đây là default của `AddDbContext`.

```csharp
// DbContext pooling - tái sử dụng instance, giảm allocation
builder.Services.AddDbContextPool<AppDbContext>(opt =>
    opt.UseSqlServer(connStr, sql =>
    {
        sql.EnableRetryOnFailure(3, TimeSpan.FromSeconds(5), null);
        sql.CommandTimeout(30);
    }), poolSize: 1024);
// Lưu ý: Pooled DbContext KHÔNG được inject scoped service vào constructor
```

`AddDbContextPool` (.NET 6+) giữ pool DbContext instance, reset state rồi tái sử dụng -- giảm GC pressure đáng kể ở high-throughput. Trade-off: không inject service vào DbContext constructor.

Scaling: Read replica cho read-heavy workload, sharding cho data lớn, CQRS tách read/write model hoàn toàn. Pitfall: connection pool exhaustion khi quên `await` async method hoặc DbContext sống quá lâu (Singleton DbContext là anti-pattern nghiêm trọng).

---

## SECTION 5: Security Principles, Data Protection & Compliance (Q33-Q40)

---

### Q33. Explain JWT, OAuth2, and OpenID Connect authentication flow in ASP.NET Core. What are common pitfalls?

**Answer:**

OAuth2 là authorization framework (cấp quyền truy cập resource), OpenID Connect (OIDC) là identity layer trên OAuth2 (xác thực user). JWT là token format phổ biến. Flow phổ biến nhất cho web app: Authorization Code + PKCE.

```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(opt =>
    {
        opt.Authority = "https://idp.example.com";
        opt.TokenValidationParameters = new()
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidAudiences = new[] { "orders-api" },
            ValidateLifetime = true,
            ClockSkew = TimeSpan.FromSeconds(30), // Giảm từ default 5 phút
        };
    });
```

Pitfalls nghiêm trọng: (1) Không validate `aud` claim -- token của API khác có thể dùng cho API bạn, (2) ClockSkew default 5 phút quá rộng, (3) Lưu sensitive data trong JWT payload -- nó chỉ Base64 encode, không encrypt, (4) Dùng symmetric key (HMAC) cho distributed system -- phải share secret, nên dùng RSA/ECDSA.

Access token nên short-lived (5-15 phút), refresh token long-lived + rotation. Không lưu token trong localStorage (XSS risk) -- dùng HttpOnly Secure cookie hoặc BFF pattern.

---

### Q34. Compare RBAC, Policy-based, and Resource-based authorization in ASP.NET Core. When do you use each?

**Answer:**

RBAC (Role-Based) đơn giản nhưng cứng nhắc. Policy-based linh hoạt hơn -- combine nhiều requirement. Resource-based kiểm tra quyền dựa trên data cụ thể (ví dụ: chỉ owner mới edit được order).

```csharp
// Policy-based: combine nhiều điều kiện
builder.Services.AddAuthorizationBuilder()
    .AddPolicy("SeniorManager", policy =>
        policy.RequireRole("Manager")
              .RequireClaim("experience_years")
              .AddRequirements(new MinExperienceRequirement(5)));

// Resource-based: kiểm tra ownership
public class OrderAuthorizationHandler
    : AuthorizationHandler<EditRequirement, Order>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext ctx, EditRequirement req, Order order)
    {
        if (ctx.User.FindFirst("sub")?.Value == order.OwnerId.ToString()
            || ctx.User.IsInRole("Admin"))
            ctx.Succeed(req);
        return Task.CompletedTask;
    }
}
```

Thực tế: dùng RBAC cho coarse-grained access (admin vs user), Policy-based cho business rules phức tạp, Resource-based cho row-level security. Đừng hardcode role string khắp nơi -- tập trung vào policy name. Pitfall: `[Authorize(Roles = "Admin")]` scattered khắp codebase rất khó maintain.

---

### Q35. How do you prevent OWASP Top 10 vulnerabilities in a .NET Core application?

**Answer:**

OWASP Top 10 (2021): Broken Access Control, Cryptographic Failures, Injection, Insecure Design, Security Misconfiguration, Vulnerable Components, Authentication Failures, Data Integrity Failures, Logging Failures, SSRF.

```csharp
// 1. SQL Injection - NGUY HIỂM vs AN TOÀN
// NGUY HIỂM: FromSqlRaw($"SELECT * FROM Users WHERE Name = '{input}'")
// AN TOÀN:
db.Users.FromSqlInterpolated($"SELECT * FROM Users WHERE Name = {input}");

// 2. Mass Assignment / Over-posting
[HttpPost]
public IActionResult Create([Bind("Name,Email")] UserDto dto)
// Hoặc tốt hơn: dùng DTO riêng cho input, không bind trực tiếp Entity

// 3. Security headers
builder.Services.AddHsts(opt => { opt.MaxAge = TimeSpan.FromDays(365); });
```

Prevention checklist: (1) `[ValidateAntiForgeryToken]` hoặc global filter, (2) Content-Security-Policy header, (3) Enable HSTS, (4) Parameterized query 100%, (5) `dotnet list package --vulnerable` thường xuyên, (6) FluentValidation cho mọi input, (7) Không expose stack trace ở production -- dùng `ProblemDetails`. Nguyên tắc defense-in-depth: không bao giờ tin tưởng chỉ một layer bảo vệ.

---

### Q36. Explain data encryption strategies: at rest, in transit, and column-level encryption in .NET Core.

**Answer:**

In transit: TLS 1.2+ bắt buộc. At rest: SQL Server TDE encrypt toàn bộ database file. Column-level: encrypt các column nhạy cảm riêng biệt.

```csharp
// Column-level encryption với Data Protection API
public class EncryptionService(IDataProtector protector)
{
    public string Encrypt(string plainText) => protector.Protect(plainText);
    public string Decrypt(string cipherText) => protector.Unprotect(cipherText);
}

// EF Core Value Converter cho automatic encrypt/decrypt
modelBuilder.Entity<Customer>()
    .Property(c => c.SSN)
    .HasConversion(
        v => _encryptor.Encrypt(v),
        v => _encryptor.Decrypt(v))
    .HasMaxLength(500); // Encrypted data lớn hơn plaintext
```

Trade-off column-level: không thể query/sort/filter trên encrypted column (trừ Always Encrypted deterministic mode cho equality check). Data Protection API tự động rotate key nhưng cần configure persistent key storage (Azure Blob, Redis) -- nếu không, restart app = mất key = mất data.

Pitfall: TDE không bảo vệ data trong memory hay query result -- DBA vẫn đọc được. Always Encrypted bảo vệ cả khỏi DBA nhưng hạn chế query capability.

---

### Q37. How do you manage secrets in .NET Core across different environments?

**Answer:**

Hierarchy ưu tiên (cao đến thấp): Environment Variables > User Secrets (dev) > appsettings.{Env}.json > appsettings.json. Trong production, dùng Azure Key Vault / AWS Secrets Manager -- KHÔNG BAO GIỜ commit secrets vào source control.

```csharp
// Production: Azure Key Vault
builder.Configuration.AddAzureKeyVault(
    new Uri("https://myvault.vault.azure.net/"),
    new DefaultAzureCredential()); // Managed Identity - no secrets needed!

// Binding vào strongly-typed options
builder.Services.AddOptions<DatabaseOptions>()
    .BindConfiguration("Database")
    .ValidateDataAnnotations()
    .ValidateOnStart(); // Fail fast nếu config invalid

public class DatabaseOptions
{
    [Required] public string ConnectionString { get; set; } = null!;
    [Range(1, 200)] public int MaxPoolSize { get; set; } = 100;
}
```

Pitfalls: (1) Log configuration values ra console/file -- vô tình expose secrets, (2) Dùng appsettings.json cho connection string production, (3) Share Key Vault giữa environments, (4) Quên `ValidateOnStart()` -- app chạy nhưng crash khi cần config lần đầu. Thêm `.gitignore` rule và scan repo bằng `gitleaks` trong CI.

---

### Q38. How do you configure CORS, CSP, and security headers properly in ASP.NET Core?

**Answer:**

CORS kiểm soát domain nào được gọi API. CSP kiểm soát resource nào browser được load. Security headers bảo vệ chống clickjacking, MIME sniffing, etc.

```csharp
// CORS - cụ thể, KHÔNG dùng AllowAnyOrigin + AllowCredentials cùng lúc
builder.Services.AddCors(opt =>
{
    opt.AddPolicy("Production", policy =>
        policy.WithOrigins("https://app.example.com")
              .WithMethods("GET", "POST", "PUT", "DELETE")
              .WithHeaders("Authorization", "Content-Type")
              .SetPreflightMaxAge(TimeSpan.FromHours(1)));
});

// Security headers middleware
app.Use(async (ctx, next) =>
{
    ctx.Response.Headers.Append("X-Content-Type-Options", "nosniff");
    ctx.Response.Headers.Append("X-Frame-Options", "DENY");
    ctx.Response.Headers.Append("Referrer-Policy", "strict-origin-when-cross-origin");
    ctx.Response.Headers.Append("Content-Security-Policy",
        "default-src 'self'; script-src 'self'; frame-ancestors 'none'");
    ctx.Response.Headers.Append("Permissions-Policy", "camera=(), microphone=()");
    await next();
});
```

Pitfalls: (1) `AllowAnyOrigin()` trong production, (2) Thiếu `frame-ancestors 'none'` trong CSP -- dễ bị clickjacking, (3) CORS chỉ là browser enforcement -- server-to-server request bypass hoàn toàn, không thay thế authentication. Nên dùng library `NWebsec` hoặc `NetEscapades.AspNetCore.SecurityHeaders`.

---

### Q39. How do you implement robust input validation and prevent anti-tampering in .NET Core APIs?

**Answer:**

Validation phải thực hiện ở nhiều layer: client-side (UX), API input validation (security), domain validation (business rules), database constraints (last line of defense). Không bao giờ tin client input.

```csharp
// FluentValidation - khai báo rõ ràng, testable
public class CreateOrderValidator : AbstractValidator<CreateOrderRequest>
{
    public CreateOrderValidator()
    {
        RuleFor(x => x.CustomerId).NotEmpty();
        RuleFor(x => x.Items).NotEmpty().Must(items => items.Count <= 100)
            .WithMessage("Maximum 100 items per order");
        RuleForEach(x => x.Items).ChildRules(item =>
        {
            item.RuleFor(i => i.Quantity).InclusiveBetween(1, 999);
            item.RuleFor(i => i.Price).GreaterThan(0).PrecisionScale(10, 2, true);
        });
    }
}

// Anti-tampering: KHÔNG tin client-sent price, re-calculate server-side
public async Task<Order> CreateOrder(CreateOrderRequest req)
{
    var product = await _db.Products.FindAsync(req.ProductId);
    var total = product.Price * req.Quantity; // Server-side price, không dùng req.Price
}
```

Anti-tampering patterns: không trust hidden fields (price, userId), dùng server-side state cho mọi thứ critical, implement idempotency key để prevent duplicate submission, validate file upload (check magic bytes, không chỉ extension). Pitfall: chỉ validate ở client -- attacker bypass JavaScript dễ dàng bằng Postman/curl.

---

### Q40. How do you implement audit logging and ensure compliance with GDPR/PCI-DSS basics in .NET Core?

**Answer:**

Audit logging ghi lại WHO did WHAT to WHICH resource WHEN. GDPR yêu cầu right to access, right to erasure, data portability. PCI-DSS yêu cầu encrypt cardholder data, restrict access, audit trail.

```csharp
// EF Core interceptor cho automatic audit logging
public class AuditInterceptor : SaveChangesInterceptor
{
    public override ValueTask<InterceptionResult<int>> SavingChangesAsync(
        DbContextEventData data, InterceptionResult<int> result, CancellationToken ct)
    {
        foreach (var entry in data.Context!.ChangeTracker.Entries()
            .Where(e => e.State is EntityState.Added or EntityState.Modified
                        or EntityState.Deleted))
        {
            data.Context.Set<AuditLog>().Add(new AuditLog
            {
                EntityType = entry.Entity.GetType().Name,
                Action = entry.State.ToString(),
                Changes = JsonSerializer.Serialize(entry.Properties
                    .Where(p => p.IsModified)
                    .ToDictionary(p => p.Metadata.Name, p => p.CurrentValue)),
                UserId = _httpContext.User.FindFirst("sub")?.Value,
                Timestamp = DateTimeOffset.UtcNow,
            });
        }
        return base.SavingChangesAsync(data, result, ct);
    }
}

// GDPR: Right to Erasure (soft delete + anonymize)
public async Task AnonymizeUser(Guid userId)
{
    var user = await _db.Users.FindAsync(userId);
    user.Email = $"deleted-{Guid.NewGuid()}@anonymized.local";
    user.Name = "REDACTED";
    user.Phone = null;
}
```

PCI-DSS: không bao giờ lưu CVV, mask card number (chỉ hiển thị 4 số cuối), dùng tokenization (Stripe, Braintree) thay vì tự xử lý card data -- giảm PCI scope đáng kể.

Audit log phải immutable (append-only table, không UPDATE/DELETE), lưu riêng biệt database/service, và retain theo compliance requirement (PCI: 1 năm online, 7 năm archive). Pitfall: log PII vào audit trail rồi không thể xóa khi user request GDPR erasure -- design audit log để separate PII từ đầu.

---

## SECTION 6: Problem-Solving, Debugging & Troubleshooting (Q41-Q48)

---

### Q41. Trình bày phương pháp có hệ thống để debug một API endpoint chậm (response > 5 giây)?

**Answer:**

Tiếp cận theo **top-down, đo trước khi đoán**. Không bao giờ đoán nguyên nhân rồi fix -- phải có data chứng minh.

**Bước 1 - Reproduce & Measure:** Xác nhận vấn đề. Kiểm tra có phải chậm consistent hay chỉ intermittent.

**Bước 2 - Xác định layer nào chậm:**
```csharp
app.Use(async (context, next) =>
{
    var sw = Stopwatch.StartNew();
    await next();
    Log.Information("Request {Method} {Path} took {Elapsed}ms",
        context.Request.Method, context.Request.Path, sw.ElapsedMilliseconds);
});
```

**Bước 3 - Thu hẹp phạm vi:**
- **Database chậm?** Bật EF Core logging, kiểm tra query plan, tìm N+1, missing index.
- **External service chậm?** Kiểm tra timeout, thêm circuit breaker, cân nhắc cache.
- **Business logic nặng?** Profile CPU, kiểm tra vòng lặp O(n^2), large object allocation.
- **Network/infra?** Kiểm tra DNS resolution, connection pool exhaustion, container resource limits.

**Bước 4 - Fix & Verify:** Áp dụng fix, đo lại với cùng điều kiện, so sánh before/after.

Kinh nghiệm thực tế: 80% API chậm là do database -- N+1 query, missing index, hoặc load quá nhiều data không cần thiết (SELECT * thay vì projection). Luôn kiểm tra DB trước.

---

### Q42. Làm thế nào để phát hiện và xử lý memory leak trong .NET Core? Dùng tool gì?

**Answer:**

Memory leak trong .NET thường do: **event handler không unsubscribe**, **static collection tích lũy**, **IDisposable không dispose** (HttpClient, DbContext), hoặc **closure capture** giữ reference ngoài ý muốn.

**Phát hiện bằng dotnet-counters (không cần restart app):**
```bash
dotnet-counters monitor --process-id <PID> \
    --counters System.Runtime[gc-heap-size,gen-2-gc-count,gen-2-size]
```
Nếu `gc-heap-size` tăng liên tục sau mỗi Gen 2 GC, rất có thể có leak.

**Phân tích bằng dotnet-dump:**
```bash
dotnet-dump collect --process-id <PID>
dotnet-dump analyze <dump-file>
> dumpheap -stat          # xem object nào nhiều nhất
> dumpheap -type MyClass  # tìm instance cụ thể
> gcroot <address>        # tìm ai đang giữ reference
```

**Phòng ngừa:** Dùng `IHttpClientFactory` thay vì `new HttpClient()` (tránh socket exhaustion). Đăng ký DbContext là Scoped, không phải Singleton. Pitfall: `MemoryCache` không giới hạn size mặc định -- phải set `SizeLimit` hoặc `AbsoluteExpiration`, nếu không cache sẽ grow vô hạn.

---

### Q43. Trình bày quy trình xử lý incident production? OODA loop áp dụng như thế nào?

**Answer:**

**OODA loop** (Observe - Orient - Decide - Act) áp dụng cho incident response:

**Observe:** Phát hiện qua alert, user report, hoặc health check fail. Xác định **severity**:
- **SEV1:** System down, mất doanh thu -> respond ngay, escalate.
- **SEV2:** Feature chính bị lỗi -> respond trong 30 phút.
- **SEV3:** Feature phụ bị lỗi, workaround có sẵn -> respond trong giờ làm việc.

**Orient:** Deploy gần đây? Traffic spike? DB change? Dependency bị lỗi?

**Decide:** Chọn giữa rollback, hotfix, scale up, hoặc toggle feature flag. Ưu tiên **khôi phục service trước, tìm root cause sau**.

**Act:** Thực hiện mitigation. Một người fix, một người communication (cập nhật status page, thông báo stakeholder).

**Communication template:**
```
[SEV2] Order Service - Elevated Error Rate
Impact: 15% orders failing at checkout
Status: Investigating - suspected DB connection pool exhaustion
ETA: 30 minutes
Next update: 14:30
```

Nguyên tắc vàng: **MTTD + MTTR** quan trọng hơn MTBF. Invest vào monitoring và runbook hơn là cố gắng prevent mọi failure.

---

### Q44. Làm thế nào phát hiện N+1 query trong EF Core mà không cần profiler bên ngoài?

**Answer:**

N+1 xảy ra khi load một list entity rồi lazy-load navigation property trong vòng lặp -- 1 query cho list + N query cho mỗi item.

**Cách 1 - Bật EF Core logging:**
```csharp
optionsBuilder
    .LogTo(Console.WriteLine, LogLevel.Information)
    .EnableSensitiveDataLogging()
    .EnableDetailedErrors();
```

**Cách 2 - EF Core Interceptor đếm query per request:**
```csharp
public class QueryCountInterceptor : DbCommandInterceptor
{
    private static readonly AsyncLocal<int> _count = new();
    public static int Count => _count.Value;
    public static void Reset() => _count.Value = 0;

    public override ValueTask<InterceptionResult<DbDataReader>> ReaderExecutingAsync(
        DbCommand cmd, CommandEventData data, InterceptionResult<DbDataReader> result,
        CancellationToken ct)
    {
        _count.Value++;
        return base.ReaderExecutingAsync(cmd, data, result, ct);
    }
}
// Trong middleware: reset đầu request, log cuối request. Alert nếu > 10 queries.
```

**Cách 3:** Dùng `ConfigureWarnings` để throw exception khi lazy loading xảy ra:
```csharp
optionsBuilder.ConfigureWarnings(w =>
    w.Throw(CoreEventId.NavigationLazyLoading));
```

**Fix:** `.Include()` / `.ThenInclude()` (eager loading), hoặc tốt hơn là **projection** với `.Select()` để chỉ lấy đúng field cần.

---

### Q45. Giải thích cách chẩn đoán deadlock trong SQL Server và trong async code .NET?

**Answer:**

Đây là hai loại deadlock khác nhau nhưng đều gây hệ thống treo.

**SQL Server Deadlock:** Hai transaction lock resource theo thứ tự ngược nhau. Chẩn đoán: bật **Deadlock Graph** qua Extended Events, hoặc query `sys.dm_exec_requests`. Fix: đảm bảo các transaction access table/row **theo cùng thứ tự**, giảm transaction scope, dùng `SNAPSHOT ISOLATION` khi phù hợp.

**Async Deadlock trong .NET:** Phổ biến nhất là gọi `.Result` hoặc `.Wait()` trên async method trong synchronization context (ASP.NET Framework cũ, WPF, WinForms):
```csharp
// DEADLOCK trong ASP.NET Framework (KHÔNG xảy ra trong ASP.NET Core)
public IActionResult Get()
{
    var data = GetDataAsync().Result;  // block thread, giữ sync context
    return Ok(data);
}

// FIX: async all the way
public async Task<IActionResult> Get()
{
    var data = await GetDataAsync();
    return Ok(data);
}
```

ASP.NET Core không có `SynchronizationContext` nên ít gặp dạng này, nhưng vẫn có thể deadlock khi dùng `SemaphoreSlim` hoặc custom sync primitives sai cách. Nguyên tắc: **async all the way** -- không bao giờ mix sync và async code.

---

### Q46. Làm sao debug hiệu quả trong hệ thống distributed? Correlation ID, OpenTelemetry và structured logging hoạt động ra sao?

**Answer:**

Trong distributed system, một request đi qua 5-10 service. Không có correlation thì log chỉ là đống text vô nghĩa. Ba trụ cột observability: **Logs**, **Metrics**, **Traces**.

**Correlation ID** -- gán một ID duy nhất cho mỗi request, truyền qua mọi service:
```csharp
app.Use(async (context, next) =>
{
    var correlationId = context.Request.Headers["X-Correlation-Id"]
        .FirstOrDefault() ?? Guid.NewGuid().ToString();
    using (LogContext.PushProperty("CorrelationId", correlationId))
    {
        context.Response.Headers["X-Correlation-Id"] = correlationId;
        await next();
    }
});
```

**Structured Logging** (Serilog) -- log dạng key-value để có thể query:
```csharp
// Thay vì: Log.Info($"Order {orderId} created by {userId}")
Log.Information("Order created {@OrderId} by {@UserId}", orderId, userId);
// => Query trong Seq/Kibana: WHERE OrderId = 'xxx' AND CorrelationId = 'yyy'
```

**OpenTelemetry** -- standard cho distributed tracing:
```csharp
builder.Services.AddOpenTelemetry()
    .WithTracing(t => t
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .AddSqlClientInstrumentation()
        .AddOtlpExporter());  // xuất sang Jaeger, Zipkin, hoặc Grafana Tempo
```

Khi debug incident: tìm correlation ID từ error log, query trace để thấy toàn bộ call chain, xác định service/span nào chậm hoặc lỗi.

---

### Q47. Race condition là gì? Cho ví dụ trong .NET và cách xử lý?

**Answer:**

Race condition xảy ra khi nhiều thread/request truy cập shared resource đồng thời và kết quả phụ thuộc vào thứ tự thực thi. Đây là loại bug nguy hiểm vì **khó reproduce**.

**Ví dụ kinh điển -- double spending:**
```csharp
// Hai request đồng thời rút tiền từ cùng account
public async Task Withdraw(Guid accountId, decimal amount)
{
    var account = await _db.Accounts.FindAsync(accountId);
    if (account.Balance >= amount)   // cả 2 request đều thấy balance = 1000
    {
        account.Balance -= amount;   // cả 2 đều trừ 800 -> balance = 200
        await _db.SaveChangesAsync();// nhưng đúng ra phải fail request thứ 2
    }
}
```

**Giải pháp theo cấp độ:**

1. **Optimistic Concurrency (phổ biến nhất):** EF Core `[Timestamp]` / `[ConcurrencyCheck]` -- SaveChanges throw `DbUpdateConcurrencyException` nếu data đã bị thay đổi -> retry.

2. **Pessimistic Locking:** `SELECT ... WITH (UPDLOCK, ROWLOCK)` -- lock row trong DB. Dùng khi conflict rate cao.

3. **Distributed Lock** (Redis): Khi xử lý cross-service, dùng `RedLock` algorithm.

4. **Idempotency Key:** Cho mỗi operation một unique key, check trước khi execute -- đặc biệt quan trọng cho payment.

Trade-off: Optimistic lock đơn giản, performance tốt khi conflict ít. Pessimistic lock chắc chắn hơn nhưng giảm throughput. 90% trường hợp dùng optimistic concurrency + retry là đủ.

---

### Q48. Post-mortem analysis nên thực hiện như thế nào? Blameless culture có nghĩa là gì trong thực tế?

**Answer:**

Post-mortem là quá trình phân tích incident để **học hỏi và ngăn chặn tái diễn**, không phải để quy trách nhiệm.

**Blameless culture** nghĩa là: tập trung vào **hệ thống và quy trình** đã fail, không phải **con người**. Một engineer deploy code lỗi lên production không phải là root cause -- root cause là: tại sao code review không catch? Tại sao không có automated test? Tại sao deploy process cho phép push thẳng lên prod?

**Template post-mortem:**
```markdown
## Incident: [Tên] - [Ngày]
## Severity: SEV2 | Duration: 45 phút | Users affected: ~2000

## Timeline (UTC)
- 14:00 - Deploy version 2.3.1
- 14:05 - Alert: error rate > 5%
- 14:12 - On-call engineer acknowledged
- 14:15 - Identified: missing DB migration
- 14:20 - Rollback initiated
- 14:25 - Service restored

## Root Cause
Migration script không được include trong deployment pipeline.

## Contributing Factors
- Không có checklist cho DB migration trong deploy process
- Staging environment đã apply migration thủ công trước đó

## Action Items
- [ ] Thêm migration check vào CI/CD pipeline (owner: DevOps, deadline: 2 tuần)
- [ ] Tạo runbook cho DB migration rollback (owner: Backend lead, deadline: 1 tuần)
- [ ] Thêm health check endpoint kiểm tra DB schema version (owner: Dev team)

## Lessons Learned
- Manual steps in deployment = ticking time bomb
- Staging phải reflect production process chính xác
```

Quy tắc: post-mortem phải được thực hiện trong vòng **48 giờ** sau incident. Mọi action item phải có **owner** và **deadline** cụ thể. Review lại action items trong sprint planning để đảm bảo được thực hiện.

---

> **Summary:** 48 câu hỏi, 6 sections, mỗi section 8 câu. Các câu trả lời được viết ở mức senior với trade-offs, code examples, và kinh nghiệm thực tế. Dùng để ôn tập phỏng vấn vị trí Senior Full-Stack Developer (.NET Core + ReactJS).
