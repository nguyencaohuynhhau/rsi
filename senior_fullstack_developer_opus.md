# Senior Full-Stack Developer Interview (.NET Core + ReactJS)

> **Position:** Senior Full-Stack Developer (4+ years)
> **Stack:** .NET Core (C#) + ReactJS (TypeScript)
> **Format:** 48 questions (6 sections x 8 questions), senior-level answers with trade-offs and real-world context

---

## SECTION 1: C#, JavaScript/TypeScript & Modern Web Technologies (Q1-Q8)

---

### Q1. So sanh cac tinh nang nang cao cua C# (records, pattern matching, nullable reference types) voi cac tinh nang tuong duong trong TypeScript. Khi nao ban su dung tung tinh nang trong du an full-stack?

**Answer:**

Records trong C# va `type`/`interface` trong TypeScript deu dung de dinh nghia data model, nhung C# records cung cap value equality, immutability va `with` expression co san o muc ngon ngu, trong khi TypeScript can thu vien ben ngoai hoac convention thu cong.

```csharp
// C# (.NET 8) - Record voi pattern matching
public record OrderDto(string Id, decimal Total, OrderStatus Status);

public string GetStatusMessage(OrderDto order) => order switch
{
    { Status: OrderStatus.Pending, Total: > 1000 } => "Don lon cho duyet",
    { Status: OrderStatus.Shipped } => "Da giao",
    _ => "Dang xu ly"
};

// Nullable reference types - bat trong .csproj
public string? GetCustomerName(int id) // compiler canh bao neu dung ma khong check null
```

```typescript
// TypeScript 5+ - Discriminated union thay the pattern matching
type Order =
  | { status: "pending"; total: number }
  | { status: "shipped"; trackingId: string };

function getStatusMessage(order: Order): string {
  switch (order.status) {
    case "pending": return order.total > 1000 ? "Don lon cho duyet" : "Cho xu ly";
    case "shipped": return `Da giao: ${order.trackingId}`; // TypeScript tu narrowing type
  }
}
```

**Trade-off:** C# records manh hon ve immutability (init-only properties, positional records) nhung TypeScript discriminated unions linh hoat hon cho UI state machine. Nullable reference types trong C# la compile-time check tuong tu TypeScript strict mode -- ca hai deu nen bat tu dau du an, bat giua chung se sinh ra hang tram warning kho xu ly dan.

---

### Q2. Giai thich su khac biet cot loi giua async/await trong C# va JavaScript. Tai sao hieu sai mo hinh threading co the gay bug nghiem trong trong ung dung full-stack?

**Answer:**

Diem khac biet quan trong nhat: C# async/await chay tren thread pool da luong thuc su, con JavaScript async/await chay tren single-threaded event loop. Trong C#, `await` giai phong thread hien tai ve pool va continuation co the chay tren thread khac. Trong JavaScript, `await` chi nhuong quyen lai cho event loop, moi thu van tren mot thread duy nhat.

```csharp
// C# - NGUY HIEM: deadlock trong ASP.NET (legacy SynchronizationContext)
public string GetData()
{
    // KHONG BAO GIO LAM THE NAY - .Result block thread, gay deadlock
    var result = _httpClient.GetStringAsync("/api/data").Result;
    return result;
}

// DUNG: async xuyen suot (async all the way)
public async Task<string> GetDataAsync()
{
    var result = await _httpClient.GetStringAsync("/api/data");
    // Trong .NET 8 minimal API, khong co SynchronizationContext
    // nen ConfigureAwait(false) khong con can thiet o tang API
    return result;
}
```

```typescript
// JavaScript - Khong co deadlock kieu C# nhung co pitfall rieng
// SAI: forEach khong await duoc
async function processOrders(ids: string[]) {
  ids.forEach(async (id) => {
    await fetchOrder(id); // Fire-and-forget, khong cho thuc su!
  });
}

// DUNG: dung for...of (tuan tu) hoac Promise.all (song song)
async function processOrders(ids: string[]) {
  await Promise.all(ids.map((id) => fetchOrder(id))); // song song
  // hoac: for (const id of ids) { await fetchOrder(id); } // tuan tu
}
```

**Pitfall thuc te:** Trong C#, quen `await` mot Task se nuot exception am tham (fire-and-forget). Trong JavaScript, unhandled promise rejection gio crash process trong Node.js. Khi lam full-stack, loi pho bien nhat la dev quen JavaScript nghi C# async cung single-thread, dan den race condition khi truy cap shared state ma khong dung lock hoac ConcurrentDictionary.

---

### Q3. React 18+ gioi thieu Concurrent Rendering voi useTransition va useDeferredValue. Giai thich cach hoat dong, so sanh voi debounce truyen thong, va cho vi du tich hop voi .NET API.

**Answer:**

Concurrent Rendering cho phep React tam dung render khong quan trong de uu tien tuong tac nguoi dung (nhu go phim). Khac voi debounce (tri hoan thuc thi), useTransition danh dau state update la "low priority" -- React van bat dau render ngay nhung co the interrupt neu co update uu tien cao hon.

```typescript
// useTransition - cho action chu dong (user trigger)
function SearchPage() {
  const [query, setQuery] = useState("");
  const [results, setResults] = useState<ProductDto[]>([]);
  const [isPending, startTransition] = useTransition();

  const handleSearch = (value: string) => {
    setQuery(value); // HIGH priority - input phan hoi ngay
    startTransition(async () => {
      // LOW priority - React co the interrupt render nay
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

// useDeferredValue - cho gia tri bi dong (derived/prop)
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
// .NET 8 API endpoint ho tro search
app.MapGet("/api/products", async (string? q, AppDbContext db, CancellationToken ct) =>
{
    // CancellationToken quan trong: khi React abort request cu, server cung dung query
    var query = db.Products.AsNoTracking();
    if (!string.IsNullOrEmpty(q))
        query = query.Where(p => EF.Functions.ILike(p.Name, $"%{q}%"));
    return await query.Take(50).ToListAsync(ct);
});
```

**Trade-off:** useTransition phu hop cho search/filter nang, nhung khong thay the debounce cho API call -- nen ket hop ca hai: debounce (300ms) de giam request den server, useTransition de giu UI responsive trong khi render ket qua. useDeferredValue phu hop hon khi data den tu prop hoac external store ma ban khong kiem soat thoi diem update.

---

### Q4. So sanh he thong Dependency Injection cua .NET Core voi cac giai phap state/dependency management trong React (Context API, Zustand, Jotai). Khi nao dung cai nao?

**Answer:**

.NET Core DI la IoC container hoan chinh quan ly lifetime (Singleton, Scoped, Transient) o tang application, con React Context/Zustand quan ly state va dependency o tang UI component tree. Chung giai quyet bai toan khac nhau nhung cung nguyen ly: tach dependency ra khoi noi su dung.

```csharp
// .NET 8 DI - Keyed Services (moi trong .NET 8)
builder.Services.AddKeyedSingleton<ICache, RedisCache>("redis");
builder.Services.AddKeyedSingleton<ICache, MemoryCache>("memory");

builder.Services.AddScoped<IOrderService, OrderService>();
// Scoped = 1 instance per HTTP request, quan trong cho DbContext

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
// React - Context cho dependency thuc su (services, config)
// Zustand cho UI state (nhanh, it boilerplate)

import { create } from "zustand";

interface OrderStore {
  orders: Order[];
  fetchOrders: () => Promise<void>;
}

const useOrderStore = create<OrderStore>((set) => ({
  orders: [],
  fetchOrders: async () => {
    const data = await api.get<Order[]>("/api/orders");
    set({ orders: data });  // Chi component subscribe orders moi re-render
  },
}));

// Context - cho DI pattern (API client, theme, auth)
const ApiContext = createContext<ApiClient>(null!);
```

**Khi nao dung gi:** React Context phu hop cho gia tri it thay doi (theme, auth, API client) vi moi lan value thay doi se re-render toan bo consumer. Zustand/Jotai cho state thay doi thuong xuyen (form, filter, list). Sai lam pho bien la nhet moi thu vao Context gay re-render cascade, hoac dung Redux cho project nho khi Zustand chi can 10 dong code. O backend, pitfall la dang ky DbContext lam Singleton thay vi Scoped -- gay memory leak va stale data.

---

### Q5. Danh gia cac giai phap CSS trong React hien dai: CSS Modules, Tailwind CSS, va CSS-in-JS. Du an nao nen dung giai phap nao?

**Answer:**

Ba huong tiep can chinh khac nhau o runtime cost, developer experience, va kha nang scale. CSS Modules la zero-runtime voi scoped class names. Tailwind la utility-first voi build-time purging. CSS-in-JS cho phep dynamic styling nhung co runtime overhead.

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

// 3. CSS-in-JS (Emotion) - dynamic styling manh nhung co runtime cost
import styled from "@emotion/styled";
const Button = styled.button<{ $size: number }>`
  padding: ${(p) => p.$size * 4}px ${(p) => p.$size * 8}px;
  background: ${(p) => p.theme.colors.primary};
`;
```

| Tieu chi | CSS Modules | Tailwind | CSS-in-JS |
|---|---|---|---|
| Runtime cost | Zero | Zero | Co (parse + inject) |
| Bundle size | Nho | Rat nho (purged) | Lon hon |
| Dynamic styling | Kho | Can workaround | Rat manh |
| Server Components | Tuong thich | Tuong thich | Hau het KHONG tuong thich |

**Khuyen nghi:** Tailwind cho phan lon du an moi (fast iteration, nho gon, tuong thich RSC). CSS Modules cho du an can tach biet ro rang styling voi logic. CSS-in-JS dang giam xu huong vi khong tuong thich React Server Components -- neu can dynamic styling, xem xet Panda CSS hoac Vanilla Extract (zero-runtime CSS-in-JS).

---

### Q6. Thiet ke kien truc real-time feature su dung SignalR (.NET) va React. Neu cac gotcha pho bien va cach xu ly.

**Answer:**

SignalR cung cap abstraction tren WebSocket voi fallback (Server-Sent Events, Long Polling) va tich hop san voi .NET authentication/authorization. Kien truc tot can xu ly reconnection, message ordering, va cleanup dung cach o ca hai phia.

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
    return () => { connection.stop(); }; // QUAN TRONG: cleanup khi unmount
  }, [connection]);

  return connection;
}

// Usage - QUAN TRONG: dung functional update, tranh stale closure
useEffect(() => {
  connection.on("ReceiveMessage", (msg: ChatMessage) => {
    setMessages(prev => [...prev, msg]);
  });
  return () => { connection.off("ReceiveMessage"); };
}, [connection]);
```

**Gotcha pho bien:** (1) Quen cleanup `connection.off()` gay memory leak va duplicate messages. (2) Stale closure -- handler bat state cu, phai dung functional update hoac useRef. (3) Khi scale nhieu server instance, phai dung Redis backplane, khong thi message chi den client ket noi cung instance. (4) Khong xu ly reconnection -- user mat ket noi 30 giay se miss messages, can fetch lai history sau reconnect.

---

### Q7. Giai thich cach dat full-stack type safety giua .NET API va React frontend bang OpenAPI code generation. So sanh NSwag vs openapi-typescript.

**Answer:**

Full-stack type safety nghia la khi thay doi API contract o backend, frontend se bao loi compile-time thay vi crash runtime. Cach tiep can pho bien nhat la generate OpenAPI spec tu .NET, sau do generate TypeScript types/client tu spec do.

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
| Bundle size | Lon (generated code) | Rat nho (types erased khi compile) |
| Customization | It linh hoat | Dung fetch native, de customize |
| CI integration | Can .NET runtime | Chi can Node.js |

**Khuyen nghi:** openapi-typescript cho du an moi vi lightweight, tree-shakeable. NSwag phu hop neu team da quen hoac can generate C# client cho microservice-to-microservice. Quan trong nhat: tich hop vao CI -- moi lan build se re-generate types, neu frontend khong compile duoc thi pipeline fail som.

---

### Q8. Span<T> va Memory<T> trong C# giai quyet bai toan gi? So sanh voi cach JavaScript xu ly binary data (ArrayBuffer, TypedArray). Cho vi du full-stack xu ly file upload.

**Answer:**

`Span<T>` cho phep lam viec voi vung nho lien tuc (contiguous memory) ma khong can allocate array moi, giam GC pressure dang ke. No la stack-only (ref struct), khong the dung trong async method -- khi can async thi dung `Memory<T>`. JavaScript co `ArrayBuffer` + `TypedArray` tuong tu nhung khong co zero-allocation slicing nhu Span.

```csharp
// .NET 8 - Parse CSV header khong allocate string moi
public static IEnumerable<Range> ParseCsvHeaders(ReadOnlySpan<char> line)
{
    var ranges = new List<Range>();
    int start = 0;
    for (int i = 0; i <= line.Length; i++)
    {
        if (i == line.Length || line[i] == ',')
        {
            ranges.Add(start..i);  // Zero allocation - chi luu range
            start = i + 1;
        }
    }
    return ranges;
}

// File upload endpoint voi streaming (khong load toan bo file vao RAM)
app.MapPost("/api/upload", async (HttpRequest request, CancellationToken ct) =>
{
    await using var stream = File.Create($"/uploads/{Guid.NewGuid()}");
    await request.Body.CopyToAsync(stream, ct);
    return Results.Ok();
});
```

```typescript
// React - File upload voi progress va chunk processing
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

**Diem cot loi:** `Span<T>` giam allocation trong hot path (parsing, serialization) -- benchmark cho thay giam 40-60% GC pause. JavaScript `TypedArray.subarray()` tuong duong Span (shared buffer) con `slice()` tuong duong `ToArray()` (copy). Pitfall: Span khong dung duoc trong async/await, closure, hoac LINQ -- khi can thi chuyen sang `Memory<T>`.

---

## SECTION 2: Software Design Patterns, Data Structures & Algorithms (Q9-Q16)

---

### Q9. Hay cho vi du thuc te ve vi pham nguyen tac SOLID trong du an .NET Core va cach khac phuc?

**Answer:**

Vi pham pho bien nhat trong thuc te la **Single Responsibility** va **Dependency Inversion**. Mot controller "beo" vua validate, vua chua business logic, vua goi thang DbContext la dau hieu ro nhat.

```csharp
// VI PHAM: Controller lam qua nhieu viec
public class OrderController : ControllerBase
{
    private readonly AppDbContext _db;
    public async Task<IActionResult> Create(OrderDto dto)
    {
        // validate, tinh gia, apply discount, goi payment, save DB, send email...
        // 200 dong code o day
    }
}

// SUA: Tach responsibility ro rang
public class OrderController : ControllerBase
{
    private readonly ISender _mediator;
    public async Task<IActionResult> Create(CreateOrderCommand cmd)
        => Ok(await _mediator.Send(cmd));
}
```

**Open/Closed** thuong bi vi pham khi dung `if/else` hoac `switch` de xu ly nhieu loai nghiep vu -- giai phap la dung Strategy pattern ket hop DI. **Interface Segregation** bi vi pham khi mot interface co 15-20 method ma cac class implement chi dung 2-3 cai, buoc phai throw `NotImplementedException`.

Bay thuc te: dung ap dung SOLID mot cach cuong tin. Neu mot service chi co 30 dong code va khong co kha nang mo rong, viec tach ra 5 interface/class chi tao them complexity vo nghia. SOLID la huong dan, khong phai luat.

---

### Q10. Repository + Unit of Work pattern co con can thiet khi da dung EF Core khong? Khi nao nen dung, khi nao la thua?

**Answer:**

Day la cau hoi gay tranh cai nhieu nhat trong cong dong .NET. EF Core **da la** mot implementation cua Repository (DbSet) + Unit of Work (DbContext). Viec wrap them mot lop Generic Repository len tren thuong chi tao ra **leaky abstraction** va giau di suc manh cua EF Core (nhu `Include`, `AsNoTracking`, projection).

**Khi KHONG nen dung:** Du an CRUD don gian, team nho, chi dung 1 loai database. Generic Repository kieu `IRepository<T>` voi `GetAll()`, `GetById()` chi tao them mot lop trung gian vo ich.

**Khi NEN dung:** Khi ban thuc su can abstract data access layer -- vi du phai ho tro nhieu loai DB (SQL Server + MongoDB), hoac muon tach biet domain layer hoan toan khoi infrastructure trong Clean Architecture.

```csharp
// THAY VI Generic Repository, dung Specific Repository / Query Object
public interface IOrderRepository
{
    Task<Order?> GetPendingOrderWithItems(Guid id);  // ro intent
    Task<PagedResult<OrderSummary>> SearchOrders(OrderFilter filter);
}

// Hoac inject DbContext truc tiep vao handler (Vertical Slice)
public class GetOrderHandler : IRequestHandler<GetOrderQuery, OrderDto>
{
    private readonly AppDbContext _db;
    public async Task<OrderDto> Handle(GetOrderQuery q, CancellationToken ct)
        => await _db.Orders.Where(o => o.Id == q.Id)
            .Select(o => new OrderDto { ... }).FirstAsync(ct);
}
```

Trade-off: inject thang DbContext thi don gian nhung kho unit test (phai dung in-memory DB hoac Testcontainers). Repository thi de mock nhung tao them abstraction layer. Loi khuyen thuc te: **bat dau khong co Repository**, chi them khi co nhu cau thuc su.

---

### Q11. CQRS + MediatR mang lai loi ich gi? Pipeline behaviors hoat dong ra sao? Khi nao dung la overkill?

**Answer:**

CQRS tach model doc va ghi, giup optimize tung phia doc lap. MediatR la thu vien trien khai mediator pattern, **khong phai CQRS** -- nhung thuong dung chung. Loi ich chinh: decoupling handler khoi controller, va dac biet la **pipeline behaviors** -- cross-cutting concerns dang middleware.

```csharp
// Pipeline behavior: tu dong validate moi command/query
public class ValidationBehavior<TReq, TRes> : IPipelineBehavior<TReq, TRes>
{
    private readonly IEnumerable<IValidator<TReq>> _validators;
    public async Task<TRes> Handle(TReq req, RequestHandlerDelegate<TRes> next,
        CancellationToken ct)
    {
        var failures = _validators.SelectMany(v => v.Validate(req).Errors);
        if (failures.Any()) throw new ValidationException(failures);
        return await next();  // goi handler tiep theo trong pipeline
    }
}
```

Pipeline behaviors pho bien: **Validation**, **Logging**, **Caching** (cho query), **Transaction** (wrap command trong `BeginTransaction/Commit`), **Performance monitoring**.

**Khi nao overkill:** Du an CRUD don gian, it business logic. Neu handler chi la 3 dong goi DbContext roi return, ban dang tao them complexity khong can thiet. Ngoai ra MediatR tao **indirection** -- khi debug phai nhay qua nhieu lop, dev moi vao team se kho follow flow. Can nhac dung khi co 10+ use cases phuc tap voi cross-cutting concerns ro rang, khong dung cho microservice chi co 3-4 endpoint.

---

### Q12. Giai thich cach ap dung Strategy, Factory va Decorator pattern trong .NET Core DI container?

**Answer:**

Ba pattern nay ket hop voi DI container cua .NET Core rat tu nhien, giup tranh `if/else` chains va tuan thu Open/Closed principle.

**Strategy** -- chon algorithm tai runtime:
```csharp
// Dang ky nhieu strategy (.NET 8 Keyed Services)
services.AddKeyedScoped<IPaymentProcessor, VnPayProcessor>("vnpay");
services.AddKeyedScoped<IPaymentProcessor, MomoProcessor>("momo");

// Resolve bang key
public class PaymentService(IServiceProvider sp)
{
    public Task Pay(string method, decimal amount)
    {
        var processor = sp.GetRequiredKeyedService<IPaymentProcessor>(method);
        return processor.ProcessAsync(amount);
    }
}
```

**Decorator** -- wrap behavior len service co san. Thu vien Scrutor ho tro rat tot:
```csharp
services.AddScoped<IOrderService, OrderService>();
services.Decorate<IOrderService, CachedOrderService>();   // them caching
services.Decorate<IOrderService, LoggingOrderService>();  // them logging
// Chain: request di qua Logging -> Caching -> OrderService thuc
```

**Factory** -- khi can tao instance dua tren runtime data ma DI khong resolve duoc truc tiep, dung `Func<T>` hoac factory class dang ky trong DI.

Pitfall: dung lam dung decorator qua sau (4-5 lop), debug se rat kho trace.

---

### Q13. So sanh Clean Architecture va Vertical Slice Architecture -- khi nao chon cai nao?

**Answer:**

**Clean Architecture** to chuc code theo layer nam ngang: Domain -> Application -> Infrastructure -> Presentation. Moi layer la mot project rieng, dependency huong vao trong.

**Vertical Slice Architecture** to chuc code theo feature doc: moi feature (vi du `CreateOrder`) chua tat ca tu request/response model, handler, validation, DB query trong cung mot folder.

| Tieu chi | Clean Architecture | Vertical Slice |
|---|---|---|
| Cau truc folder | Theo layer (`/Domain`, `/Application`) | Theo feature (`/Features/Orders/Create`) |
| Coupling | Loose giua layers, shared abstractions | Loose giua features, tight trong feature |
| Them feature moi | Sua nhieu layer/project | Them 1 folder moi, khong anh huong feature khac |
| Learning curve | Cao, nhieu abstraction | Thap hon, de hieu tung feature doc lap |
| Phu hop | Domain phuc tap, team lon, can enforce rule | Feature-driven, team nho-trung, can toc do |

Trade-off quan trong: Clean Architecture de bi **over-engineering** voi qua nhieu interface va mapping. Vertical Slice de bi **code duplication** giua cac feature.

Loi khuyen thuc te: bat dau voi Vertical Slice cho MVP, refactor sang Clean Architecture khi domain du phuc tap va team du lon. Nhieu team thanh cong ket hop ca hai: Clean Architecture o muc project structure, Vertical Slice o muc feature organization trong Application layer.

---

### Q14. Giai thich Aggregate, Value Object va Domain Events trong DDD. Cho vi du thuc te?

**Answer:**

**Aggregate** la cluster cac entity co chung business invariant, truy cap qua mot Aggregate Root duy nhat. **Value Object** la object khong co identity, so sanh bang gia tri, immutable. **Domain Events** thong bao "dieu gi do da xay ra" trong domain, giup decouple giua cac aggregate.

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
        // Handler khac se: gui email, cap nhat inventory, tao invoice...
    }
}
```

Pitfall thuc te: aggregate qua lon gay lock contention trong DB. Nguyen tac la giu aggregate nho, dung domain events de dong bo giua cac aggregate thay vi nhet tat ca vao mot aggregate root. Mot sai lam pho bien: dung Entity thay cho Value Object cho `Address`, `Money`, `DateRange`.

---

### Q15. Khi nao dung Dictionary, HashSet, List trong .NET? Phan tich Big-O trong cac tinh huong thuc te?

**Answer:**

Nguyen tac chon dua tren **operation pho bien nhat** trong use case:

| Collection | Lookup | Insert | Dung khi |
|---|---|---|---|
| `List<T>` | O(n) | O(1) amortized | Du lieu it, can index, iterate nhieu |
| `Dictionary<K,V>` | O(1) avg | O(1) avg | Can map key->value, lookup nhieu |
| `HashSet<T>` | O(1) avg | O(1) avg | Can check ton tai, loai bo trung lap |

```csharp
// SAI: dung List roi .Any() trong vong lap = O(n*m)
var existingIds = await _db.Products.Select(p => p.Id).ToListAsync();
foreach (var item in importData)
    if (existingIds.Any(id => id == item.Id)) { ... }  // O(n) moi lan

// DUNG: dung HashSet = O(n+m)
var existingIds = (await _db.Products.Select(p => p.Id).ToListAsync()).ToHashSet();
foreach (var item in importData)
    if (existingIds.Contains(item.Id)) { ... }  // O(1) moi lan
```

Luu y: voi du lieu nho (duoi ~100 phan tu), `List` co the nhanh hon `Dictionary`/`HashSet` nho cache locality. Voi .NET 6+, `FrozenDictionary` / `FrozenSet` cho read-only scenarios nhanh hon dang ke vi optimize layout tai thoi diem tao.

---

### Q16. So sanh offset-based pagination va keyset (cursor) pagination? Khi nao sorting nen thuc hien trong DB vs in-memory?

**Answer:**

**Offset pagination** (`OFFSET/FETCH` hoac `.Skip().Take()`): don gian, cho phep nhay den page bat ky. Nhuoc diem: page cang lon, DB phai scan va bo qua cang nhieu row.

**Keyset pagination** (cursor): dung gia tri cua row cuoi cung lam moc:

```csharp
// Offset: cham dan o page lon
var page = await _db.Orders.OrderByDescending(o => o.CreatedAt)
    .Skip((pageNum - 1) * pageSize).Take(pageSize).ToListAsync();

// Keyset: toc do on dinh bat ke vi tri
var page = await _db.Orders
    .Where(o => o.CreatedAt < lastSeenDate
        || (o.CreatedAt == lastSeenDate && o.Id < lastSeenId))
    .OrderByDescending(o => o.CreatedAt).ThenByDescending(o => o.Id)
    .Take(pageSize).ToListAsync();
```

| Tieu chi | Offset | Keyset |
|---|---|---|
| Performance page lon | Kem (O(offset+limit)) | On dinh (O(limit)) |
| Nhay page ngau nhien | Duoc | Khong (chi next/prev) |
| Du lieu thay doi lien tuc | Bi trung/mat row | On dinh, khong bi lech |
| Phu hop | Admin panel, report it data | Feed, infinite scroll, API lon |

**Sorting:** luon uu tien sort trong DB vi DB da co index. Sort in-memory chi hop ly khi: (1) du lieu da load het vao memory roi (cache), (2) sort logic phuc tap phu thuoc business rule khong the express bang SQL, (3) du lieu tu nhieu nguon can merge roi sort. Pitfall: `.ToList()` roi `.OrderBy()` tren 100K record la cach pho bien nhat de giet performance.

---

## SECTION 3: RESTful APIs, JSON & Asynchronous Programming (Q17-Q24)

---

### Q17. What are the core RESTful API design principles and how do you implement them properly in ASP.NET Core?

**Answer:**

RESTful API xoay quanh cac nguyen tac: su dung HTTP verbs dung ngu nghia (GET khong thay doi state, PUT la idempotent, POST tao moi), resource-based URI (danh tu so nhieu, khong dung dong tu), stateless communication.

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

Pitfall pho bien: tra 200 cho moi thu thay vi dung dung status code (201 Created, 204 No Content, 409 Conflict). Nen dung `[ApiController]` attribute vi no tu dong enable model validation, binding source inference, va `ProblemDetails` response cho 400+. Trong thuc te, nen thiet ke API theo business capability chu khong phai map 1:1 voi database table.

---

### Q18. Explain API versioning strategies in ASP.NET Core. Which approach do you prefer and why?

**Answer:**

Co 4 chien luoc chinh: URL path (`/api/v2/orders`), query string (`?api-version=2`), header (`X-Api-Version`), va media type versioning.

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

URL path versioning la pho bien nhat vi de hieu, de debug, de cache. Trade-off la URL bi "xau" hon va vi pham nguyen tac URI nen identify resource chu khong phai version.

Chien luoc deprecation quan trong khong kem: dung `[ApiVersion("1.0", Deprecated = true)]` ket hop voi `Sunset` header de bao client biet timeline. Nen maintain toi da 2 version dong thoi.

---

### Q19. Compare System.Text.Json vs Newtonsoft.Json. When would you still choose Newtonsoft in a .NET 8 project?

**Answer:**

`System.Text.Json` (STJ) la default tu .NET Core 3.0, duoc thiet ke cho performance voi allocation thap hon 2-5x so voi Newtonsoft nho `Utf8JsonReader/Writer` lam viec truc tiep tren byte UTF-8. Tu .NET 8, STJ da support source generator tot hon giup trim-friendly cho AOT.

```csharp
// Source generator - zero reflection, AOT-compatible
[JsonSerializable(typeof(List<OrderDto>))]
[JsonSourceGenerationOptions(PropertyNamingPolicy = JsonKnownNamingPolicy.CamelCase)]
public partial class AppJsonContext : JsonSerializerContext { }

builder.Services.ConfigureHttpJsonOptions(opt =>
    opt.SerializerOptions.TypeInfoResolverChain.Add(AppJsonContext.Default));
```

Chon Newtonsoft khi: can `$ref`/`$id` handling cho circular references phuc tap, can `JsonPath` query, project dang dung nhieu custom `JsonConverter` Newtonsoft ma migrate ton effort lon, hoac can serialize dynamic/ExpandoObject phuc tap.

Pitfall: STJ mac dinh case-sensitive va khong serialize field (chi property). Khi chuyen tu Newtonsoft sang, day la hai loi pho bien nhat gay mat data silent.

---

### Q20. Explain how async/await works internally — state machine, SynchronizationContext, and when to use ValueTask.

**Answer:**

Khi compiler gap `async`, no generate mot struct implement `IAsyncStateMachine` voi `MoveNext()` method. Moi `await` la mot "checkpoint" -- neu Task chua complete, state machine luu current state, dang ky continuation, roi return. Khi Task complete, continuation duoc schedule qua `SynchronizationContext` (neu co) hoac `ThreadPool`.

`SynchronizationContext` trong ASP.NET Core la **null** (khong co nhu ASP.NET classic), nen `ConfigureAwait(false)` khong can thiet trong application code -- chi can trong library code de tranh deadlock khi consumer co SynchronizationContext.

`ValueTask<T>` dung khi: method thuong xuyen return synchronously (cache hit, buffer da co data). No tranh allocation Task object. Nhung **khong duoc** await `ValueTask` nhieu lan, khong `WhenAll` duoc, va khong duoc cache no. Tu .NET 8, nen dung `ValueTask` cho hot path performance-critical. Neu khong chac -- dung `Task<T>` cho an toan vi no it rang buoc hon.

---

### Q21. How do you implement CancellationToken propagation and handle graceful shutdown in ASP.NET Core?

**Answer:**

`CancellationToken` trong ASP.NET Core tu dong trigger khi client disconnect (request aborted). Nguyen tac la propagate token qua **toan bo call chain** -- tu controller xuong service, repository, den DbContext va HttpClient.

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

// Graceful shutdown voi IHostApplicationLifetime
public class CleanupService : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
            await ProcessQueueAsync(stoppingToken);
    }
}
```

Trong `Program.cs`, configure `HostOptions.ShutdownTimeout` (default 30s) de background service co du thoi gian drain. Pitfall pho bien: quen truyen token vao `HttpClient.SendAsync()` hoac `Task.Delay()`, dan den request da bi cancel nhung server van xu ly ton resource. Voi long-running operation, nen dung `CancellationTokenSource.CreateLinkedTokenSource()` de combine request cancellation voi custom timeout.

---

### Q22. How does rate limiting work in .NET 7+ and what strategies are available out of the box?

**Answer:**

.NET 7 gioi thieu built-in rate limiting middleware voi 4 algorithm: Fixed Window, Sliding Window, Token Bucket, va Concurrency Limiter.

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

Fixed Window don gian nhung co "burst at boundary" problem. Sliding Window giai quyet van de nay. Token Bucket phu hop API cho phep burst ngan. Concurrency Limiter gioi han so request dong thoi (huu ich cho heavy endpoint).

Trong production distributed, can Redis-backed rate limiting (thu vien ben thu ba) vi built-in chi hoat dong per-instance. Luon tra `Retry-After` header trong 429 response.

---

### Q23. How do you approach API documentation with OpenAPI/Swagger and contract-first design?

**Answer:**

Tu .NET 9, ASP.NET Core tich hop `Microsoft.AspNetCore.OpenApi` thay the dan Swashbuckle. Contract-first nghia la viet OpenAPI spec truoc roi generate code -- giup frontend va backend phat trien song song.

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

Dung `[ProducesResponseType]`, `[EndpointSummary]`, `[EndpointDescription]` (Minimal API .NET 7+) de document ro rang. Best practice: tich hop OpenAPI spec validation vao CI pipeline -- neu spec thay doi breaking (xoa field, doi type) thi fail build. Dung tool nhu `oasdiff` de detect breaking changes tu dong. Dung expose Swagger UI o production -- chi bat o development environment.

---

### Q24. Explain the ASP.NET Core middleware pipeline. How do you handle cross-cutting concerns efficiently?

**Answer:**

Middleware pipeline la chuoi delegate xu ly request theo thu tu FIFO va response theo thu tu LIFO (Russian doll model). Moi middleware goi `next()` de chuyen tiep hoac short-circuit pipeline.

```csharp
// Thu tu middleware CUC KY quan trong
app.UseExceptionHandler("/error");  // 1. Bat exception tu tat ca middleware phia sau
app.UseHsts();
app.UseHttpsRedirection();
app.UseCors("policy");              // 2. Truoc auth
app.UseAuthentication();            // 3. Xac thuc
app.UseAuthorization();             // 4. Phan quyen
app.UseRateLimiter();               // 5. Sau auth de rate limit per-user
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

