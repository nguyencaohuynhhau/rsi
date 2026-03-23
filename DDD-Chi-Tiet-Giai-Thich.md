# Domain-Driven Design (DDD) - Giải Thích Chi Tiết

## 1. Tổng Quan về DDD

**Domain-Driven Design (DDD)** là một phương pháp phát triển phần mềm tập trung vào việc mô hình hóa phần mềm phù hợp với domain (lĩnh vực nghiệp vụ) của nó. DDD được phát triển bởi Eric Evans và được mô tả trong cuốn sách "Domain-Driven Design: Tackling Complexity in the Heart of Software".

### Mục Tiêu Chính của DDD:
- **Tập trung vào nghiệp vụ**: Code phản ánh đúng logic nghiệp vụ thực tế
- **Giao tiếp hiệu quả**: Ngôn ngữ chung giữa developers và domain experts
- **Tổ chức code rõ ràng**: Cấu trúc code dễ hiểu và bảo trì
- **Giảm độ phức tạp**: Chia nhỏ hệ thống thành các bounded contexts

---

## 2. Các Thành Phần Chính của DDD

### 2.1. Domain Layer (Lớp Domain)

**Vị trí trong dự án**: `GoFoodBeverage.Domain`

#### 2.1.1. Entity (Thực Thể)

**Định nghĩa**: Entity là một đối tượng có danh tính (identity) duy nhất và vòng đời riêng. Hai entity có thể có cùng thuộc tính nhưng vẫn là hai đối tượng khác nhau nếu có ID khác nhau.

**Ví dụ trong dự án**:
```csharp
// Order.cs - Một Entity
public class Order : BaseEntity
{
    public Guid Id { get; set; }  // Identity duy nhất
    public Guid? StoreId { get; set; }
    public Guid? CustomerId { get; set; }
    public EnumOrderStatus StatusId { get; set; }
    // ... các thuộc tính khác
}
```

**Đặc điểm**:
- Có `Id` duy nhất (thường là `Guid`)
- Có thể thay đổi trạng thái theo thời gian
- Được so sánh bằng ID, không phải bằng giá trị thuộc tính
- Kế thừa từ `BaseEntity` để có các thuộc tính chung (CreatedTime, IsDeleted, etc.)

#### 2.1.2. Value Object (Đối Tượng Giá Trị)

**Định nghĩa**: Value Object là đối tượng không có danh tính, được xác định hoàn toàn bởi giá trị của các thuộc tính. Hai value object có cùng giá trị thì được coi là giống nhau.

**Ví dụ**:
```csharp
// Address có thể là Value Object
public class Address
{
    public string Street { get; set; }
    public string City { get; set; }
    public string PostalCode { get; set; }
    
    // So sánh bằng giá trị, không phải ID
    public override bool Equals(object obj)
    {
        if (obj is Address other)
            return Street == other.Street && 
                   City == other.City && 
                   PostalCode == other.PostalCode;
        return false;
    }
}
```

**Đặc điểm**:
- **Immutable** (bất biến): Không thể thay đổi sau khi tạo
- So sánh bằng giá trị, không phải ID
- Không có vòng đời riêng
- Thường được nhúng trong Entity

#### 2.1.3. Aggregate (Tập Hợp)

**Định nghĩa**: Aggregate là một nhóm các Entity và Value Object được coi như một đơn vị duy nhất. Aggregate có một Aggregate Root (gốc) là Entity duy nhất được truy cập từ bên ngoài.

**Ví dụ trong dự án**:
```
Order (Aggregate Root)
├── OrderItem (Entity con)
│   ├── OrderItemOption (Entity con)
│   └── OrderItemTopping (Entity con)
├── OrderPaymentTransaction (Entity con)
└── OrderDelivery (Entity con)
```

**Quy tắc**:
- Chỉ có Aggregate Root được truy cập từ bên ngoài
- Các Entity con chỉ được truy cập thông qua Aggregate Root
- Aggregate đảm bảo tính nhất quán dữ liệu (consistency boundary)
- Một transaction chỉ nên thay đổi một Aggregate

#### 2.1.4. Domain Service (Dịch Vụ Domain)

**Định nghĩa**: Domain Service chứa logic nghiệp vụ không thuộc về một Entity cụ thể nào, mà liên quan đến nhiều Entity hoặc logic phức tạp.

**Khi nào sử dụng**:
- Logic nghiệp vụ cần nhiều Entity
- Logic phức tạp không phù hợp với một Entity
- Tính toán hoặc quy tắc nghiệp vụ phức tạp

