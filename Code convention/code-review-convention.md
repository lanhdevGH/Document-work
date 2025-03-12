# Code Convention cho Backend C# ASP.NET theo Clean Architecture

## 1. Quy trình Code Review

- **Pull Request (PR)** phải có ít nhất 1 người review trước khi merge
- PR phải có test nếu tính năng yêu cầu
- PR cần được chia nhỏ để dễ review (tối đa 400 dòng code thay đổi)
- Không được merge PR khi chưa được approve
- Review code theo checklist sau:
  - Kiểm tra convention
  - Kiểm tra logic nghiệp vụ
  - Kiểm tra bảo mật
  - Kiểm tra hiệu năng

Ví dụ về quy trình:
```
1. Developer tạo branch từ develop với format: feature/ten-tinh-nang
2. Developer push code và tạo PR
3. Gán ít nhất 1 reviewer
4. Sửa code theo góp ý của reviewer
5. Merge khi được approve
```

## 2. Tổ chức Clean Architecture

- Tổ chức project theo 4 layer chính:
  - **Domain Layer**: Chứa entities, domain exceptions, interfaces
  - **Application Layer**: Chứa use cases, services, interfaces cho infrastructure
  - **Infrastructure Layer**: Chứa repositories, external services, database
  - **API Layer**: Chứa controllers, middleware, filters

- Không đặt business logic trong controller
- Tuân thủ dependency rule: các layer bên trong không được phụ thuộc vào layer bên ngoài

Ví dụ về cấu trúc thư mục:
```
Solution/
├── Domain/
│   ├── Entities/
│   ├── Exceptions/
│   └── Interfaces/
├── Application/
│   ├── DTOs/
│   ├── Interfaces/
│   ├── Mappings/
│   └── Services/
├── Infrastructure/
│   ├── Persistence/
│   ├── Services/
│   └── Configurations/
└── API/
    ├── Controllers/
    ├── Filters/
    └── Middlewares/
```

## 3. Code dễ đọc, dễ hiểu

- Đặt tên class, method có ý nghĩa rõ ràng
- Áp dụng quy tắc đặt tên:
  - **PascalCase**: Class, Method, Property
  - **camelCase**: Biến, tham số
- Không viết tắt khó hiểu
- Comment khi cần thiết, không lạm dụng comment để giải thích code kém

Ví dụ về cách đặt tên tốt:
```csharp
// Tốt
public class UserService
{
    public async Task<User> GetUserByIdAsync(int userId)
    {
        // Code xử lý
    }
}

// Không tốt
public class usrSvc
{
    public async Task<User> getUsrById(int id)
    {
        // Code xử lý
    }
}
```

## 4. Tuân thủ Coding Style

- **PascalCase** cho class, method, property, enum
- **camelCase** cho biến cục bộ, tham số
- Tuân thủ Microsoft .NET Coding Guidelines
- Không hardcode giá trị, sử dụng configuration
- Khoảng cách và thụt đầu dòng nhất quán

Ví dụ về coding style:
```csharp
public class UserController : ControllerBase
{
    private readonly IUserService _userService;
    
    public UserController(IUserService userService)
    {
        _userService = userService;
    }
    
    [HttpGet("{id}")]
    public async Task<ActionResult<UserDto>> GetUserById(int id)
    {
        var user = await _userService.GetUserByIdAsync(id);
        
        if (user == null)
        {
            return NotFound();
        }
        
        return Ok(user);
    }
}
```

## 5. Kiểm tra Null & Exception

- Luôn kiểm tra null trước khi sử dụng object
- Sử dụng `?.` và `??` operator khi cần thiết
- Không dùng `catch (Exception)` chung chung, bắt lỗi cụ thể
- Không "swallow" exception (bắt lỗi mà không xử lý)
- Log exception đầy đủ

Ví dụ về xử lý exception:
```csharp
public async Task<User> GetUserByIdAsync(int userId)
{
    try
    {
        var user = await _userRepository.GetByIdAsync(userId);
        
        if (user == null)
        {
            throw new NotFoundException($"User with ID {userId} not found");
        }
        
        return user;
    }
    catch (DbException ex)
    {
        _logger.LogError(ex, "Database error when retrieving user {UserId}", userId);
        throw new DatabaseException("Error retrieving user from database", ex);
    }
}
```

## 6. Tuân thủ SOLID Principles