Pitfall kinh dien: dat CORS sau Authorization -- browser preflight request se bi 401 truoc khi CORS header duoc them, gay loi kho debug. Dung `IMiddleware` (transient) khi can inject scoped service, dung convention-based middleware (singleton implicitly) cho stateless logic vi performance tot hon.

---

## SECTION 4: Relational Databases & ORM Frameworks (Q25-Q32)

---

### Q25. Compare EF Core vs Dapper. When do you use each, and can they coexist?

**Answer:**

EF Core la full ORM voi change tracking, migration, LINQ provider -- phu hop CRUD-heavy domain. Dapper la micro-ORM, chi map query result sang object -- nhanh hon 5-10x cho read-heavy, reporting.

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

    // Dapper cho complex read - raw SQL, toi uu performance
    public async Task<IEnumerable<OrderSummaryDto>> GetSummaryAsync(int year)
    {
        return await dapper.QueryAsync<OrderSummaryDto>(
            "SELECT o.Id, o.Total, c.Name FROM Orders o " +
            "JOIN Customers c ON o.CustomerId = c.Id " +
            "WHERE YEAR(o.CreatedAt) = @Year", new { Year = year });
    }
}
```

Nguyen tac: EF Core cho write path (can transaction, change tracking), Dapper cho read path phuc tap (report, dashboard). Co the share cung connection: `efContext.Database.GetDbConnection()`. Pitfall: khong mix EF Core tracking voi Dapper update tren cung entity trong cung request.

---

### Q26. Explain EF Core performance optimizations: AsNoTracking, Compiled Queries, and Split Queries.

**Answer:**

`AsNoTracking()` tat change tracker -- tiet kiem memory va CPU dang ke (nhanh hon 30-50%). `Compiled Queries` compile expression tree 1 lan, tai su dung. `Split Query` tranh cartesian explosion.

```csharp
// Compiled Query
private static readonly Func<AppDbContext, int, Task<Order?>> _getOrder =
    EF.CompileAsyncQuery((AppDbContext db, int id) =>
        db.Orders.AsNoTracking()
          .Include(o => o.Items)
          .FirstOrDefault(o => o.Id == id));