**Ví dụ**:
```csharp
public class OrderCalculationService
{
    public decimal CalculateTotalPrice(Order order, List<Discount> discounts)
    {
        // Logic tính toán phức tạp liên quan đến Order và Discount
    }
}
```

#### 2.1.5. Repository Interface (Giao Diện Repository)

**Định nghĩa**: Repository là một pattern cung cấp cách truy cập tập hợp các Entity một cách trừu tượng. Interface của Repository được định nghĩa trong Domain layer.

**Vị trí**: `GoFoodBeverage.Interfaces`

**Ví dụ**:
```csharp
public interface IOrderRepository
{
    Task<Order> GetByIdAsync(Guid id);
    Task<List<Order>> GetByCustomerIdAsync(Guid customerId);
    Task AddAsync(Order order);
    Task UpdateAsync(Order order);
}
```

**Lợi ích**:
- Domain layer không phụ thuộc vào Infrastructure
- Dễ dàng test bằng mock repository
- Có thể thay đổi cách lưu trữ mà không ảnh hưởng domain logic

#### 2.1.6. Domain Events (Sự Kiện Domain)

**Định nghĩa**: Domain Events đại diện cho những sự kiện quan trọng xảy ra trong domain. Các thành phần khác có thể lắng nghe và phản ứng với các sự kiện này.

**Ví dụ**:
```csharp
public class OrderCreatedEvent : IDomainEvent
{
    public Guid OrderId { get; set; }
    public Guid CustomerId { get; set; }
    public DateTime CreatedAt { get; set; }
}
```

**Sử dụng**:
- Thông báo khi Order được tạo
- Kích hoạt các hành động khác (gửi email, cập nhật inventory, etc.)
- Tách biệt các bounded contexts

---

### 2.2. Application Layer (Lớp Ứng Dụng)

**Vị trí trong dự án**: `GoFoodBeverage.Application`

> **Lưu ý quan trọng**: Application layer trong dự án này sử dụng **Vertical Slice Architecture** kết hợp với **CQRS Pattern** và **MediatR**. Xem tài liệu chi tiết tại: `documents/Vertical-Slice-Architecture-Chi-Tiet.md`

#### 2.2.1. Use Cases / Features (Vertical Slice Architecture)

**Định nghĩa**: Application layer chứa các use cases (trường hợp sử dụng) của hệ thống. Mỗi use case đại diện cho một hành động mà người dùng có thể thực hiện. Dự án sử dụng **Vertical Slice Architecture** để tổ chức code theo features thay vì theo layers.

**Cấu trúc trong dự án** (Vertical Slice):
```
GoFoodBeverage.Application/
└── Features/                    ← VERTICAL SLICE ARCHITECTURE
    ├── Orders/
    │   ├── Commands/           ← CQRS Commands (thay đổi trạng thái)
    │   │   ├── CancelOrderRequest.cs (Request + Handler)
    │   │   ├── UpdateOrderStatusRequest.cs
    │   │   └── ...
    │   └── Queries/            ← CQRS Queries (đọc dữ liệu)
    │       ├── GetOrderByIdRequest.cs (Request + Handler)
    │       ├── GetOrdersManagementRequest.cs
    │       └── ...
    ├── Products/
    │   ├── Commands/
    │   │   ├── CreateProductRequest.cs
    │   │   └── UpdateProductRequest.cs
    │   └── Queries/
    │       ├── GetProductsRequest.cs
    │       └── GetProductByIdRequest.cs
    └── Accounts/
        ├── Commands/
        └── Queries/
```

**Đặc điểm của Vertical Slice Architecture**:
- Mỗi feature có folder riêng (Orders, Products, Accounts, etc.)
- Tách Commands (thay đổi) và Queries (đọc) theo CQRS
- Request và Handler thường nằm trong cùng file
- Sử dụng MediatR để routing Request → Handler
- Mỗi slice độc lập, ít phụ thuộc lẫn nhau

**Ví dụ với CQRS Pattern**:
```csharp
// Command - Thay đổi trạng thái
public class CreateOrderCommand : IRequest<OrderDto>
{
    public Guid CustomerId { get; set; }
    public List<OrderItemDto> Items { get; set; }
}

// Handler - Xử lý command
public class CreateOrderCommandHandler : IRequestHandler<CreateOrderCommand, OrderDto>
{
    private readonly IOrderRepository _orderRepository;
    
    public async Task<OrderDto> Handle(CreateOrderCommand request, CancellationToken cancellationToken)
    {
        // 1. Tạo Order entity
        // 2. Validate business rules
        // 3. Lưu vào database
        // 4. Publish domain events
        // 5. Return DTO
    }
}

// Query - Đọc dữ liệu
public class GetOrderQuery : IRequest<OrderDto>
{
    public Guid OrderId { get; set; }
}
```