- **S (Single Responsibility)**: Mỗi class chỉ có một nhiệm vụ duy nhất
- **O (Open/Closed)**: Thiết kế để mở rộng nhưng không sửa đổi code hiện tại
- **L (Liskov Substitution)**: Class con có thể thay thế class cha mà không gây lỗi
- **I (Interface Segregation)**: Tách interface thành nhiều interface nhỏ chuyên biệt
- **D (Dependency Inversion)**: Inject dependency qua interface

Ví dụ về Dependency Inversion:
```csharp
// Tốt
public class UserService
{
    private readonly IUserRepository _userRepository;
    
    public UserService(IUserRepository userRepository)
    {
        _userRepository = userRepository;
    }
}

// Không tốt
public class UserService
{
    private readonly SqlUserRepository _userRepository;
    
    public UserService()
    {
        _userRepository = new SqlUserRepository();
    }
}
```

## 7. Viết Unit Test & Integration Test

- Code phải có unit test hoặc integration test tương ứng
- Khi fix bug, thêm test case để tránh bug tái diễn
- Test phải rõ ràng và bao quát các trường hợp quan trọng
- Sử dụng mocking framework (Moq) để test độc lập
- Chạy test tự động trước khi merge vào branch chính

Ví dụ về unit test:
```csharp
[Fact]
public async Task GetUserById_WhenUserExists_ReturnsUser()
{
    // Arrange
    var userId = 1;
    var expectedUser = new User { Id = userId, Name = "Test User" };
    
    _mockUserRepository
        .Setup(repo => repo.GetByIdAsync(userId))
        .ReturnsAsync(expectedUser);
    
    // Act
    var result = await _userService.GetUserByIdAsync(userId);
    
    // Assert
    Assert.NotNull(result);
    Assert.Equal(userId, result.Id);
    Assert.Equal("Test User", result.Name);
}
```

## 8. Performance & Security

- Chỉ select những dữ liệu cần thiết, không dùng `SELECT *`
- Luôn sử dụng parameterized query để tránh SQL Injection
- Không hardcode API Key, mật khẩu trong code
- Mã hóa dữ liệu nhạy cảm (mật khẩu, token)
- Sử dụng caching khi cần thiết

Ví dụ về parameterized query:
```csharp
// Tốt
var user = await _context.Users
    .Where(u => u.Id == userId)
    .FirstOrDefaultAsync();

// Không tốt (nguy cơ SQL Injection)
var query = $"SELECT * FROM Users WHERE Id = {userId}";
var user = await _context.Database.ExecuteSqlRawAsync(query);
```

## 9. Tối ưu hóa API & Database

- Giảm thiểu số lượng request API
- Sử dụng async/await để tránh block thread
- Tránh N+1 Query bằng cách sử dụng Include() và eager loading
- Sử dụng pagination cho các API trả về nhiều dữ liệu
- Sử dụng index cho database và caching cho các query phổ biến

Ví dụ về eager loading:
```csharp
// Tốt (tránh N+1 query)
var users = await _context.Users
    .Include(u => u.Role)
    .Include(u => u.Department)
    .ToListAsync();

// Không tốt (N+1 query)
var users = await _context.Users.ToListAsync();
foreach (var user in users)
{
    await _context.Entry(user).Reference(u => u.Role).LoadAsync();
    await _context.Entry(user).Reference(u => u.Department).LoadAsync();
}
```

## 10. Không vi phạm nguyên tắc Clean Code