// Split Query - tranh cartesian explosion
var orders = await db.Orders
    .Include(o => o.Items)       // 1000 orders x 50 items = 50k rows neu single query
    .Include(o => o.Payments)
    .AsSplitQuery()              // 3 query rieng, moi query nho gon
    .ToListAsync(ct);
```

Trade-off: `AsSplitQuery()` giam data transfer nhung tang so roundtrip -- neu latency den DB cao thi single query co the nhanh hon. Compiled Query huu ich nhat cho hot path duoc goi hang ngan lan/giay. Nen set `AsNoTracking` o DbContext level cho read-only context.

---

### Q27. How do you design an effective indexing strategy — composite, covering, and filtered indexes?

**Answer:**

Index strategy phai dua tren actual query patterns, khong phai doan. Composite index thu tu column cuc ky quan trong -- tuan theo quy tac "equality first, range last, high selectivity truoc".

```sql
-- Composite + Covering: tranh key lookup
CREATE INDEX IX_Orders_Status_CreatedAt
ON Orders (Status, CreatedAt DESC)
INCLUDE (Total, CustomerId);

-- Filtered: chi index subset data, nho hon va nhanh hon
CREATE INDEX IX_Orders_Active
ON Orders (CreatedAt DESC)
WHERE Status = 'Active' AND IsDeleted = 0;
-- Chi huu ich khi filter match < 30% total rows
```

Covering index chua tat ca column can thiet cho query, tranh key lookup ve clustered index -- co the cai thien performance 10x cho analytical query. Filtered index nho hon, maintain nhanh hon, nhung chi duoc dung khi query co WHERE clause match chinh xac filter condition.

Pitfall: over-indexing lam cham INSERT/UPDATE. Trong EF Core migration, dung `HasIndex().HasFilter()` va `HasIndex().IncludeProperties()` (.NET 7+). Luon kiem tra index usage thuc te bang `sys.dm_db_index_usage_stats` -- xoa index khong duoc dung.

---

### Q28. How do you detect and resolve the N+1 problem in EF Core?

**Answer:**

N+1 xay ra khi load parent entity roi lazy load child entity trong loop -- 1 query cho parent + N query cho moi child. Day la "silent performance killer" pho bien nhat.

```csharp
// N+1 Problem - 1 + N queries
var orders = await db.Orders.ToListAsync();
foreach (var order in orders)
    Console.WriteLine(order.Customer.Name); // Lazy load moi iteration!

