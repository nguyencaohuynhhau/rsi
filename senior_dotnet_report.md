# Báo Cáo Phân Tích & Tổng Hợp Câu Hỏi Phỏng Vấn Senior .NET Developer

**Được tạo bởi: Backend Architect / Document Generator Agent**
**Ngày tạo:** 16/09/2026

Báo cáo này tổng hợp, phân tích và trả lời chuyên sâu các câu hỏi phỏng vấn dành cho vị trí Senior .NET Developer từ các tài liệu trong hệ thống (`senior .net developer.md`, `senior_fullstack_developer_opus.md`, `senior_fullstack_developer_sonnet.md`). Các câu trả lời được trình bày dưới góc độ của một Senior/Architect, tập trung vào hiệu năng, kiến trúc và best practices.

---

## 1. Ngôn Ngữ C# & Hiệu Năng (C# Core & Performance)

### Q: Cơ chế `async/await` hoạt động ra sao? Phân biệt `Task` và `ValueTask`.
**Trả lời (Góc nhìn Senior):**
- **Cơ chế:** C# compiler chuyển đổi `async/await` thành một state machine (`IAsyncStateMachine`). Khi gặp `await` cho một I/O bound, thread hiện tại được trả về Thread Pool thay vì bị block. Khi I/O hoàn thành, một thread bất kỳ từ Thread Pool sẽ tiếp tục thực thi đoạn code tiếp theo. Điều này giúp ứng dụng xử lý hàng chục ngàn request đồng thời mà không bị cạn kiệt thread.
- **Task vs ValueTask:** 
  - `Task` là reference type, luôn tiêu tốn bộ nhớ trên Heap (gây áp lực cho Garbage Collector).
  - `ValueTask` là một `struct`, không tốn Heap allocation nếu kết quả trả về ngay lập tức một cách đồng bộ (ví dụ: lấy dữ liệu từ memory cache).
  - *Best Practice:* Chỉ dùng `ValueTask` ở các "hot paths" (gọi liên tục hàng ngàn lần) và khi tỷ lệ hoàn thành đồng bộ cao. Tuyệt đối không `await` một `ValueTask` nhiều lần.

### Q: Tránh lỗi Deadlock trong lập trình bất đồng bộ như thế nào?
**Trả lời:**
- Tránh sử dụng `.Result` hoặc `.Wait()` trong các ngữ cảnh có `SynchronizationContext` (như WinForms, WPF, hoặc ASP.NET MVC cũ).
- Trong ASP.NET Core không còn `SynchronizationContext`, nhưng dùng `.Result` vẫn gây ra hiện tượng **Thread Pool Starvation** (tắc nghẽn thread pool).
- *Quy tắc vàng:* "Async all the way" (Bất đồng bộ toàn bộ từ Controller xuống tận Database) và luôn dùng `.ConfigureAwait(false)` khi viết thư viện (class library) để không capture context.

### Q: C# Records và Pattern Matching mang lại lợi ích gì so với Class thông thường?
**Trả lời:**
- **Records** cung cấp Value Equality (so sánh theo giá trị thay vì tham chiếu) và Immutability (bất biến) thông qua `init-only properties` và biểu thức `with`. Rất phù hợp để làm DTO (Data Transfer Objects) hoặc Event Messages trong Microservices.
- **Pattern Matching** kết hợp với Records giúp viết code ngắn gọn, rành mạch hơn thay vì dùng hàng loạt lệnh `if-else` lồng nhau, đặc biệt hữu dụng khi build các state machine hoặc logic business phức tạp.

---

## 2. Kiến Trúc ASP.NET Core

### Q: Dependency Injection (DI) hoạt động thế nào? Tránh lỗi Captive Dependency ra sao?
**Trả lời:**
- DI container quản lý vòng đời theo 3 mức: `Transient` (tạo mới mỗi lần), `Scoped` (một bản cho mỗi HTTP request) và `Singleton` (một bản duy nhất toàn ứng dụng).
- **Captive Dependency** xảy ra khi inject một service có vòng đời ngắn (như `Scoped DbContext`) vào một service có vòng đời dài hơn (như `Singleton`). Hậu quả là DbContext bị giữ lại mãi mãi, gây lỗi memory leak và data cũ.
- *Khắc phục:* Đảm bảo bật `ValidateScopes = true` (mặc định bật ở môi trường Development) để framework báo lỗi ngay lúc khởi động. Nếu thực sự cần gọi Scoped trong Singleton (như BackgroundService), phải tự inject `IServiceProvider` và gọi `CreateScope()`.

