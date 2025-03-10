# Code Convention cho C# và ASP.NET với Clean Architecture

## 1. Sử dụng Dependency Injection (DI)

### Mục đích
Tạo instance của class thông qua DI để tăng tính kiểm thử và bảo trì.

### Chi tiết
- Sử dụng built-in DI container của ASP.NET Core
- Không tạo instance bằng toán tử `new` đối với những class có dependency

### Ví dụ tốt
```csharp
// Program.cs
builder.Services.AddScoped<IUserRepository, UserRepository>();
builder.Services.AddScoped<IUserService, UserService>();

// UserService.cs
public class UserService : IUserService
{
    private readonly IUserRepository _userRepository;
    
    public UserService(IUserRepository userRepository)
    {
        _userRepository = userRepository;
    }
}
```

### Ví dụ xấu
```csharp
public class UserService : IUserService
{
    private readonly UserRepository _userRepository;
    
    public UserService()
    {
        _userRepository = new UserRepository(); // Tránh tạo instance trực tiếp
    }
}
```

## 2. Đăng ký Dependency trong DI Container

### Mục đích
Đăng ký dịch vụ với thời gian sống phù hợp: Singleton, Scoped, Transient.

### Chi tiết
- **Singleton**: Instance được tạo một lần duy nhất trong suốt vòng đời của ứng dụng
- **Scoped**: Instance được tạo mới cho mỗi HTTP request
- **Transient**: Instance được tạo mới mỗi khi được yêu cầu

### Ví dụ
```csharp
// Program.cs
// Singleton - dịch vụ không có state hoặc state được chia sẻ
builder.Services.AddSingleton<ICacheService, RedisCacheService>();

// Scoped - phù hợp cho hầu hết các service
builder.Services.AddScoped<IUserRepository, UserRepository>();
builder.Services.AddScoped<IUserService, UserService>();

// Transient - cho các service nhẹ không lưu state
builder.Services.AddTransient<IEmailValidator, EmailValidator>();
```

## 3. Không inject nguyên cả IServiceProvider

### Mục đích
Tránh phụ thuộc trực tiếp vào container DI, chỉ inject dịch vụ cụ thể.

### Chi tiết
- Không được truyền `IServiceProvider` vào constructor
- Sử dụng Factory Pattern nếu cần tạo service động

### Ví dụ xấu
```csharp
public class UserService
{
    private readonly IServiceProvider _serviceProvider;
    
    public UserService(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }
    
    public void ProcessUser(int userId)
    {
        // Anti-pattern: tạo service từ container
        var repo = _serviceProvider.GetService<IUserRepository>();
        var user = repo.GetById(userId);
    }
}
```

### Ví dụ tốt
```csharp
public class UserService
{
    private readonly IUserRepository _userRepository;
    
    public UserService(IUserRepository userRepository)
    {
        _userRepository = userRepository;
    }
    
    public void ProcessUser(int userId)
    {
        var user = _userRepository.GetById(userId);
    }
}
```

## 4. Sử dụng Interface thay vì Implementation

### Mục đích
Định nghĩa dịch vụ qua interface để dễ mở rộng và kiểm thử.

### Chi tiết
- Tạo interface cho mỗi service
- Đăng ký interface-implementation trong DI container
- Chỉ inject interface vào constructor

### Ví dụ
```csharp
// Interface
public interface INotificationService
{
    Task SendEmailAsync(string to, string subject, string body);
    Task SendSmsAsync(string to, string message);
}

// Implementation
public class NotificationService : INotificationService
{
    public Task SendEmailAsync(string to, string subject, string body)
    {
        // Triển khai gửi email
        return Task.CompletedTask;
    }
    
    public Task SendSmsAsync(string to, string message)
    {
        // Triển khai gửi SMS
        return Task.CompletedTask;
    }
}

// Đăng ký
builder.Services.AddScoped<INotificationService, NotificationService>();

// Sử dụng
public class UserController
{
    private readonly INotificationService _notificationService;
    
    public UserController(INotificationService notificationService)
    {
        _notificationService = notificationService;
    }
}
```