// Fix 1: Eager loading
var orders = await db.Orders.Include(o => o.Customer).ToListAsync(ct);

// Fix 2: Projection - tot nhat cho performance
var orders = await db.Orders
    .Select(o => new OrderDto
    {
        Id = o.Id,
        CustomerName = o.Customer.Name,
        ItemCount = o.Items.Count
    }).ToListAsync(ct);
```

Best practice: **tat lazy loading hoan toan** bang cach khong install proxy package. Projection (Select) la giai phap tot nhat vi chi query dung column can thiet. Trong production, nen co automated test dem so query cho critical endpoint.

---

### Q29. What strategies do you use for zero-downtime database migrations in production?

**Answer:**

Zero-downtime migration yeu cau backward-compatible changes -- code cu va code moi phai chay dong thoi tren cung schema (expand-contract pattern).

```
// Expand-Contract: doi ten column "Name" -> "FullName"
// Phase 1 (Expand): Them column moi, deploy code doc/ghi ca hai
ALTER TABLE Customers ADD FullName NVARCHAR(200);
UPDATE Customers SET FullName = Name;
-- Deploy code V2: write ca Name va FullName, read tu FullName

// Phase 2 (Contract): Sau khi 100% traffic dung V2
ALTER TABLE Customers DROP COLUMN Name;
```

Nguyen tac vang: **khong bao gio** lam trong mot migration: rename column, change type, drop column, add NOT NULL column without default. Moi thao tac phai tach thanh 2-3 migration rieng biet.

Khong dung `dotnet ef database update` o production. Export SQL script bang `dotnet ef migrations script --idempotent`, review ky, roi chay qua CI/CD pipeline. Voi database lon, them column hoac index phai dung `ONLINE = ON` (SQL Server) de tranh lock table. Luon co rollback script cho moi migration.

---

### Q30. Explain transaction isolation levels and how you handle concurrency conflicts in EF Core.

**Answer:**

SQL Server co 5 isolation level: Read Uncommitted, Read Committed (default), Repeatable Read, Serializable, va Snapshot. Moi level trade-off giua consistency va concurrency/performance.

```csharp
// Optimistic concurrency voi EF Core - pho bien nhat cho web app
public class Order
{
    public int Id { get; set; }