### Q: Giải thích Middleware Pipeline. Làm sao để thiết kế luồng xử lý chuẩn?
**Trả lời:**
- Middleware hoạt động theo mô hình Russian Doll (búp bê Nga), các request đi qua từng lớp và đi ngược trở lại. Việc sắp xếp thứ tự là tối quan trọng.
- Thứ tự chuẩn: `ExceptionHandler` $\rightarrow$ `HSTS/HTTPS` $\rightarrow$ `Routing` $\rightarrow$ `CORS` $\rightarrow$ `Authentication` $\rightarrow$ `Authorization` $\rightarrow$ `Custom Middlewares` $\rightarrow$ `Endpoints`. 
- Nếu đặt CORS sau Auth, request Preflight (`OPTIONS`) sẽ bị chặn và fail.

---

## 3. Database & Entity Framework Core

### Q: EF Core Change Tracking hoạt động thế nào? Làm sao tối ưu hiệu năng truy vấn?
**Trả lời:**
- `ChangeTracker` lưu lại snapshot ban đầu của các entities. Khi gọi `SaveChanges()`, nó so sánh dữ liệu hiện tại với snapshot để tạo câu lệnh SQL (`UPDATE`, `INSERT`, `DELETE`).
- **Tối ưu hóa (Senior Tricks):**
  - Luôn dùng `.AsNoTracking()` cho các truy vấn chỉ đọc (read-only) để bỏ qua overhead của ChangeTracker.
  - Dùng `.AsSplitQuery()` khi Include nhiều quan hệ 1-N (One-to-Many) để tránh hiện tượng nổ tổ hợp Cartesian (Cartesian explosion) khiến dữ liệu phình to.
  - Sử dụng Projection (`.Select()`) thay vì kéo nguyên một Entity khổng lồ khi chỉ cần hiển thị vài cột lên UI.
  - Tận dụng `ExecuteUpdateAsync` và `ExecuteDeleteAsync` (EF Core 7+) để cập nhật/xóa hàng loạt không cần tracking.

### Q: Lựa chọn giữa EF Core và Dapper khi nào?
**Trả lời:**
- Không cần chọn 1 trong 2, mà nên kết hợp theo kiến trúc **CQRS**:
  - Dùng **EF Core** cho luồng Write (Command): Tận dụng Change Tracking, Unit of Work và khả năng xử lý business rules toàn vẹn.
  - Dùng **Dapper** cho luồng Read (Query): Ánh xạ thẳng raw SQL có tính tối ưu cao (ví dụ: báo cáo thống kê phức tạp, view join nhiều bảng), đem lại tốc độ cực nhanh.

---

## 4. Kiến Trúc Hệ Thống (System Architecture) & Microservices

### Q: Xử lý giao dịch phân tán (Distributed Transactions) trong Microservices thế nào?
**Trả lời:**
- Tránh dùng 2PC (Two-Phase Commit) vì gây lock database và dễ điểm chết (Single Point of Failure).
- Áp dụng **Saga Pattern** với các bù trừ (Compensating actions). Dùng Orchestration (có một orchestrator trung tâm quản lý luồng) để dễ theo dõi tiến trình và debug.
- Sử dụng **Outbox Pattern**: Khi ghi vào DB của service A, đồng thời ghi event vào một bảng `Outbox` chung một giao dịch (transaction). Một background worker sẽ lấy event từ Outbox để publish lên Kafka/RabbitMQ. Điều này đảm bảo tính nhất quán (At-least-once delivery).

### Q: Tại sao và khi nào dùng CQRS?
**Trả lời:**
- CQRS chia tách rõ ràng trách nhiệm Đọc (Query) và Ghi (Command). 
- Dùng cho các hệ thống có tỉ lệ Đọc/Ghi chênh lệch lớn, domain phức tạp cần áp dụng Domain-Driven Design (DDD). 
- Triển khai thường thông qua thư viện `MediatR` trong .NET để tạo pipeline behaviors (cắt ngang validation, logging). Không nên dùng cho các app CRUD đơn giản vì sẽ over-engineering.