## 5. Không lạm dụng Static Dependency

### Mục đích
Hạn chế class/static method để tránh phụ thuộc cứng.

### Chi tiết
- Không sử dụng static class cho business logic
- Dùng static class cho utility functions không có state hoặc dependency

### Ví dụ tốt - Utility không có dependency
```csharp
public static class StringUtils
{
    public static string Truncate(string value, int maxLength)
    {
        if (string.IsNullOrEmpty(value)) return value;
        return value.Length <= maxLength ? value : value.Substring(0, maxLength);
    }
}
```

### Ví dụ xấu - Business logic với static class
```csharp
public static class UserService
{
    public static User GetUser(int userId)
    {
        // Phụ thuộc cứng, khó kiểm thử
        var dbContext = new AppDbContext();
        return dbContext.Users.Find(userId);
    }
}
```

## 6. Không inject dependency không sử dụng

### Mục đích
Chỉ inject dịch vụ cần thiết để giảm phức tạp.

### Chi tiết
- Chỉ inject các dependency thực sự được sử dụng
- Xem xét tách dịch vụ nếu cần quá nhiều dependency

### Ví dụ xấu
```csharp
public class UserService
{
    private readonly IUserRepository _userRepository;
    private readonly IEmailService _emailService; // Không sử dụng trong class này
    private readonly ILoggerFactory _loggerFactory; // Không sử dụng
    
    public UserService(
        IUserRepository userRepository,
        IEmailService emailService,
        ILoggerFactory loggerFactory)
    {
        _userRepository = userRepository;
        _emailService = emailService;
        _loggerFactory = loggerFactory;
    }
    
    public User GetUser(int id)
    {
        return _userRepository.GetById(id);
        // _emailService và _loggerFactory không được sử dụng
    }
}
```

### Ví dụ tốt
```csharp
public class UserService
{
    private readonly IUserRepository _userRepository;
    
    public UserService(IUserRepository userRepository)
    {
        _userRepository = userRepository;
    }
    
    public User GetUser(int id)
    {
        return _userRepository.GetById(id);
    }
}
```

## 7. Dùng Options Pattern cho cấu hình

### Mục đích
Tách biệt cấu hình khỏi code bằng IOptions<T>.

### Chi tiết
- Tạo POCO class để biểu diễn cấu hình
- Đăng ký cấu hình trong DI container
- Inject IOptions<T> vào service

### Ví dụ
```csharp
// POCO class cho cấu hình
public class EmailSettings
{
    public string SmtpServer { get; set; }
    public int Port { get; set; }
    public string Username { get; set; }
    public string Password { get; set; }
}

// Đăng ký trong Program.cs
builder.Services.Configure<EmailSettings>(
    builder.Configuration.GetSection("EmailSettings"));

// Sử dụng trong service
public class EmailService : IEmailService
{
    private readonly EmailSettings _settings;
    
    public EmailService(IOptions<EmailSettings> options)
    {
        _settings = options.Value;
    }
    
    public Task SendEmailAsync(string to, string subject, string body)
    {
        // Sử dụng _settings.SmtpServer, _settings.Port, ...
        return Task.CompletedTask;
    }
}
```

## 8. Ưu tiên Constructor Injection

### Mục đích
Inject dependency qua constructor để đảm bảo tính minh bạch.

### Chi tiết
- Luôn sử dụng constructor injection thay vì property injection
- Làm rõ các dependency cần thiết

### Ví dụ tốt - Constructor Injection
```csharp
public class OrderService : IOrderService
{
    private readonly IOrderRepository _orderRepository;
    private readonly ILogger<OrderService> _logger;
    
    // Constructor Injection
    public OrderService(
        IOrderRepository orderRepository,
        ILogger<OrderService> logger)
    {
        _orderRepository = orderRepository;
        _logger = logger;
    }
}
```