    [Timestamp] // SQL Server rowversion - tu dong
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
    await db.SaveChangesAsync(ct); // retry voi gia tri moi
}
```

Web application 99% dung Optimistic Concurrency (`[Timestamp]`) vi request ngan, conflict hiem. Pessimistic locking (SELECT FOR UPDATE) chi dung cho critical business logic nhu inventory deduction. Snapshot Isolation tot cho reporting nhung ton them tempdb space. Pitfall: Serializable gay deadlock rat de trong high-concurrency.

---

### Q31. How do you approach query optimization — execution plans, parameter sniffing, and EF Core query analysis?

**Answer:**

Query optimization bat dau tu execution plan -- xem actual plan (khong phai estimated) de biet query thuc su chay the nao. Tim cac operator ton kem: Table Scan, Key Lookup, Sort (spill to tempdb).

```sql
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

-- Parameter sniffing problem:
-- SP compiled voi @Status = 'Active' (90% rows) -> table scan plan
-- Khi goi voi @Status = 'VIP' (0.1% rows) -> van dung plan cu!
SELECT * FROM Orders WHERE Status = @Status
OPTION (OPTIMIZE FOR (@Status UNKNOWN));
```

Trong EF Core, dung `ToQueryString()` de xem generated SQL truoc khi chay. Dung `Microsoft.EntityFrameworkCore.Diagnostics` de log slow query.

Checklist: (1) Kiem tra logical reads trong STATISTICS IO, (2) Tranh SELECT * -- dung projection, (3) Kiem tra implicit conversion (nvarchar vs varchar gay index scan thay vi seek), (4) EF Core `HasConversion()` co the gay implicit conversion SQL -- loi an rat kho phat hien.

---

### Q32. Explain connection pooling, DbContext lifetime management, and scaling strategies for database access.

**Answer:**

ADO.NET connection pooling mac dinh bat -- pool size mac dinh 100 per connection string. DbContext trong ASP.NET Core nen register Scoped (1 instance per request) -- day la default cua `AddDbContext`.

```csharp
// DbContext pooling - tai su dung instance, giam allocation
builder.Services.AddDbContextPool<AppDbContext>(opt =>
    opt.UseSqlServer(connStr, sql =>
    {
        sql.EnableRetryOnFailure(3, TimeSpan.FromSeconds(5), null);
        sql.CommandTimeout(30);
    }), poolSize: 1024);