#### 2.2.2. DTOs (Data Transfer Objects)

**Định nghĩa**: DTO là các đối tượng được sử dụng để truyền dữ liệu giữa các layer. DTO khác với Entity ở chỗ nó chỉ chứa dữ liệu, không có logic nghiệp vụ.

**Ví dụ**:
```csharp
public class OrderDto
{
    public Guid Id { get; set; }
    public string CustomerName { get; set; }
    public decimal TotalAmount { get; set; }
    public List<OrderItemDto> Items { get; set; }
}
```

**Lợi ích**:
- Tách biệt domain model khỏi API contract
- Có thể thêm/bớt fields mà không ảnh hưởng Entity
- Tối ưu hiệu suất (chỉ trả về dữ liệu cần thiết)

#### 2.2.3. Application Services

**Định nghĩa**: Application Services điều phối các hoạt động giữa các Repository và Domain Services để thực hiện use cases.

**Trách nhiệm**:
- Điều phối các Repository
- Gọi Domain Services khi cần
- Xử lý transactions
- Mapping giữa Entity và DTO
- Publish domain events

**Ví dụ**:
```csharp
public class OrderApplicationService
{
    private readonly IOrderRepository _orderRepository;
    private readonly IProductRepository _productRepository;
    private readonly IUnitOfWork _unitOfWork;
    
    public async Task<OrderDto> CreateOrderAsync(CreateOrderCommand command)
    {
        // 1. Validate
        // 2. Tạo Order aggregate
        // 3. Lưu vào database
        // 4. Publish events
        // 5. Return DTO
    }
}
```

#### 2.2.4. Mappings (AutoMapper)

**Định nghĩa**: Sử dụng AutoMapper để chuyển đổi giữa Entity và DTO.

**Ví dụ**:
```csharp
public class OrderProfile : Profile
{
    public OrderProfile()
    {
        CreateMap<Order, OrderDto>();
        CreateMap<CreateOrderCommand, Order>();
    }
}
```

---

### 2.3. Infrastructure Layer (Lớp Hạ Tầng)

**Vị trí trong dự án**: `GoFoodBeverage.Infrastructure`

#### 2.3.1. Repository Implementation

**Định nghĩa**: Infrastructure layer triển khai các Repository interface được định nghĩa trong Domain layer.

**Ví dụ**:
```csharp
public class OrderRepository : GenericRepository<Order>, IOrderRepository
{
    private readonly GoFoodBeverageDbContext _context;
    
    public OrderRepository(GoFoodBeverageDbContext context) : base(context)
    {
        _context = context;
    }
    
    public async Task<Order> GetByIdWithItemsAsync(Guid id)
    {
        return await _context.Orders
            .Include(o => o.OrderItems)
            .FirstOrDefaultAsync(o => o.Id == id);
    }
}
```

**Đặc điểm**:
- Sử dụng Entity Framework Core
- Triển khai các phương thức truy vấn cụ thể
- Xử lý database operations

#### 2.3.2. DbContext

**Định nghĩa**: DbContext là trung tâm của Entity Framework, quản lý kết nối database và các Entity.

**Ví dụ trong dự án**:
```csharp
public class GoFoodBeverageDbContext : DbContext
{
    public DbSet<Order> Orders { get; set; }
    public DbSet<Product> Products { get; set; }
    // ... các DbSet khác
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Cấu hình relationships, indexes, etc.
    }
}
```

**Trách nhiệm**:
- Quản lý database connection
- Tracking changes
- Thực hiện migrations
- Cấu hình entity relationships

#### 2.3.3. Unit of Work Pattern

**Định nghĩa**: Unit of Work đảm bảo tất cả các thay đổi được commit trong một transaction duy nhất.

**Ví dụ**:
```csharp
public class UnitOfWork : IUnitOfWork
{
    private readonly GoFoodBeverageDbContext _context;
    
    public async Task<int> SaveChangesAsync()
    {
        return await _context.SaveChangesAsync();
    }
    
    public async Task BeginTransactionAsync()
    {
        await _context.Database.BeginTransactionAsync();
    }
    
    public async Task CommitTransactionAsync()
    {
        await _context.Database.CommitTransactionAsync();
    }
}
```

