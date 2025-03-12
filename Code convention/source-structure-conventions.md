# Quy ước viết mã (Code Convention) cho ASP.NET C# Clean Architecture

## Mục lục
1. [Kiến trúc tầng](#1-kiến-trúc-tầng)
2. [Cấu trúc thư mục](#2-cấu-trúc-thư-mục)
3. [Quy tắc đặt tên](#3-quy-tắc-đặt-tên)
4. [Tổ chức file](#4-tổ-chức-file)
5. [Phân tách trách nhiệm](#5-phân-tách-trách-nhiệm)
6. [Dependency Injection](#6-dependency-injection)
7. [Kích thước file](#7-kích-thước-file)
8. [Comment và tài liệu hóa](#8-comment-và-tài-liệu-hóa)
9. [Tránh dư thừa](#9-tránh-dư-thừa)

## 1. Kiến trúc tầng

### 1.1. Domain Layer
- **Độc lập hoàn toàn**, không phụ thuộc vào bất kỳ tầng nào khác
- Chứa các thành phần cốt lõi: Entities, Value Objects, Exceptions
- Biểu diễn logic nghiệp vụ chính, không phụ thuộc framework/công nghệ cụ thể

```csharp
// Domain/Entities/User.cs
public class User
{
    public Guid Id { get; private set; }
    public string Email { get; private set; }
    public string Name { get; private set; }
    
    public User(string email, string name)
    {
        if (string.IsNullOrEmpty(email))
            throw new DomainException("Email không được để trống");
            
        Id = Guid.NewGuid();
        Email = email;
        Name = name;
    }
    
    public void ChangeName(string newName)
    {
        if (string.IsNullOrEmpty(newName))
            throw new DomainException("Tên không được để trống");
            
        Name = newName;
    }
}
```

### 1.2. Application Layer
- **Chỉ phụ thuộc vào Domain Layer**
- Chứa logic ứng dụng (use cases)
- Định nghĩa interfaces cho các dịch vụ bên ngoài

```csharp
// Application/Services/IUserService.cs
public interface IUserService
{
    Task<UserDto> GetUserByIdAsync(Guid id);
    Task<UserDto> CreateUserAsync(CreateUserCommand command);
}

// Application/Commands/CreateUserCommand.cs
public class CreateUserCommand : IRequest<UserDto>
{
    public string Email { get; set; }
    public string Name { get; set; }
}
```

### 1.3. Infrastructure Layer
- **Phụ thuộc vào Domain và Application Layers**
- Triển khai các interfaces được định nghĩa trong Application
- Xử lý cơ sở dữ liệu, giao tiếp với dịch vụ bên ngoài

```csharp
// Infrastructure/Repositories/UserRepository.cs
public class UserRepository : IUserRepository
{
    private readonly AppDbContext _context;
    
    public UserRepository(AppDbContext context)
    {
        _context = context;
    }
    
    public async Task<User> GetByIdAsync(Guid id)
    {
        return await _context.Users.FindAsync(id);
    }
}
```

### 1.4. Presentation Layer
- **Phụ thuộc vào Application Layer**
- Xử lý các yêu cầu từ người dùng
- Chứa Controllers, Filters, View Models

```csharp
// Presentation/Controllers/UserController.cs
[ApiController]
[Route("api/[controller]")]
public class UserController : ControllerBase
{
    private readonly IMediator _mediator;
    
    public UserController(IMediator mediator)
    {
        _mediator = mediator;
    }
    
    [HttpPost]
    public async Task<ActionResult<UserDto>> CreateUser(CreateUserCommand command)
    {
        var result = await _mediator.Send(command);
        return Ok(result);
    }
}
```

## 2. Cấu trúc thư mục

### 2.1. Cấu trúc dự án
- Mỗi tầng là một project riêng biệt
- Tổ chức solution như sau:

```
Solution/
├── src/
│   ├── ProjectName.Domain/
│   ├── ProjectName.Application/
│   ├── ProjectName.Infrastructure/
│   └── ProjectName.Api/ (Presentation)
└── tests/
    ├── ProjectName.Domain.Tests/
    ├── ProjectName.Application.Tests/
    ├── ProjectName.Infrastructure.Tests/
    └── ProjectName.Api.Tests/
```

### 2.2. Tổ chức thư mục bên trong mỗi tầng

#### Domain Layer
```
ProjectName.Domain/
├── Entities/
├── ValueObjects/
├── Enums/
├── Events/
└── Exceptions/
```

#### Application Layer
```
ProjectName.Application/
├── Common/
│   ├── Behaviors/
│   ├── Exceptions/
│   └── Interfaces/
├── DTOs/
└── Features/
    ├── Users/
    │   ├── Commands/
    │   └── Queries/
    └── Products/
```

#### Infrastructure Layer
```
ProjectName.Infrastructure/
├── Data/
│   ├── Configurations/
│   ├── Migrations/
│   ├── Repositories/
│   └── AppDbContext.cs
├── ExternalServices/
└── DependencyInjection.cs
```

#### Presentation Layer
```
ProjectName.Api/
├── Controllers/
├── Filters/
├── Middlewares/
└── Program.cs
```

## 3. Quy tắc đặt tên

### 3.1. Quy ước chung
- Sử dụng **PascalCase** cho tên class, interface, property, enum, method
- Sử dụng **camelCase** cho tên biến local, parameter
- Sử dụng **_camelCase** cho private field (với dấu gạch dưới đầu tiên)
- Tên phải phản ánh rõ mục đích, không quá chung chung

### 3.2. Tên file và thư mục
- Tên file phải trùng với tên class/interface chính trong file
- Dùng **PascalCase** cho tên file và thư mục
- Không sử dụng số hoặc ký tự đặc biệt (trừ trường hợp cần thiết)

```
UserService.cs
ProductRepository.cs
CreateOrderCommand.cs
```

### 3.3. Đặt tên theo chức năng
- **Controllers**: Thêm hậu tố "Controller" (VD: UserController)
- **Interfaces**: Thêm tiền tố "I" (VD: IUserRepository)
- **DTOs**: Thêm hậu tố "Dto" (VD: UserDto)
- **Commands/Queries**: Thêm hậu tố "Command" hoặc "Query" (VD: CreateUserCommand)

```csharp
// Domain
public interface IUserRepository { }

// Application
public class UserDto { }
public class CreateUserCommand { }

// Infrastructure
public class UserRepository : IUserRepository { }

// Presentation
public class UserController : ControllerBase { }
```

## 4. Tổ chức file

### 4.1. Domain Layer
- Mỗi Entity/Value Object/Exception đặt trong file riêng
- Tên file phải trùng với tên class bên trong

```
Domain/Entities/User.cs
Domain/Entities/Product.cs
Domain/ValueObjects/Address.cs
Domain/Exceptions/DomainException.cs
```

### 4.2. Application Layer
- Mỗi Interface/Service/Command/Query đặt trong file riêng
- DTOs có thể nhóm theo entity hoặc tính năng

```
Application/Interfaces/IUserService.cs
Application/Commands/CreateUserCommand.cs
Application/Queries/GetUserByIdQuery.cs
Application/DTOs/UserDto.cs
```

### 4.3. Infrastructure Layer
- File liên quan đến database/repository/external service đặt ở thư mục phù hợp
- Repository cho mỗi Entity đặt trong file riêng

```
Infrastructure/Data/Repositories/UserRepository.cs
Infrastructure/Data/Configurations/UserConfiguration.cs
Infrastructure/ExternalServices/EmailService.cs
```

### 4.4. Presentation Layer
- Mỗi Controller đảm nhiệm một nhóm tính năng liên quan
- Các Middleware, Filter đặt trong thư mục riêng

```
Presentation/Controllers/UserController.cs
Presentation/Controllers/ProductController.cs
Presentation/Middlewares/ExceptionHandlingMiddleware.cs
```

## 5. Phân tách trách nhiệm

### 5.1. Domain Layer
- Chỉ chứa logic nghiệp vụ thuần túy
- Không tham chiếu tới cơ sở dữ liệu hoặc UI

```csharp
// Domain/Entities/Order.cs - Logic nghiệp vụ nằm trong domain
public class Order
{
    private readonly List<OrderItem> _items = new();
    public IReadOnlyCollection<OrderItem> Items => _items.AsReadOnly();
    
    public void AddItem(Product product, int quantity)
    {
        if (product == null)
            throw new DomainException("Không thể thêm sản phẩm null");
            
        var existingItem = _items.FirstOrDefault(i => i.ProductId == product.Id);
        if (existingItem != null)
            existingItem.IncreaseQuantity(quantity);
        else
            _items.Add(new OrderItem(product.Id, quantity, product.Price));
    }
    
    public decimal CalculateTotal() => _items.Sum(item => item.GetSubTotal());
}
```

### 5.2. Application Layer
- Điều phối logic nghiệp vụ, không triển khai chi tiết
- Định nghĩa interface, không triển khai

```csharp
// Application/Features/Orders/Commands/CreateOrder/CreateOrderCommandHandler.cs
public class CreateOrderCommandHandler : IRequestHandler<CreateOrderCommand, OrderDto>
{
    private readonly IOrderRepository _orderRepository;
    private readonly IProductRepository _productRepository;
    
    public CreateOrderCommandHandler(
        IOrderRepository orderRepository,
        IProductRepository productRepository)
    {
        _orderRepository = orderRepository;
        _productRepository = productRepository;
    }
    
    public async Task<OrderDto> Handle(CreateOrderCommand request, CancellationToken cancellationToken)
    {
        var order = new Order(request.UserId);
        
        foreach (var item in request.Items)
        {
            var product = await _productRepository.GetByIdAsync(item.ProductId);
            order.AddItem(product, item.Quantity);
        }
        
        await _orderRepository.AddAsync(order);
        
        return new OrderDto { /* map từ order sang OrderDto */ };
    }
}
```

### 5.3. Infrastructure Layer
- Triển khai các interface đã định nghĩa trong Application
- Xử lý giao tiếp với cơ sở dữ liệu, dịch vụ bên ngoài

```csharp
// Infrastructure/Data/Repositories/OrderRepository.cs
public class OrderRepository : IOrderRepository
{
    private readonly AppDbContext _context;
    
    public OrderRepository(AppDbContext context)
    {
        _context = context;
    }
    
    public async Task<Order> GetByIdAsync(Guid id)
    {
        return await _context.Orders
            .Include(o => o.Items)
            .FirstOrDefaultAsync(o => o.Id == id);
    }
    
    public async Task AddAsync(Order order)
    {
        await _context.Orders.AddAsync(order);
    }
}
```

### 5.4. Presentation Layer
- Chỉ xử lý yêu cầu từ người dùng, chuyển đến Application
- Không chứa logic nghiệp vụ

```csharp
// Presentation/Controllers/OrderController.cs
[ApiController]
[Route("api/[controller]")]
public class OrderController : ControllerBase
{
    private readonly IMediator _mediator;
    
    public OrderController(IMediator mediator)
    {
        _mediator = mediator;
    }
    
    [HttpPost]
    public async Task<ActionResult<OrderDto>> CreateOrder(CreateOrderCommand command)
    {
        var result = await _mediator.Send(command);
        return CreatedAtAction(nameof(GetOrderById), new { id = result.Id }, result);
    }
}
```

## 6. Dependency Injection

### 6.1. Nguyên tắc cơ bản
- Inject dependency qua constructor
- Inject interface, không inject concrete class
- Tránh sử dụng Service Locator pattern

### 6.2. Đăng ký Dependency
- Đăng ký các dependency trong Program.cs/Startup.cs
- Sử dụng lifetime phù hợp (Scoped, Transient, Singleton)

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Đăng ký các dependency
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
    
builder.Services.AddScoped<IUserRepository, UserRepository>();
builder.Services.AddScoped<IOrderRepository, OrderRepository>();
builder.Services.AddScoped<IProductRepository, ProductRepository>();

builder.Services.AddTransient<IEmailService, EmailService>();

builder.Services.AddMediatR(cfg => cfg.RegisterServicesFromAssembly(Assembly.GetExecutingAssembly()));

// Thêm controllers
builder.Services.AddControllers();

var app = builder.Build();

// Cấu hình pipeline
app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

## 7. Kích thước file

### 7.1. Giới hạn kích thước
- Mỗi file không quá 200-300 dòng code
- Nếu file lớn hơn, cân nhắc tách thành nhiều file hoặc class
- Một file chỉ chứa một class/interface chính

### 7.2. Phương pháp tổ chức class lớn
- Sử dụng partial class để tách code của class lớn
- Tách logic thành các class nhỏ hơn có trách nhiệm rõ ràng

```csharp
// Thay vì một class lớn:
public class UserService
{
    // 300+ dòng code với nhiều phương thức
}

// Tách thành nhiều file với partial class:
// UserService.cs
public partial class UserService
{
    // Constructor và các thành phần chung
}

// UserService.Authentication.cs
public partial class UserService
{
    // Các phương thức liên quan đến authentication
}
```

## 8. Comment và tài liệu hóa

### 8.1. Nguyên tắc chung
- Viết code tự giải thích, ưu tiên tên biến/phương thức rõ ràng
- Comment để giải thích "tại sao" hơn là "làm gì"
- Comment ngắn gọn ở đầu file

### 8.2. Comment ở đầu file

```csharp
// Purpose: Xử lý logic người dùng
// Author: Nguyễn Văn A
// Created: 2023-10-20
```

### 8.3. XML Documentation cho API

```csharp
/// <summary>
/// Tạo mới một người dùng với thông tin cơ bản
/// </summary>
/// <param name="email">Email của người dùng</param>
/// <param name="name">Tên đầy đủ của người dùng</param>
/// <returns>User entity mới được tạo</returns>
public User CreateUser(string email, string name)
{
    // Implementation
}
```

## 9. Tránh dư thừa

### 9.1. Nguyên tắc DRY (Don't Repeat Yourself)
- Không sao chép code giữa các tầng
- Logic dùng lại đặt ở Application/Domain
- Sử dụng extension method cho các thao tác phổ biến

```csharp
// Thay vì lặp lại code validation trong nhiều nơi:
public class UserController
{
    public IActionResult CreateUser(string email)
    {
        if (string.IsNullOrEmpty(email) || !email.Contains("@"))
            return BadRequest("Email không hợp lệ");
        // ...
    }
}

// Hãy sử dụng helper/extension:
public static class ValidationHelper
{
    public static bool IsValidEmail(this string email)
    {
        return !string.IsNullOrEmpty(email) && email.Contains("@");
    }
}

// Và sử dụng như sau:
public IActionResult CreateUser(string email)
{
    if (!email.IsValidEmail())
        return BadRequest("Email không hợp lệ");
    // ...
}
```

### 9.2. Tránh Anti-patterns
- Tránh God Classes (class quá lớn làm quá nhiều việc)
- Tránh Swiss Army Knife Interfaces (interface với quá nhiều phương thức)

```csharp
// Anti-pattern: God Class
public class UserManager
{
    // Phương thức về authentication
    public bool Login(string username, string password) { ... }
    public void Logout() { ... }
    
    // Phương thức về profile
    public void UpdateProfile(UserProfile profile) { ... }
    public UserProfile GetProfile(Guid userId) { ... }
    
    // Phương thức về permissions
    public bool HasPermission(Guid userId, string permission) { ... }
    
    // Phương thức về email
    public void SendWelcomeEmail(Guid userId) { ... }
}

// Cách tốt hơn: Tách thành nhiều service nhỏ
public class AuthenticationService { ... }
public class UserProfileService { ... }
public class PermissionService { ... }
public class EmailNotificationService { ... }
```