// Luu y: Pooled DbContext KHONG duoc inject scoped service vao constructor
```

`AddDbContextPool` (.NET 6+) giu pool DbContext instance, reset state roi tai su dung -- giam GC pressure dang ke o high-throughput. Trade-off: khong inject service vao DbContext constructor.

Scaling: Read replica cho read-heavy workload, sharding cho data lon, CQRS tach read/write model hoan toan. Pitfall: connection pool exhaustion khi quen `await` async method hoac DbContext song qua lau (Singleton DbContext la anti-pattern nghiem trong).

---

## SECTION 5: Security Principles, Data Protection & Compliance (Q33-Q40)

---

### Q33. Explain JWT, OAuth2, and OpenID Connect authentication flow in ASP.NET Core. What are common pitfalls?

**Answer:**

OAuth2 la authorization framework (cap quyen truy cap resource), OpenID Connect (OIDC) la identity layer tren OAuth2 (xac thuc user). JWT la token format pho bien. Flow pho bien nhat cho web app: Authorization Code + PKCE.

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
            ClockSkew = TimeSpan.FromSeconds(30), // Giam tu default 5 phut
        };
    });
```

Pitfalls nghiem trong: (1) Khong validate `aud` claim -- token cua API khac co the dung cho API ban, (2) ClockSkew default 5 phut qua rong, (3) Luu sensitive data trong JWT payload -- no chi Base64 encode, khong encrypt, (4) Dung symmetric key (HMAC) cho distributed system -- phai share secret, nen dung RSA/ECDSA.

Access token nen short-lived (5-15 phut), refresh token long-lived + rotation. Khong luu token trong localStorage (XSS risk) -- dung HttpOnly Secure cookie hoac BFF pattern.

---

### Q34. Compare RBAC, Policy-based, and Resource-based authorization in ASP.NET Core. When do you use each?

**Answer:**

RBAC (Role-Based) don gian nhung cung nhac. Policy-based linh hoat hon -- combine nhieu requirement. Resource-based kiem tra quyen dua tren data cu the (vi du: chi owner moi edit duoc order).

```csharp
// Policy-based: combine nhieu dieu kien
builder.Services.AddAuthorizationBuilder()
    .AddPolicy("SeniorManager", policy =>
        policy.RequireRole("Manager")
              .RequireClaim("experience_years")
              .AddRequirements(new MinExperienceRequirement(5)));

// Resource-based: kiem tra ownership
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

Thuc te: dung RBAC cho coarse-grained access (admin vs user), Policy-based cho business rules phuc tap, Resource-based cho row-level security. Dung hardcode role string khap noi -- tap trung vao policy name. Pitfall: `[Authorize(Roles = "Admin")]` scattered khap codebase rat kho maintain.

---

### Q35. How do you prevent OWASP Top 10 vulnerabilities in a .NET Core application?

**Answer:**

OWASP Top 10 (2021): Broken Access Control, Cryptographic Failures, Injection, Insecure Design, Security Misconfiguration, Vulnerable Components, Authentication Failures, Data Integrity Failures, Logging Failures, SSRF.

```csharp
// 1. SQL Injection - NGUY HIEM vs AN TOAN
// NGUY HIEM: FromSqlRaw($"SELECT * FROM Users WHERE Name = '{input}'")
// AN TOAN:
db.Users.FromSqlInterpolated($"SELECT * FROM Users WHERE Name = {input}");

// 2. Mass Assignment / Over-posting
[HttpPost]
public IActionResult Create([Bind("Name,Email")] UserDto dto)
// Hoac tot hon: dung DTO rieng cho input, khong bind truc tiep Entity

// 3. Security headers
builder.Services.AddHsts(opt => { opt.MaxAge = TimeSpan.FromDays(365); });
```

Prevention checklist: (1) `[ValidateAntiForgeryToken]` hoac global filter, (2) Content-Security-Policy header, (3) Enable HSTS, (4) Parameterized query 100%, (5) `dotnet list package --vulnerable` thuong xuyen, (6) FluentValidation cho moi input, (7) Khong expose stack trace o production -- dung `ProblemDetails`. Nguyen tac defense-in-depth: khong bao gio tin tuong chi mot layer bao ve.

---

### Q36. Explain data encryption strategies: at rest, in transit, and column-level encryption in .NET Core.

**Answer:**

In transit: TLS 1.2+ bat buoc. At rest: SQL Server TDE encrypt toan bo database file. Column-level: encrypt cac column nhay cam rieng biet.

```csharp
// Column-level encryption voi Data Protection API
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
    .HasMaxLength(500); // Encrypted data lon hon plaintext