**Lợi ích**:
- Đảm bảo tính nhất quán dữ liệu
- Quản lý transactions hiệu quả
- Dễ dàng rollback khi có lỗi

#### 2.3.4. External Services Integration

**Định nghĩa**: Infrastructure layer tích hợp với các dịch vụ bên ngoài như:
- Payment gateways
- Email services
- SMS services
- File storage
- Message queues (RabbitMQ)

**Ví dụ trong dự án**:
- `GoFoodBeverage.Payment` - Tích hợp thanh toán
- `GoFoodBeverage.Email` - Gửi email
- `GoFoodBeverage.EventBusRabbitMQ` - Message queue

---

### 2.4. Presentation Layer (Lớp Giao Diện)

**Vị trí trong dự án**: `GoFoodBeverage.WebApi`

#### 2.4.1. Controllers

**Định nghĩa**: Controllers nhận HTTP requests và gọi Application Services.

**Ví dụ**:
```csharp
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly IMediator _mediator;
    
    [HttpPost]
    public async Task<ActionResult<OrderDto>> CreateOrder(CreateOrderCommand command)
    {
        var order = await _mediator.Send(command);
        return Ok(order);
    }
}
```

**Trách nhiệm**:
- Nhận HTTP requests
- Validate input
- Gọi Application layer
- Trả về HTTP responses
- Xử lý errors

#### 2.4.2. Middleware

**Định nghĩa**: Middleware xử lý các cross-cutting concerns như:
- Authentication/Authorization
- Logging
- Error handling
- Request validation

**Ví dụ trong dự án**:
```csharp
// Error handling middleware
public class ErrorHandlingMiddleware
{
    // Xử lý exceptions và trả về error response
}
```

---

## 3. Các Pattern và Khái Niệm Quan Trọng

### 3.1. Bounded Context (Ngữ Cảnh Giới Hạn)

**Định nghĩa**: Bounded Context là ranh giới trong đó một domain model cụ thể được áp dụng. Mỗi bounded context có:
- Domain model riêng
- Ubiquitous Language riêng
- Team riêng (có thể)

**Ví dụ trong dự án**:
- `GoFoodBeverage` - Context chính cho F&B
- `GoFnbSocial` - Context cho tính năng social
- `GoFoodBeverage.POS` - Context cho POS system

**Lợi ích**:
- Giảm độ phức tạp
- Cho phép các team làm việc độc lập
- Mỗi context có thể có công nghệ khác nhau

### 3.2. Ubiquitous Language (Ngôn Ngữ Chung)

**Định nghĩa**: Ubiquitous Language là ngôn ngữ chung được sử dụng bởi developers và domain experts. Tên class, method, variable phải phản ánh ngôn ngữ này.

**Ví dụ**:
- `Order` thay vì `Purchase`
- `Customer` thay vì `User`
- `Product` thay vì `Item`
- `Branch` thay vì `Location`

### 3.3. CQRS (Command Query Responsibility Segregation)

**Định nghĩa**: Tách biệt các operations đọc (Query) và ghi (Command).

**Commands** (Thay đổi trạng thái):
```csharp
CreateOrderCommand
UpdateOrderCommand
CancelOrderCommand
```

**Queries** (Đọc dữ liệu):
```csharp
GetOrderQuery
GetOrdersByCustomerQuery
GetOrderStatisticsQuery
```

**Lợi ích**:
- Tối ưu riêng cho read và write
- Có thể scale độc lập
- Dễ dàng cache cho queries

### 3.4. Event-Driven Architecture

**Định nghĩa**: Sử dụng events để giao tiếp giữa các components.

**Ví dụ**:
```csharp
// Khi Order được tạo
OrderCreatedEvent → 
    ├── InventoryService: Trừ số lượng sản phẩm
    ├── NotificationService: Gửi email cho customer
    └── AnalyticsService: Cập nhật thống kê
```

**Lợi ích**:
- Loose coupling giữa các services
- Dễ dàng thêm features mới
- Hỗ trợ eventual consistency

---

## 4. Cấu Trúc Thư Mục trong Dự Án