- Tuân thủ nguyên tắc DRY (Don't Repeat Yourself)
- Tuân thủ nguyên tắc KISS (Keep It Simple, Stupid)
- Không để TODO comment kéo dài trong code
- Giữ method ngắn gọn, không quá 30 dòng

Ví dụ về DRY:
```csharp
// Tốt (tách thành method riêng)
public async Task<UserDto> GetUserAsync(int userId)
{
    var user = await _userRepository.GetByIdAsync(userId);
    return MapToDto(user);
}

public async Task<UserDto> GetUserByEmailAsync(string email)
{
    var user = await _userRepository.GetByEmailAsync(email);
    return MapToDto(user);
}

private UserDto MapToDto(User user)
{
    return new UserDto
    {
        Id = user.Id,
        Name = user.Name,
        Email = user.Email
    };
}

// Không tốt (duplicate code)
public async Task<UserDto> GetUserAsync(int userId)
{
    var user = await _userRepository.GetByIdAsync(userId);
    return new UserDto
    {
        Id = user.Id,
        Name = user.Name,
        Email = user.Email
    };
}

public async Task<UserDto> GetUserByEmailAsync(string email)
{
    var user = await _userRepository.GetByEmailAsync(email);
    return new UserDto
    {
        Id = user.Id,
        Name = user.Name,
        Email = user.Email
    };
}
```

## 11. Quản lý Dependency hợp lý

- Hạn chế sử dụng static class
- Sử dụng Dependency Injection (DI) container của ASP.NET Core
- Đăng ký service trong Startup.cs hoặc Program.cs
- Không lạm dụng Global Variable

Ví dụ về đăng ký DI:
```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Repository
    services.AddScoped<IUserRepository, UserRepository>();
    services.AddScoped<IProductRepository, ProductRepository>();
    
    // Services
    services.AddScoped<IUserService, UserService>();
    services.AddScoped<IProductService, ProductService>();
    
    // Other
    services.AddSingleton<ICacheService, RedisCacheService>();
}
```

## 12. Quy tắc đặt tên biến và hàm

- Tên biến phải thể hiện rõ chức năng
- Biến boolean nên bắt đầu bằng is, has, can, should
- Không dùng tên biến chung chung (temp, data, obj)
- Method tên động từ, thể hiện hành động

Ví dụ về đặt tên:
```csharp
// Tốt
bool isActive = user.Status == UserStatus.Active;
bool hasPermission = user.Role.Permissions.Contains(permission);
List<Product> featuredProducts = GetFeaturedProducts();

// Không tốt
bool flag = user.Status == UserStatus.Active;
bool check = user.Role.Permissions.Contains(permission);
var data = GetFeaturedProducts();
```

## 13. Comment & Documentation

- Sử dụng XML Documentation cho public method
- Comment chỉ khi thực sự cần thiết
- Comment giải thích "tại sao" chứ không phải "làm gì"
- Không dùng comment để giải thích code xấu

Ví dụ về XML Documentation:
```csharp
/// <summary>
/// Lấy thông tin người dùng theo ID
/// </summary>
/// <param name="userId">ID của người dùng</param>
/// <returns>Thông tin người dùng hoặc null nếu không tồn tại</returns>
/// <exception cref="NotFoundException">Khi không tìm thấy người dùng</exception>
public async Task<UserDto> GetUserByIdAsync(int userId)
{
    // Implementation
}
```

## 14. Refactor Code

- Refactor code khi thấy code trùng lặp hoặc phức tạp
- Tách method dài thành nhiều method nhỏ hơn
- Sử dụng extension method để mở rộng chức năng
- Refactor cần có test để đảm bảo không ảnh hưởng chức năng

Ví dụ về refactor:
```csharp
// Trước khi refactor
public async Task<IEnumerable<ProductDto>> GetProductsAsync(int categoryId)
{
    var products = await _context.Products
        .Where(p => p.CategoryId == categoryId)
        .Where(p => p.IsActive)
        .OrderByDescending(p => p.CreatedAt)
        .Take(20)
        .ToListAsync();
        
    var result = new List<ProductDto>();
    foreach (var product in products)
    {
        result.Add(new ProductDto
        {
            Id = product.Id,
            Name = product.Name,
            Price = product.Price,
            ImageUrl = product.ImageUrl
        });
    }
    
    return result;
}

// Sau khi refactor
public async Task<IEnumerable<ProductDto>> GetProductsAsync(int categoryId)
{
    var products = await GetActiveProductsByCategoryAsync(categoryId);
    return products.Select(MapToDto);
}

private async Task<List<Product>> GetActiveProductsByCategoryAsync(int categoryId)
{
    return await _context.Products
        .Where(p => p.CategoryId == categoryId && p.IsActive)
        .OrderByDescending(p => p.CreatedAt)
        .Take(20)
        .ToListAsync();
}

private ProductDto MapToDto(Product product)
{
    return new ProductDto
    {
        Id = product.Id,
        Name = product.Name,
        Price = product.Price,
        ImageUrl = product.ImageUrl
    };
}
```

## 15. Quản lý Logging

- Sử dụng ILogger thay vì Console.WriteLine()
- Log với các level phù hợp (Debug, Info, Warning, Error)
- Không log dữ liệu nhạy cảm (password, token)
- Log đầy đủ thông tin cho Exception

Ví dụ về logging:
```csharp
public async Task<User> GetUserByIdAsync(int userId)
{
    _logger.LogInformation("Getting user with ID {UserId}", userId);
    
    try
    {
        var user = await _userRepository.GetByIdAsync(userId);
        
        if (user == null)
        {
            _logger.LogWarning("User with ID {UserId} not found", userId);
            throw new NotFoundException($"User with ID {userId} not found");
        }
        
        return user;
    }
    catch (Exception ex) when (ex is not NotFoundException)
    {
        _logger.LogError(ex, "Error getting user with ID {UserId}", userId);
        throw;
    }
}
```

## 16. Quản lý Configuration

- Sử dụng appsettings.json, user secrets hoặc biến môi trường
- Phân tách config theo môi trường (Development, Staging, Production)
- Không push thông tin nhạy cảm lên Git
- Sử dụng strongly-typed configuration

Ví dụ về strongly-typed configuration:
```csharp
// Định nghĩa class config
public class DatabaseSettings
{
    public string ConnectionString { get; set; }
    public int CommandTimeout { get; set; }
    public bool EnableRetry { get; set; }
}

// Đăng ký trong Program.cs
services.Configure<DatabaseSettings>(
    Configuration.GetSection("DatabaseSettings"));
    
// Sử dụng trong service
public class UserRepository : IUserRepository
{
    private readonly DatabaseSettings _dbSettings;
    
    public UserRepository(IOptions<DatabaseSettings> dbSettings)
    {
        _dbSettings = dbSettings.Value;
    }
}
```

## 17. Quản lý Transaction

- Sử dụng Unit of Work pattern
- Đảm bảo transaction được rollback khi có lỗi
- Tránh transaction lồng nhau hoặc quá dài

Ví dụ về Unit of Work:
```csharp
public async Task CreateOrderAsync(OrderDto orderDto)
{
    using var transaction = await _context.Database.BeginTransactionAsync();
    
    try
    {
        // Tạo order
        var order = new Order
        {
            UserId = orderDto.UserId,
            TotalAmount = orderDto.TotalAmount,
            CreatedAt = DateTime.UtcNow
        };
        
        _context.Orders.Add(order);
        await _context.SaveChangesAsync();
        
        // Tạo order items
        foreach (var item in orderDto.Items)
        {
            var orderItem = new OrderItem
            {
                OrderId = order.Id,
                ProductId = item.ProductId,
                Quantity = item.Quantity,
                Price = item.Price
            };
            
            _context.OrderItems.Add(orderItem);
        }
        
        await _context.SaveChangesAsync();
        await transaction.CommitAsync();
    }
    catch
    {
        await transaction.RollbackAsync();
        throw;
    }
}
```

## 18. Làm sạch mã nguồn

- Xóa code thừa, file không cần thiết
- Không push code debug tạm thời
- Xóa các commented code không còn sử dụng
- Sử dụng công cụ như ReSharper, SonarQube để phát hiện code smell

Ví dụ về clean code:
```csharp
// Không tốt - có code thừa và debug code
public async Task<User> GetUserByIdAsync(int userId)
{
    // Cách cũ
    // var user = _context.Users.Find(userId);
    
    Console.WriteLine("Debug: Getting user " + userId);
    
    var user = await _context.Users
        .FirstOrDefaultAsync(u => u.Id == userId);
        
    // TODO: Remove this later
    Debug.WriteLine("User found: " + (user != null));
    
    return user;
}

// Tốt - đã clean
public async Task<User> GetUserByIdAsync(int userId)
{
    return await _context.Users
        .FirstOrDefaultAsync(u => u.Id == userId);
}
```

## 19. Quản lý phiên bản API

- Sử dụng versioning cho API (v1, v2)
- Không thay đổi API đã public mà không có backward compatibility
- Sử dụng swagger để tài liệu hóa API
- Cung cấp thông báo deprecation trước khi xóa API cũ

Ví dụ về API versioning:
```csharp
// Startup.cs
services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
});

// Controller
[ApiController]
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/users")]
public class UsersV1Controller : ControllerBase
{
    // API v1
}

[ApiController]
[ApiVersion("2.0")]
[Route("api/v{version:apiVersion}/users")]
public class UsersV2Controller : ControllerBase
{
    // API v2 với cải tiến mới
}
```

## 20. Sử dụng Git hợp lý

- Viết commit message rõ ràng, tuân thủ format thống nhất
- Không commit trực tiếp vào branch chính (main/develop)
- Sử dụng Git Flow hoặc Github Flow
- Resolve conflict trước khi merge

Ví dụ về commit message:
```
feat: Thêm chức năng đăng nhập bằng Google

- Thêm GoogleAuthProvider
- Cập nhật UserService để xử lý Google OAuth
- Thêm unit test cho chức năng mới
```