```

Trade-off column-level: khong the query/sort/filter tren encrypted column (tru Always Encrypted deterministic mode cho equality check). Data Protection API tu dong rotate key nhung can configure persistent key storage (Azure Blob, Redis) -- neu khong, restart app = mat key = mat data.

Pitfall: TDE khong bao ve data trong memory hay query result -- DBA van doc duoc. Always Encrypted bao ve ca khoi DBA nhung han che query capability.

---

### Q37. How do you manage secrets in .NET Core across different environments?

**Answer:**

Hierarchy uu tien (cao den thap): Environment Variables > User Secrets (dev) > appsettings.{Env}.json > appsettings.json. Trong production, dung Azure Key Vault / AWS Secrets Manager -- KHONG BAO GIO commit secrets vao source control.

```csharp
// Production: Azure Key Vault
builder.Configuration.AddAzureKeyVault(
    new Uri("https://myvault.vault.azure.net/"),
    new DefaultAzureCredential()); // Managed Identity - no secrets needed!

// Binding vao strongly-typed options
builder.Services.AddOptions<DatabaseOptions>()
    .BindConfiguration("Database")
    .ValidateDataAnnotations()
    .ValidateOnStart(); // Fail fast neu config invalid

public class DatabaseOptions
{
    [Required] public string ConnectionString { get; set; } = null!;
    [Range(1, 200)] public int MaxPoolSize { get; set; } = 100;
}
```

Pitfalls: (1) Log configuration values ra console/file -- vo tinh expose secrets, (2) Dung appsettings.json cho connection string production, (3) Share Key Vault giua environments, (4) Quen `ValidateOnStart()` -- app chay nhung crash khi can config lan dau. Them `.gitignore` rule va scan repo bang `gitleaks` trong CI.

---

### Q38. How do you configure CORS, CSP, and security headers properly in ASP.NET Core?

**Answer:**

CORS kiem soat domain nao duoc goi API. CSP kiem soat resource nao browser duoc load. Security headers bao ve chong clickjacking, MIME sniffing, etc.

```csharp
// CORS - cu the, KHONG dung AllowAnyOrigin + AllowCredentials cung luc
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

Pitfalls: (1) `AllowAnyOrigin()` trong production, (2) Thieu `frame-ancestors 'none'` trong CSP -- de bi clickjacking, (3) CORS chi la browser enforcement -- server-to-server request bypass hoan toan, khong thay the authentication. Nen dung library `NWebsec` hoac `NetEscapades.AspNetCore.SecurityHeaders`.

---

### Q39. How do you implement robust input validation and prevent anti-tampering in .NET Core APIs?

**Answer:**

Validation phai thuc hien o nhieu layer: client-side (UX), API input validation (security), domain validation (business rules), database constraints (last line of defense). Khong bao gio tin client input.

```csharp
// FluentValidation - khai bao ro rang, testable
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

// Anti-tampering: KHONG tin client-sent price, re-calculate server-side
public async Task<Order> CreateOrder(CreateOrderRequest req)
{
    var product = await _db.Products.FindAsync(req.ProductId);
    var total = product.Price * req.Quantity; // Server-side price, khong dung req.Price
}
```

Anti-tampering patterns: khong trust hidden fields (price, userId), dung server-side state cho moi thu critical, implement idempotency key de prevent duplicate submission, validate file upload (check magic bytes, khong chi extension). Pitfall: chi validate o client -- attacker bypass JavaScript de dang bang Postman/curl.

---

### Q40. How do you implement audit logging and ensure compliance with GDPR/PCI-DSS basics in .NET Core?

**Answer:**

Audit logging ghi lai WHO did WHAT to WHICH resource WHEN. GDPR yeu cau right to access, right to erasure, data portability. PCI-DSS yeu cau encrypt cardholder data, restrict access, audit trail.

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

PCI-DSS: khong bao gio luu CVV, mask card number (chi hien thi 4 so cuoi), dung tokenization (Stripe, Braintree) thay vi tu xu ly card data -- giam PCI scope dang ke.

Audit log phai immutable (append-only table, khong UPDATE/DELETE), luu rieng biet database/service, va retain theo compliance requirement (PCI: 1 nam online, 7 nam archive). Pitfall: log PII vao audit trail roi khong the xoa khi user request GDPR erasure -- design audit log de separate PII tu dau.

---

## SECTION 6: Problem-Solving, Debugging & Troubleshooting (Q41-Q48)

---

### Q41. Trinh bay phuong phap co he thong de debug mot API endpoint cham (response > 5 giay)?

**Answer:**

Tiep can theo **top-down, do truoc khi doan**. Khong bao gio doan nguyen nhan roi fix -- phai co data chung minh.

**Buoc 1 - Reproduce & Measure:** Xac nhan van de. Kiem tra co phai cham consistent hay chi intermittent.

**Buoc 2 - Xac dinh layer nao cham:**
```csharp
app.Use(async (context, next) =>
{
    var sw = Stopwatch.StartNew();
    await next();
    Log.Information("Request {Method} {Path} took {Elapsed}ms",
        context.Request.Method, context.Request.Path, sw.ElapsedMilliseconds);
});
```

**Buoc 3 - Thu hep pham vi:**
- **Database cham?** Bat EF Core logging, kiem tra query plan, tim N+1, missing index.
- **External service cham?** Kiem tra timeout, them circuit breaker, can nhac cache.
- **Business logic nang?** Profile CPU, kiem tra vong lap O(n^2), large object allocation.
- **Network/infra?** Kiem tra DNS resolution, connection pool exhaustion, container resource limits.

**Buoc 4 - Fix & Verify:** Ap dung fix, do lai voi cung dieu kien, so sanh before/after.

Kinh nghiem thuc te: 80% API cham la do database -- N+1 query, missing index, hoac load qua nhieu data khong can thiet (SELECT * thay vi projection). Luon kiem tra DB truoc.

---

### Q42. Lam the nao de phat hien va xu ly memory leak trong .NET Core? Dung tool gi?

**Answer:**

Memory leak trong .NET thuong do: **event handler khong unsubscribe**, **static collection tich luy**, **IDisposable khong dispose** (HttpClient, DbContext), hoac **closure capture** giu reference ngoai y muon.

**Phat hien bang dotnet-counters (khong can restart app):**
```bash
dotnet-counters monitor --process-id <PID> \
    --counters System.Runtime[gc-heap-size,gen-2-gc-count,gen-2-size]
```
Neu `gc-heap-size` tang lien tuc sau moi Gen 2 GC, rat co the co leak.

**Phan tich bang dotnet-dump:**
```bash
dotnet-dump collect --process-id <PID>
dotnet-dump analyze <dump-file>
> dumpheap -stat          # xem object nao nhieu nhat
> dumpheap -type MyClass  # tim instance cu the
> gcroot <address>        # tim ai dang giu reference
```

**Phong ngua:** Dung `IHttpClientFactory` thay vi `new HttpClient()` (tranh socket exhaustion). Dang ky DbContext la Scoped, khong phai Singleton. Pitfall: `MemoryCache` khong gioi han size mac dinh -- phai set `SizeLimit` hoac `AbsoluteExpiration`, neu khong cache se grow vo han.

---

### Q43. Trinh bay quy trinh xu ly incident production? OODA loop ap dung nhu the nao?

**Answer:**

**OODA loop** (Observe - Orient - Decide - Act) ap dung cho incident response:

**Observe:** Phat hien qua alert, user report, hoac health check fail. Xac dinh **severity**:
- **SEV1:** System down, mat doanh thu -> respond ngay, escalate.
- **SEV2:** Feature chinh bi loi -> respond trong 30 phut.
- **SEV3:** Feature phu bi loi, workaround co san -> respond trong gio lam viec.