```
GoFoodBeverage/
├── Domain/                    # Domain Layer
│   ├── Entities/             # Các Entity
│   ├── Enums/                # Các Enum
│   ├── Base/                 # Base classes (BaseEntity)
│   └── Settings/             # Domain settings
│
├── Application/              # Application Layer
│   ├── Features/             # Use cases (CQRS)
│   ├── Mappings/             # AutoMapper profiles
│   ├── Providers/            # Application providers
│   └── IntegrationEvents/    # Integration events
│
├── Infrastructure/           # Infrastructure Layer
│   ├── Repositories/         # Repository implementations
│   ├── Contexts/             # DbContext
│   └── Extensions/           # Infrastructure extensions
│
├── Interfaces/               # Repository & Service interfaces
│   ├── IRepositories/        # Repository interfaces
│   └── IServices/            # Service interfaces
│
├── WebApi/                   # Presentation Layer
│   ├── Controllers/          # API Controllers
│   ├── Middlewares/          # Custom middlewares
│   └── wwwroot/              # Static files
│
└── Common/                   # Shared code
    ├── Constants/            # Constants
    ├── Extensions/           # Extension methods
    └── Models/               # Common models
```

---

## 5. Luồng Xử Lý Điển Hình

### 5.1. Tạo Order Mới

```
1. Client gửi HTTP POST /api/orders
   ↓
2. OrdersController nhận request
   ↓
3. Controller tạo CreateOrderCommand
   ↓
4. Mediator gửi command đến CreateOrderCommandHandler
   ↓
5. Handler:
   - Validate input
   - Tạo Order entity (Domain)
   - Gọi IOrderRepository.AddAsync()
   ↓
6. Repository (Infrastructure) lưu vào database
   ↓
7. Handler publish OrderCreatedEvent
   ↓
8. Event handlers xử lý:
   - Update inventory
   - Send notification
   - Update analytics
   ↓
9. Handler map Order → OrderDto
   ↓
10. Controller trả về OrderDto cho client
```

### 5.2. Đọc Order

```
1. Client gửi HTTP GET /api/orders/{id}
   ↓
2. OrdersController nhận request
   ↓
3. Controller tạo GetOrderQuery
   ↓
4. Mediator gửi query đến GetOrderQueryHandler
   ↓
5. Handler:
   - Gọi IOrderRepository.GetByIdAsync()
   ↓
6. Repository query database
   ↓
7. Handler map Order → OrderDto
   ↓
8. Controller trả về OrderDto cho client
```

---

## 6. Best Practices

### 6.1. Domain Layer
- ✅ Entity chỉ chứa logic nghiệp vụ
- ✅ Sử dụng Value Objects cho các giá trị phức tạp
- ✅ Đảm bảo Aggregate boundaries
- ✅ Sử dụng Domain Events cho side effects
- ❌ Không phụ thuộc vào Infrastructure

### 6.2. Application Layer
- ✅ Mỗi use case là một class riêng
- ✅ Sử dụng DTOs, không expose Entity
- ✅ Validate input ở Application layer
- ✅ Xử lý transactions
- ❌ Không chứa logic nghiệp vụ phức tạp

### 6.3. Infrastructure Layer
- ✅ Triển khai Repository interfaces
- ✅ Xử lý database operations
- ✅ Tích hợp external services
- ❌ Không chứa business logic

### 6.4. Presentation Layer
- ✅ Controllers chỉ điều phối
- ✅ Validate HTTP requests
- ✅ Xử lý errors và trả về proper status codes
- ❌ Không chứa business logic

---

## 7. Lợi Ích của DDD

1. **Code dễ hiểu**: Code phản ánh đúng nghiệp vụ
2. **Dễ bảo trì**: Cấu trúc rõ ràng, tách biệt concerns
3. **Dễ test**: Có thể mock repositories và test logic độc lập
4. **Scalable**: Có thể scale từng layer độc lập
5. **Team collaboration**: Developers và domain experts dùng chung ngôn ngữ
6. **Flexible**: Dễ dàng thay đổi implementation mà không ảnh hưởng domain

---

## 8. Kết Luận

DDD là một phương pháp mạnh mẽ để xây dựng phần mềm phức tạp. Bằng cách tập trung vào domain và tổ chức code theo các layers rõ ràng, chúng ta có thể:

- Xây dựng phần mềm phản ánh đúng nghiệp vụ
- Dễ dàng bảo trì và mở rộng
- Cải thiện giao tiếp giữa team members
- Giảm độ phức tạp của hệ thống

Dự án GoFoodBeverage đã áp dụng các nguyên tắc DDD với cấu trúc rõ ràng:
- **Domain Layer**: Chứa business logic và entities
- **Application Layer**: Điều phối use cases
- **Infrastructure Layer**: Xử lý database và external services
- **Presentation Layer**: Expose APIs

Việc hiểu rõ từng thành phần và cách chúng tương tác với nhau sẽ giúp bạn phát triển và bảo trì hệ thống hiệu quả hơn.