### Ví dụ xấu - Property Injection
```csharp
public class OrderService : IOrderService
{
    // Property Injection - không rõ ràng về dependency
    public IOrderRepository OrderRepository { get; set; }
    public ILogger<OrderService> Logger { get; set; }
    
    public OrderService()
    {
        // Constructor trống không thể hiện rõ các dependency cần thiết
    }
}
```

## 9. Tránh Lifecycle Mismatch

### Mục đích
Không inject Scoped/Transient vào Singleton để tránh lỗi.

### Chi tiết
- Singleton service chỉ được inject các Singleton service khác
- Scoped service có thể inject Singleton và Scoped service
- Transient service có thể inject bất kỳ loại service nào

### Ví dụ xấu - Lifecycle Mismatch
```csharp
// Đăng ký dịch vụ
builder.Services.AddSingleton<ICacheService, RedisCacheService>();
builder.Services.AddScoped<IUserRepository, UserRepository>();

// KHÔNG NÊN: Singleton service inject Scoped service
public class RedisCacheService : ICacheService
{
    private readonly IUserRepository _userRepository; // Scoped service
    
    public RedisCacheService(IUserRepository userRepository)
    {
        _userRepository = userRepository; // Lỗi tiềm ẩn!
    }
}
```

### Ví dụ tốt
```csharp
// Đăng ký đúng lifecycle
builder.Services.AddSingleton<ICacheService, RedisCacheService>();
builder.Services.AddSingleton<IGlobalConfig, GlobalConfig>();

// Singleton service chỉ inject Singleton service
public class RedisCacheService : ICacheService
{
    private readonly IGlobalConfig _config; // Singleton service
    
    public RedisCacheService(IGlobalConfig config)
    {
        _config = config;
    }
}
```

## 10. Validate Dependency Injection

### Mục đích
Kiểm tra tính hợp lệ của dependency (ví dụ: không null).

### Chi tiết
- Validate dependencies trong constructor
- Sử dụng guard clauses để đảm bảo dependency không null

### Ví dụ
```csharp
public class ProductService : IProductService
{
    private readonly IProductRepository _productRepository;
    private readonly ILogger<ProductService> _logger;
    
    public ProductService(
        IProductRepository productRepository,
        ILogger<ProductService> logger)
    {
        _productRepository = productRepository ?? 
            throw new ArgumentNullException(nameof(productRepository));
        _logger = logger ?? 
            throw new ArgumentNullException(nameof(logger));
    }
}
```

## 11. Sử dụng Factory Pattern cho dynamic dependency

### Mục đích
Tạo đối tượng động qua factory khi cần runtime decision.

### Chi tiết
- Sử dụng Factory pattern khi cần chọn implementation trong runtime
- Đăng ký factory trong DI container

### Ví dụ
```csharp
// Factory interface
public interface IPaymentProcessorFactory
{
    IPaymentProcessor Create(PaymentMethod method);
}

// Factory implementation
public class PaymentProcessorFactory : IPaymentProcessorFactory
{
    private readonly IServiceProvider _serviceProvider;
    
    public PaymentProcessorFactory(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }
    
    public IPaymentProcessor Create(PaymentMethod method)
    {
        return method switch
        {
            PaymentMethod.CreditCard => _serviceProvider.GetRequiredService<ICreditCardProcessor>(),
            PaymentMethod.BankTransfer => _serviceProvider.GetRequiredService<IBankTransferProcessor>(),
            PaymentMethod.Paypal => _serviceProvider.GetRequiredService<IPaypalProcessor>(),
            _ => throw new ArgumentException($"Unsupported payment method: {method}")
        };
    }
}

// Đăng ký factory
builder.Services.AddSingleton<IPaymentProcessorFactory, PaymentProcessorFactory>();

// Sử dụng factory
public class OrderService
{
    private readonly IPaymentProcessorFactory _processorFactory;
    
    public OrderService(IPaymentProcessorFactory processorFactory)
    {
        _processorFactory = processorFactory;
    }
    
    public void ProcessPayment(Order order)
    {
        var processor = _processorFactory.Create(order.PaymentMethod);
        processor.Process(order.Amount);
    }
}
```