---

## 5. Security & Observability

### Q: Quản lý Authentication và Authorization bằng JWT?
**Trả lời:**
- JWT là Stateless, server không cần lưu Session.
- *Rủi ro:* Token bị lộ không thể thu hồi ngay lập tức (cho đến khi hết hạn). 
- *Giải pháp:* Dùng Access Token có TTL (thời gian sống) cực ngắn (5-15 phút) kết hợp Refresh Token lưu dưới HttpOnly, Secure Cookie để xoay vòng token. Chống XSS bằng cách tuyệt đối không lưu token trong `localStorage`.

### Q: Observability trong Production?
**Trả lời:**
- Dùng **OpenTelemetry** để thiết lập Distributed Tracing, truyền `Correlation ID` qua mọi HTTP Headers và Message queues, giúp log được xâu chuỗi trong ELK Stack hoặc Jaeger.
- Ghi log có cấu trúc (Structured Logging) qua Serilog thay vì nối chuỗi, giúp query dễ dàng.
- Bật ASP.NET Core Health Checks để Kubernetes có thể quản lý vòng đời container (`Liveness` và `Readiness`).

---
*Báo cáo được tổng hợp dựa trên sự phân tích đa chiều giữa các phương pháp kỹ thuật tốt nhất (Best Practices) hiện tại trên nền tảng .NET.*

---

## 6. Các Chủ Đề Nâng Cao Mới Nhất (2024-2025: .NET 8/9 & C# 13)

### Q: Xây dựng hệ thống Cloud-Native & Observability với .NET Aspire như thế nào?
**Trả lời (Góc nhìn Architect):**
- Trong các hệ thống phân tán phức tạp, việc setup môi trường local giống với production, cũng như cấu hình tracing/metrics là một ác mộng. **.NET Aspire** (.NET 8/9) ra đời để giải quyết bài toán này.
- Bằng cách sử dụng `AppHost` và `ServiceDefaults`, ta có thể inject các chuẩn về OpenTelemetry (Logs, Metrics, Tracing), HealthChecks, và Resilience (Polly) một cách nhất quán cho tất cả các microservices.
- .NET Aspire hỗ trợ tự động discover và kết nối các resource (Redis, Postgres, RabbitMQ) mà không cần hardcode connection strings. Triển khai (Deploy) kiến trúc lên cloud dễ dàng thông qua `azd` (Azure Developer CLI) hoặc dùng `Aspirate` để sinh ra Kubernetes manifests.

### Q: Tích hợp AI & Agentic Workflows với Semantic Kernel?
**Trả lời:**
- Để xây dựng hệ thống RAG (Retrieval-Augmented Generation) hoặc AI Agent tự động hóa, **Semantic Kernel** là lựa chọn tối ưu thay vì gọi trực tiếp OpenAI API, giúp tránh phụ thuộc nhà cung cấp (Vendor lock-in).
- Cấu trúc trừu tượng linh hoạt: **Plugins, Planners (Agentic workflow), Prompts, và Memory**.
- Có thể hoán đổi Model (ví dụ: OpenAI GPT-4 sang Ollama/Phi-3 local) chỉ bằng việc cấu hình lại connector mà không đụng vào business logic. Hơn nữa, .NET 9 tích hợp sẵn OpenTelemetry cho AI, giúp tracking chính xác latency và lượng token tiêu thụ.