**Orient:** Deploy gan day? Traffic spike? DB change? Dependency bi loi?

**Decide:** Chon giua rollback, hotfix, scale up, hoac toggle feature flag. Uu tien **khoi phuc service truoc, tim root cause sau**.

**Act:** Thuc hien mitigation. Mot nguoi fix, mot nguoi communication (cap nhat status page, thong bao stakeholder).

**Communication template:**
```
[SEV2] Order Service - Elevated Error Rate
Impact: 15% orders failing at checkout
Status: Investigating - suspected DB connection pool exhaustion
ETA: 30 minutes
Next update: 14:30
```

Nguyen tac vang: **MTTD + MTTR** quan trong hon MTBF. Invest vao monitoring va runbook hon la co gang prevent moi failure.

---

### Q44. Lam the nao phat hien N+1 query trong EF Core ma khong can profiler ben ngoai?

**Answer:**

N+1 xay ra khi load mot list entity roi lazy-load navigation property trong vong lap -- 1 query cho list + N query cho moi item.

**Cach 1 - Bat EF Core logging:**
```csharp
optionsBuilder
    .LogTo(Console.WriteLine, LogLevel.Information)
    .EnableSensitiveDataLogging()
    .EnableDetailedErrors();
```

**Cach 2 - EF Core Interceptor dem query per request:**
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
// Trong middleware: reset dau request, log cuoi request. Alert neu > 10 queries.
```

**Cach 3:** Dung `ConfigureWarnings` de throw exception khi lazy loading xay ra:
```csharp
optionsBuilder.ConfigureWarnings(w =>
    w.Throw(CoreEventId.NavigationLazyLoading));
```

**Fix:** `.Include()` / `.ThenInclude()` (eager loading), hoac tot hon la **projection** voi `.Select()` de chi lay dung field can.

---

### Q45. Giai thich cach chan doan deadlock trong SQL Server va trong async code .NET?

**Answer:**

Day la hai loai deadlock khac nhau nhung deu gay he thong treo.

**SQL Server Deadlock:** Hai transaction lock resource theo thu tu nguoc nhau. Chan doan: bat **Deadlock Graph** qua Extended Events, hoac query `sys.dm_exec_requests`. Fix: dam bao cac transaction access table/row **theo cung thu tu**, giam transaction scope, dung `SNAPSHOT ISOLATION` khi phu hop.

**Async Deadlock trong .NET:** Pho bien nhat la goi `.Result` hoac `.Wait()` tren async method trong synchronization context (ASP.NET Framework cu, WPF, WinForms):
```csharp
// DEADLOCK trong ASP.NET Framework (KHONG xay ra trong ASP.NET Core)
public IActionResult Get()
{
    var data = GetDataAsync().Result;  // block thread, giu sync context
    return Ok(data);
}

// FIX: async all the way
public async Task<IActionResult> Get()
{
    var data = await GetDataAsync();
    return Ok(data);
}
```

ASP.NET Core khong co `SynchronizationContext` nen it gap dang nay, nhung van co the deadlock khi dung `SemaphoreSlim` hoac custom sync primitives sai cach. Nguyen tac: **async all the way** -- khong bao gio mix sync va async code.

---

### Q46. Lam sao debug hieu qua trong he thong distributed? Correlation ID, OpenTelemetry va structured logging hoat dong ra sao?

**Answer:**

Trong distributed system, mot request di qua 5-10 service. Khong co correlation thi log chi la dong text vo nghia. Ba tru cot observability: **Logs**, **Metrics**, **Traces**.

**Correlation ID** -- gan mot ID duy nhat cho moi request, truyen qua moi service:
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

**Structured Logging** (Serilog) -- log dang key-value de co the query:
```csharp
// Thay vi: Log.Info($"Order {orderId} created by {userId}")
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
        .AddOtlpExporter());  // xuat sang Jaeger, Zipkin, hoac Grafana Tempo
```

Khi debug incident: tim correlation ID tu error log, query trace de thay toan bo call chain, xac dinh service/span nao cham hoac loi.

---

### Q47. Race condition la gi? Cho vi du trong .NET va cach xu ly?

**Answer:**

Race condition xay ra khi nhieu thread/request truy cap shared resource dong thoi va ket qua phu thuoc vao thu tu thuc thi. Day la loai bug nguy hiem vi **kho reproduce**.

**Vi du kinh dien -- double spending:**
```csharp
// Hai request dong thoi rut tien tu cung account
public async Task Withdraw(Guid accountId, decimal amount)
{
    var account = await _db.Accounts.FindAsync(accountId);
    if (account.Balance >= amount)   // ca 2 request deu thay balance = 1000
    {
        account.Balance -= amount;   // ca 2 deu tru 800 -> balance = 200
        await _db.SaveChangesAsync();// nhung dung ra phai fail request thu 2
    }
}
```

**Giai phap theo cap do:**

1. **Optimistic Concurrency (pho bien nhat):** EF Core `[Timestamp]` / `[ConcurrencyCheck]` -- SaveChanges throw `DbUpdateConcurrencyException` neu data da bi thay doi -> retry.

2. **Pessimistic Locking:** `SELECT ... WITH (UPDLOCK, ROWLOCK)` -- lock row trong DB. Dung khi conflict rate cao.

3. **Distributed Lock** (Redis): Khi xu ly cross-service, dung `RedLock` algorithm.

4. **Idempotency Key:** Cho moi operation mot unique key, check truoc khi execute -- dac biet quan trong cho payment.

Trade-off: Optimistic lock don gian, performance tot khi conflict it. Pessimistic lock chac chan hon nhung giam throughput. 90% truong hop dung optimistic concurrency + retry la du.

---

### Q48. Post-mortem analysis nen thuc hien nhu the nao? Blameless culture co nghia la gi trong thuc te?

**Answer:**

Post-mortem la qua trinh phan tich incident de **hoc hoi va ngan chan tai dien**, khong phai de quy trach nhiem.

**Blameless culture** nghia la: tap trung vao **he thong va quy trinh** da fail, khong phai **con nguoi**. Mot engineer deploy code loi len production khong phai la root cause -- root cause la: tai sao code review khong catch? Tai sao khong co automated test? Tai sao deploy process cho phep push thang len prod?

**Template post-mortem:**
```markdown
## Incident: [Ten] - [Ngay]
## Severity: SEV2 | Duration: 45 phut | Users affected: ~2000

## Timeline (UTC)
- 14:00 - Deploy version 2.3.1
- 14:05 - Alert: error rate > 5%
- 14:12 - On-call engineer acknowledged
- 14:15 - Identified: missing DB migration
- 14:20 - Rollback initiated
- 14:25 - Service restored

## Root Cause
Migration script khong duoc include trong deployment pipeline.

## Contributing Factors
- Khong co checklist cho DB migration trong deploy process
- Staging environment da apply migration thu cong truoc do

## Action Items
- [ ] Them migration check vao CI/CD pipeline (owner: DevOps, deadline: 2 tuan)
- [ ] Tao runbook cho DB migration rollback (owner: Backend lead, deadline: 1 tuan)
- [ ] Them health check endpoint kiem tra DB schema version (owner: Dev team)

## Lessons Learned
- Manual steps in deployment = ticking time bomb
- Staging phai reflect production process chinh xac
```

Quy tac: post-mortem phai duoc thuc hien trong vong **48 gio** sau incident. Moi action item phai co **owner** va **deadline** cu the. Review lai action items trong sprint planning de dam bao duoc thuc hien.

---

> **Summary:** 48 cau hoi, 6 sections, moi section 8 cau. Cac cau tra loi duoc viet o muc senior voi trade-offs, code examples, va kinh nghiem thuc te. Dung de on tap phong van vi tri Senior Full-Stack Developer (.NET Core + ReactJS).