## 12. Đảm bảo dịch vụ disposable được giải phóng

### Mục đích
Triển khai IDisposable cho dịch vụ có tài nguyên cần giải phóng.

### Chi tiết
- Implement IDisposable cho các service có quản lý tài nguyên
- Cẩn thận với các dịch vụ Singleton

### Ví dụ
```csharp
public class DatabaseService : IDatabaseService, IDisposable
{
    private readonly SqlConnection _connection;
    private bool _disposed = false;
    
    public DatabaseService(string connectionString)
    {
        _connection = new SqlConnection(connectionString);
        _connection.Open();
    }
    
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }
    
    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                _connection?.Dispose();
            }
            
            _disposed = true;
        }
    }
    
    // Destructor
    ~DatabaseService()
    {
        Dispose(false);
    }
}
```

## 13. Tránh Service Locator Pattern

### Mục đích
Không dùng GetService() trong business logic.

### Chi tiết
- Không sử dụng IServiceProvider.GetService() trong business logic
- Tránh ẩn dependency bằng cách truy cập trực tiếp vào container

### Ví dụ xấu - Service Locator Pattern
```csharp
public class UserController : ControllerBase
{
    private readonly IServiceProvider _serviceProvider;
    
    public UserController(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }
    
    [HttpGet("{id}")]
    public IActionResult GetUser(int id)
    {
        // Anti-pattern: Service Locator
        var userService = _serviceProvider.GetService<IUserService>();
        var user = userService.GetUser(id);
        return Ok(user);
    }
}
```

### Ví dụ tốt
```csharp
public class UserController : ControllerBase
{
    private readonly IUserService _userService;
    
    public UserController(IUserService userService)
    {
        _userService = userService;
    }
    
    [HttpGet("{id}")]
    public IActionResult GetUser(int id)
    {
        var user = _userService.GetUser(id);
        return Ok(user);
    }
}
```

## 14. Giữ constructor đơn giản

### Mục đích
Không thực hiện logic phức tạp trong constructor.

### Chi tiết
- Constructor chỉ nên khởi tạo và validate dependency
- Đưa logic phức tạp vào các method riêng

### Ví dụ xấu
```csharp
public class UserService : IUserService
{
    private readonly IUserRepository _userRepository;
    private readonly IEnumerable<User> _cachedAdminUsers;
    
    public UserService(IUserRepository userRepository)
    {
        _userRepository = userRepository;
        
        // Logic phức tạp trong constructor - không nên
        _cachedAdminUsers = _userRepository.GetAllUsers()
            .Where(u => u.IsAdmin)
            .ToList();
    }
}
```

### Ví dụ tốt
```csharp
public class UserService : IUserService
{
    private readonly IUserRepository _userRepository;
    private IEnumerable<User> _cachedAdminUsers;
    private readonly object _lockObject = new object();
    
    public UserService(IUserRepository userRepository)
    {
        _userRepository = userRepository;
    }
    
    private void InitializeAdminCache()
    {
        if (_cachedAdminUsers == null)
        {
            lock (_lockObject)
            {
                if (_cachedAdminUsers == null)
                {
                    _cachedAdminUsers = _userRepository.GetAllUsers()
                        .Where(u => u.IsAdmin)
                        .ToList();
                }
            }
        }
    }
    
    public IEnumerable<User> GetAdminUsers()
    {
        InitializeAdminCache();
        return _cachedAdminUsers;
    }
}
```

## 15. Sử dụng Decorator Pattern cho cross-cutting concerns

### Mục đích
Áp dụng AOP qua decorator để xử lý logging, cache...

### Chi tiết
- Sử dụng decorator để thêm chức năng cho service mà không sửa đổi code
- Áp dụng cho logging, caching, validation, transaction...