### Q: Khi nào áp dụng Native AOT và Minimal APIs trong kiến trúc Serverless?
**Trả lời:**
- **Native AOT (Ahead-of-Time):** Rất phù hợp cho AWS Lambda, Azure Functions hoặc high-density containers vì dung lượng image siêu nhỏ (vài chục MB), khởi động tức thì (millisecond startup), không cần JIT compiler, memory footprint cực thấp.
- **Trade-offs:** Không hỗ trợ dynamic loading (`Assembly.Load`), các thư viện dùng Reflection mạnh lúc runtime sẽ bị crash.
- *Giải pháp:* Sử dụng **Source Generators** (được giới thiệu ở C# 11/12) như `System.Text.Json` source generator và tính năng Interceptors/Compile-time DI để sinh code ngay từ lúc build.

### Q: Tối ưu hiệu năng cực đại với các tính năng của C# 13?
**Trả lời:**
- **Về `System.Threading.Lock`:** C# 13 giới thiệu class `Lock` mới, ưu việt hơn pattern `lock (new object())` truyền thống do không tốn chi phí object header và tạo thêm gánh nặng cho Garbage Collector (GC). Runtime tối ưu trực tiếp `Lock` mới để nhẹ hơn và tránh tranh chấp khóa (lock contention) tốt hơn.
- **Về `ref struct` implement interface:** C# 13 cho phép `ref struct` (luôn nằm trên Stack) implement các interfaces (`IEquatable<T>`, `IDisposable`). Nhờ vậy, có thể viết các hàm generic nhận `Span<T>` mà không bị Boxing (chuyển qua Heap), đảm bảo absolute zero-allocation trong các xử lý chuỗi/mạng cường độ cao.

---

## 7. Tư Duy Kiến Trúc Cơ Sở Dữ Liệu (Database Architecture)

### Q: Việc tạo Index không phải là "viên đạn bạc". Khi nào SQL Server quyết định Table Scan thay vì dùng Index đã tạo?
**Trả lời (Góc nhìn Architect):**
- **Non-SARGable Queries:** Dùng hàm trực tiếp trên cột (ví dụ: `YEAR(Date) = 2024` hoặc `LIKE '%abc'`). Điều này làm hỏng cấu trúc B-Tree khiến SQL Server không thể dùng Index Seek.
- **Implicit Conversion:** Kiểu dữ liệu truyền vào khác kiểu dữ liệu của cột (như truyền `INT` vào cột `VARCHAR`), SQL Server phải convert toàn bộ dữ liệu trong bảng để so sánh.
- **Low Selectivity & Key Lookup Cost:** Nếu Index có độ chọn lọc thấp (ví dụ trả về 40% số dòng của bảng), chi phí nhảy (Key Lookup) từ Non-Clustered Index về Clustered Index (Random I/O) sẽ đắt hơn nhiều so với việc Table Scan (Sequential I/O). SQL Server Optimizer sẽ tự quyết định bỏ qua Index.

### Q: Làm sao tối ưu Index cho các cột có Low Cardinality (Giới tính, Trạng thái)?
**Trả lời:**
- Không bao giờ đánh Index đơn lẻ trên các cột này. 
- *Giải pháp thiết kế:* 
  1. Dùng **Filtered Index**: Chỉ lập chỉ mục trên phần dữ liệu có ý nghĩa (Ví dụ: `WHERE Status = 'Active'`).
  2. Dùng **Composite Index**: Gom cột Low Cardinality đứng trước cột High Cardinality nếu chúng hay được query cùng nhau (theo quy tắc Leftmost Prefix Rule).
  3. Dùng **Covering Index (`INCLUDE`)**: Kéo các cột cần SELECT vào Leaf Node của Index để tránh hoàn toàn thao tác Key Lookup.

---

## 8. System Design: Kiến Trúc Xử Lý Dữ Liệu Lớn & AI Streaming

### Q: Trong hệ thống Video AI Analytics, phân tích Trade-off giữa Batch Processing và Real-time Streaming (RTSP).
**Trả lời:**
- **Batch Processing (Xử lý theo lô):** Hệ thống lấy video từ camera theo chu kỳ (poll) rồi giao cho các worker tải về và xử lý. 
  - *Ưu điểm:* Độ tin cậy cực cao, không rớt dữ liệu dù mạng chập chờn. Nếu worker chết, job chỉ việc retry qua Queue. Dễ dàng kiểm soát tải GPU (Scale-out Workers). Thích hợp cho bài toán điểm danh, điểm danh bù.
  - *Nhược điểm:* Độ trễ cao, không phù hợp cho cảnh báo an ninh tức thời.
- **Real-time Streaming (RTSP):** 
  - *Ưu điểm:* Độ trễ thấp (vài mili-giây), cảnh báo xâm nhập thời gian thực.
  - *Nhược điểm:* Tốn chi phí duy trì hàng ngàn connection liên tục. Mạng rớt đồng nghĩa mất frame vĩnh viễn. Rủi ro Out-of-Memory (OOM) nếu traffic tăng đột biến.

### Q: Làm sao tránh OOM khi tích hợp AI Pipeline vào Backend Backend .NET/Python?
**Trả lời:**
- Phân bổ Workload triệt để: **GPU** chỉ được dùng cho quá trình Inference (chạy mạng Neural như SCRFD Detect, ArcFace Embedding). Các task còn lại như Decode Video (nếu không dùng NVDEC), xử lý ma trận (Cosine Similarity), Crop ảnh và Upload lên Cloudflare R2 phải được đẩy sang **CPU**.
- Sử dụng **Cô Lập Tiến Trình (Fault Isolation)**: Thay vì dùng thread, hãy spawn các subprocess (multiprocessing pool) để gán cho mỗi job một vùng VRAM riêng. Nếu memory leak xảy ra, container không sập mà chỉ chết đúng subprocess đó và tự động phục hồi.

---

## 9. Domain-Driven Design (DDD) & Cấu Trúc Hệ Thống

### Q: Phân biệt Entity, Value Object và Aggregate Root trong thiết kế DDD?
**Trả lời (Góc nhìn Senior):**
- **Entity (Thực Thể):** Được định danh bằng `ID` duy nhất. Vòng đời có thể thay đổi state theo thời gian. Ví dụ: `Order`, `Customer`. So sánh hai Entity phải dựa trên ID chứ không dựa vào thuộc tính.
- **Value Object (Đối Tượng Giá Trị):** Không có danh tính. Bất biến (Immutable) sau khi tạo. Bị ràng buộc hoàn toàn bởi giá trị của nó. Ví dụ: `Address` (đổi tên đường là sinh ra đối tượng Address mới), `Money`. So sánh bằng giá trị các thuộc tính.
- **Aggregate Root (Gốc Tập Hợp):** Là một Entity đại diện cho cả một cụm các Entity con và Value Object đi kèm (Ví dụ: `Order` là Root, chứa các `OrderItem`). Nó bảo vệ tính toàn vẹn (consistency boundary). *Nguyên tắc:* Các service bên ngoài chỉ được phép tương tác với Aggregate Root, tuyệt đối không chọc thẳng vào các Entity con.

### Q: Khi nào nên sử dụng Domain Services và Domain Events?
**Trả lời:**
- **Domain Services:** Chứa các business logic rải rác trên nhiều Aggregates hoặc quá phức tạp để nhét vào một Entity (Ví dụ: `CalculateDiscountService` cần tương tác với `Order`, `CustomerRank` và `PromotionCampaign`).
- **Domain Events:** Được kích hoạt (fire) khi có một thay đổi state quan trọng xảy ra trong Domain (ví dụ: `OrderCreatedEvent`). Dùng để decouple logic (side-effects) ra khỏi Entity chính, thường được các Event Handler bắt lại để thực thi các tác vụ như gửi Email, đẩy qua Outbox table cho Microservices khác xử lý mà không làm phình to hàm `CreateOrder`.

---

## 10. Xử Lý Đồng Thời (Concurrency) & Bài Toán Bán Lố (Over-selling)

### Q: Hệ thống Flash Sale có 1 tồn kho nhưng 10.000 người cùng bấm mua. Làm sao để giải quyết triệt để bài toán Race Condition và thiết kế hệ thống thế nào để không sập?
**Trả lời (Góc nhìn Architect):**
Bản chất của việc bán lố (over-selling) là do nhiều luồng (thread) cùng đọc được `Stock = 1`, vượt qua vòng kiểm tra `if (stock > 0)` và cùng gọi lệnh Update trừ kho. Để giải quyết, ta cần thiết kế 3 tầng phòng thủ: Tối ưu CSDL (SQL Server), Chặn request ở RAM (Redis), và Xếp hàng bằng Message Queue (RabbitMQ).

#### Tầng 1: Khóa ở cấp độ Database (SQL Server)
Nếu chỉ dựa vào Database, chúng ta có 2 cách tiếp cận để chặn Race Condition:
- **Cách 1: Khóa bi quan (Pessimistic Locking)**
  - Dùng câu lệnh `SELECT ... WITH (UPDLOCK, ROWLOCK)` để khóa cứng dòng dữ liệu. 
  - Người đầu tiên sẽ giữ khóa (lock) dòng này cho đến khi giao dịch (transaction) hoàn tất. 9.999 người đến sau sẽ bị SQL Server bắt đứng chờ (Block) ở trạng thái chờ khóa.
  - *Trade-off:* An toàn tuyệt đối nhưng gây nghẽn cổ chai (bottleneck) nghiêm trọng. Nếu số lượng truy cập quá lớn, sẽ dẫn đến cạn kiệt Connection Pool và làm sập toàn bộ CSDL. Ít được khuyên dùng cho Flash Sale.
- **Cách 2: Khóa lạc quan (Optimistic Locking)**
  - Không khóa dòng dữ liệu lúc SELECT. Thay vào đó, gộp việc kiểm tra vào chính câu lệnh UPDATE (vì bản thân UPDATE trên SQL Server đã có Row-level lock nguyên tử).
  - *Ví dụ SQL:* `UPDATE Products SET Stock = Stock - 1 WHERE Id = 1 AND Stock >= 1`
  - *Kết quả:* Thread đầu tiên chạy sẽ trả về `RowsAffected = 1` (Mua thành công). Những thread sau chạy vào sẽ bị sai điều kiện `Stock >= 1` (vì người đầu tiên đã trừ kho về 0), trả về `RowsAffected = 0` (Hết hàng). EF Core sẽ văng lỗi `DbUpdateConcurrencyException`. Cách này không gây nghẽn CSDL.

#### Tầng 2: Đưa chốt chặn lên RAM với Redis (Bảo vệ Database)
Để không làm sập Database với 10.000 request, ta phải chặn chúng ở Redis. Trước Flash Sale, nạp tồn kho lên Redis: `await _redisDb.StringSetAsync("product_stock:1", 100);`.
Khi Flash sale diễn ra, ta có 2 cách trừ kho nguyên tử trên Redis:
- **Cách 1: Trừ kho trực tiếp (Không dùng Lua Script)**
  - Dùng hàm `StringDecrementAsync`, thao tác này nguyên tử (atomic): trừ 1 và trả về giá trị mới ngay lập tức.
  - *Code:* `long remainingStock = await _redisDb.StringDecrementAsync("product_stock:1");`
  - Nếu `remainingStock >= 0` $\rightarrow$ Cho phép đi tiếp. Nếu `< 0` $\rightarrow$ Trả về lỗi "Hết hàng" (HTTP 400).
- **Cách 2: Dùng Lua Script (Kiểm soát chặt chẽ nhất)**
  - Redis chạy đơn luồng (single-threaded). Bằng cách gửi một đoạn Lua Script, Redis sẽ thực thi khối code đó nguyên tử từ đầu đến cuối mà không bị can thiệp bởi request khác.
  - *Lua Script:*
    ```lua
    local stock = tonumber(redis.call('GET', KEYS[1]))
    if stock <= 0 then
        return 0 -- Hết hàng
    else
        redis.call('DECR', KEYS[1])
        return 1 -- Trừ thành công
    end
    ```
  - *Lợi ích:* Đảm bảo kho không bao giờ rớt xuống số âm (như Cách 1). Trả về chính xác 1 (thành công) hoặc 0 (thất bại).

#### Tầng 3: Xếp hàng với RabbitMQ & Cập nhật UI với SignalR
- **API (Producer):** Những người lọt qua được cửa ải Redis ở Tầng 2 (tức là có hàng) sẽ được Backend nhét message `OrderRequest` vào RabbitMQ. API lập tức trả về HTTP 202 (Accepted) kèm lời nhắn "Hệ thống đang xử lý". Giao diện người dùng sẽ hiện Spinner quay.
- **Worker (Consumer):** Lấy từng message ra từ RabbitMQ theo thứ tự (`PrefetchCount = 1`) và thong thả thực thi lệnh Update vào Database (Sử dụng Optimistic Update ở Tầng 1 để chốt lần cuối).
- **Phản hồi UI (SignalR):** Sau khi ghi DB thành công, Worker dùng SignalR bắn message trực tiếp tới `ConnectionId` của User đó: "Chúc mừng, thanh toán thành công!". Trình duyệt nhận được message sẽ tắt Spinner và chuyển hướng đến trang thanh toán. Đây là cơ chế Asynchronous Request-Reply hoàn hảo cho các hệ thống tải cao.