### Ví dụ
```csharp
// Interface
public interface IUserService
{
    User GetUser(int id);
}

// Implementation
public class UserService : IUserService
{
    private readonly IUserRepository _repository;
    
    public UserService(IUserRepository repository)
    {
        _repository = repository;
    }
    
    public User GetUser(int id)
    {
        return _repository.GetById(id);
    }
}

// Decorator cho caching
public class CachedUserService : IUserService
{
    private readonly IUserService _userService;
    private readonly IMemoryCache _cache;
    
    public CachedUserService(
        IUserService userService,
        IMemoryCache cache)
    {
        _userService = userService;
        _cache = cache;
    }
    
    public User GetUser(int id)
    {
        string cacheKey = $"User_{id}";
        
        if (!_cache.TryGetValue(cacheKey, out User user))
        {
            user = _userService.GetUser(id);
            _cache.Set(cacheKey, user, TimeSpan.FromMinutes(10));
        }
        
        return user;
    }
}

// Đăng ký với decorator
builder.Services.AddScoped<IUserRepository, UserRepository>();
builder.Services.AddScoped<IUserService, UserService>();
builder.Services.Decorate<IUserService, CachedUserService>();
```

## 16. Tách biệt layer bằng DI

### Mục đích
Inject abstraction giữa các tầng (ví dụ: Repository → Service).

### Chi tiết
- Định nghĩa interface cho mỗi tầng
- Đảm bảo phụ thuộc một chiều từ tầng cao đến tầng thấp

### Ví dụ Clean Architecture
```csharp
// Domain/Entities
public class User 
{
    public int Id { get; set; }
    public string Name { get; set; }
}

// Application/Interfaces
public interface IUserRepository
{
    User GetById(int id);
    void Add(User user);
}

// Application/Services
public class UserService : IUserService
{
    private readonly IUserRepository _repository;
    
    public UserService(IUserRepository repository)
    {
        _repository = repository;
    }
    
    public UserDto GetUser(int id)
    {
        var user = _repository.GetById(id);
        return new UserDto { Id = user.Id, Name = user.Name };
    }
}

// Infrastructure/Repositories
public class UserRepository : IUserRepository
{
    private readonly AppDbContext _dbContext;
    
    public UserRepository(AppDbContext dbContext)
    {
        _dbContext = dbContext;
    }
    
    public User GetById(int id)
    {
        return _dbContext.Users.Find(id);
    }
    
    public void Add(User user)
    {
        _dbContext.Users.Add(user);
        _dbContext.SaveChanges();
    }
}

// Presentation/Controllers
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;
    
    public UsersController(IUserService userService)
    {
        _userService = userService;
    }
    
    [HttpGet("{id}")]
    public IActionResult GetUser(int id)
    {
        var user = _userService.GetUser(id);
        return Ok(user);
    }
}
```

## 17. Kiểm thử với Mock/Stub qua DI

### Mục đích
Thay thế dependency bằng mock trong unit test.

### Chi tiết
- Sử dụng framework like Moq để tạo mock object
- Test service logic độc lập với implementation

### Ví dụ
```csharp
[Fact]
public void GetUser_ExistingId_ReturnsUser()
{
    // Arrange
    var userId = 1;
    var mockUser = new User { Id = userId, Name = "Test User" };
    
    // Mock repository
    var mockRepo = new Mock<IUserRepository>();
    mockRepo.Setup(repo => repo.GetById(userId))
        .Returns(mockUser);
    
    // Create service with mock dependency
    var userService = new UserService(mockRepo.Object);
    
    // Act
    var result = userService.GetUser(userId);
    
    // Assert
    Assert.Equal(userId, result.Id);
    Assert.Equal("Test User", result.Name);
    mockRepo.Verify(repo => repo.GetById(userId), Times.Once);
}
```

## 18. Tài liệu hóa các dependency

### Mục đích
Ghi chú rõ ràng các dependency và mục đích sử dụng.

### Chi tiết
- Sử dụng XML documentation trên constructor
- Giải thích mục đích của từng dependency

### Ví dụ
```csharp
/// <summary>
/// Dịch vụ xử lý đơn hàng và thanh toán
/// </summary>
public class OrderService : IOrderService
{
    private readonly IOrderRepository _orderRepository;
    private readonly IPaymentService _paymentService;
    private readonly ILogger<OrderService> _logger;
    
    /// <summary>
    /// Khởi tạo OrderService với các dependencies cần thiết
    /// </summary>
    /// <param name="orderRepository">Repository để truy xuất và lưu trữ đơn hàng</param>
    /// <param name="paymentService">Dịch vụ xử lý thanh toán</param>
    /// <param name="logger">Logger để ghi log các hoạt động của service</param>
    public OrderService(
        IOrderRepository orderRepository,
        IPaymentService paymentService,
        ILogger<OrderService> logger)
    {
        _orderRepository = orderRepository;
        _paymentService = paymentService;
        _logger = logger;
    }
    
    // Implementation
}
```

## 19. Sử dụng third-party DI container nếu cần

### Mục đích
Tích hợp Autofac, Ninject... khi cần tính năng nâng cao.

### Chi tiết
- ASP.NET Core có built-in DI container, nhưng đôi khi cần tính năng nâng cao
- Autofac cung cấp tính năng như assembly scanning, decorator, etc.

### Ví dụ với Autofac
```csharp
// Program.cs
builder.Host.UseServiceProviderFactory(new AutofacServiceProviderFactory());
builder.Host.ConfigureContainer<ContainerBuilder>(containerBuilder =>
{
    // Đăng ký theo module
    containerBuilder.RegisterModule<DataAccessModule>();
    containerBuilder.RegisterModule<ServiceModule>();
    
    // Đăng ký tất cả repository trong assembly
    containerBuilder.RegisterAssemblyTypes(typeof(Program).Assembly)
        .Where(t => t.Name.EndsWith("Repository"))
        .AsImplementedInterfaces()
        .InstancePerLifetimeScope();
        
    // Đăng ký decorator
    containerBuilder.RegisterType<UserService>().As<IUserService>();
    containerBuilder.RegisterDecorator<CachedUserService, IUserService>();
});
```

## 20. Tránh Circular Dependency

### Mục đích
Phát hiện và loại bỏ phụ thuộc vòng giữa các service.

### Chi tiết
- Tránh tình huống ServiceA phụ thuộc ServiceB và ServiceB phụ thuộc ServiceA
- Refactor để tách biệt trách nhiệm

### Ví dụ xấu - Circular Dependency
```csharp
// ServiceA phụ thuộc ServiceB
public class ServiceA : IServiceA
{
    private readonly IServiceB _serviceB;
    
    public ServiceA(IServiceB serviceB)
    {
        _serviceB = serviceB;
    }
}

// ServiceB phụ thuộc ServiceA - Circular Dependency!
public class ServiceB : IServiceB
{
    private readonly IServiceA _serviceA;
    
    public ServiceB(IServiceA serviceA)
    {
        _serviceA = serviceA;
    }
}
```

### Ví dụ tốt - Giải quyết Circular Dependency
```csharp
// Cách 1: Tạo interface mới IServiceC cho chức năng chung
public interface IServiceC
{
    void CommonMethod();
}

public class ServiceA : IServiceA
{
    private readonly IServiceB _serviceB;
    
    public ServiceA(IServiceB serviceB)
    {
        _serviceB = serviceB;
    }
}

public class ServiceB : IServiceB
{
    private readonly IServiceC _serviceC;
    
    public ServiceB(IServiceC serviceC)
    {
        _serviceC = serviceC;
    }
}

// Cách 2: Sử dụng event/delegate
public class ServiceA : IServiceA
{
    public event Action<string> OnSomethingHappened;
    
    public void DoSomething()
    {
        // Xử lý logic
        OnSomethingHappened?.Invoke("Something happened");
    }
}

public class ServiceB : IServiceB
{
    public ServiceB(IServiceA serviceA)
    {
        serviceA.OnSomethingHappened += HandleEvent;
    }
    
    private void HandleEvent(string message)
    {
        // Xử lý sự kiện
    }
}
```
