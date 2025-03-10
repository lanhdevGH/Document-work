# Quy định về Code Convention cho dự án C# ASP.NET (Clean Architecture)

## 1. Thụt đầu dòng (Indentation)
- Sử dụng **4 khoảng trắng** cho mỗi cấp thụt đầu dòng, không sử dụng tab.
- Điều này đảm bảo mã nguồn hiển thị nhất quán trên các trình soạn thảo khác nhau.

```csharp
public void SomeMethod()
{
    if (condition)
    {
        DoSomething();
    }
}
```

## 2. Kiểu đặt dấu ngoặc nhọn (Bracing Style)
- Sử dụng phong cách Allman (dấu { xuống dòng mới).
- Dấu ngoặc mở luôn nằm trên một dòng riêng biệt.

```csharp
public class Example
{
    public void SomeMethod()
    {
        if (someCondition)
        {
            // code
        }
    }
}
```

## 3. Quy tắc đặt tên (Naming Conventions)
- **PascalCase** cho class, property và method công khai
- **camelCase** cho biến cục bộ và tham số
- **_camelCase** cho các thành viên private

```csharp
public class UserService
{
    private readonly IUserRepository _userRepository;
    
    public UserService(IUserRepository userRepository)
    {
        _userRepository = userRepository;
    }
    
    public User GetUserById(int userId)
    {
        var currentUser = _userRepository.Find(userId);
        return currentUser;
    }
}
```

## 4. Độ dài dòng (Line Length)
- Tối đa **120 ký tự** trên một dòng.
- Ngắt dòng nếu vượt quá giới hạn này, đảm bảo tính dễ đọc.

```csharp
// Ngắt dòng khi dài quá 120 ký tự
public void SomeMethodWithManyParameters(string parameter1, string parameter2, string parameter3, 
    string parameter4, string parameter5)
{
    // code
}
```

## 5. Bình luận trong mã (Commenting)
- Tập trung giải thích "tại sao" làm điều gì đó, không chỉ viết lại những gì mã đang làm.
- Sử dụng XML doc cho tất cả các method công khai.

```csharp
/// <summary>
/// Xác thực thông tin người dùng dựa trên token đã cung cấp
/// </summary>
/// <param name="token">JWT token từ người dùng</param>
/// <returns>Thông tin người dùng đã xác thực</returns>
/// <exception cref="UnauthorizedException">Khi token không hợp lệ</exception>
public User AuthenticateUser(string token)
{
    // Giải mã token sử dụng thuật toán HS256
    // vì yêu cầu bảo mật cao từ khách hàng
    var decodedToken = _jwtService.Decode(token);
    
    return _userRepository.Find(decodedToken.UserId);
}
```

## 6. Khoảng trắng (Whitespace)
- Để một dòng trống giữa các method.
- Sử dụng khoảng trắng xung quanh các toán tử.
- Không để khoảng trắng thừa ở cuối dòng.

```csharp
public int Add(int a, int b)
{
    return a + b;
}

public int Subtract(int a, int b)
{
    return a - b;
}
```

## 7. Cấu trúc tệp mã (Code File Structure)
- Mỗi file chỉ chứa một class.
- Tên file phải khớp với tên class.
- Thứ tự cấu trúc: using → namespace → class.

```csharp
// UserService.cs
using System;
using System.Threading.Tasks;
using Project.Domain.Entities;

namespace Project.Application.Services
{
    public class UserService
    {
        // code
    }
}
```

## 8. Tổ chức mã (Code Organization)
- Tuân thủ cấu trúc Clean Architecture với các tầng rõ ràng:
  - **Domain**: Entities, Value Objects, Enums
  - **Application**: Services, Interfaces, DTOs, Validators
  - **Infrastructure**: Repositories, External Services, Database
  - **Presentation**: Controllers, Middleware, Filters

```
Solution/
  ├─ Domain/
  │   ├─ Entities/
  │   ├─ ValueObjects/
  │   └─ Enums/
  ├─ Application/
  │   ├─ Services/
  │   ├─ Interfaces/
  │   ├─ DTOs/
  │   └─ Validators/
  ├─ Infrastructure/
  │   ├─ Persistence/
  │   │   ├─ Repositories/
  │   │   └─ DbContext/
  │   └─ ExternalServices/
  └─ API/
      ├─ Controllers/
      ├─ Middleware/
      └─ Filters/
```

## 9. Xử lý chuỗi (String Handling)
- Sử dụng string interpolation `$"{variable}"` thay vì nối chuỗi.
- Với chuỗi lặp lại, lưu vào biến const.

```csharp
// Tốt
private const string ErrorMessagePrefix = "Lỗi xảy ra: ";
var message = $"{ErrorMessagePrefix}{exception.Message}";

// Không tốt
var message = "Lỗi xảy ra: " + exception.Message;
```

## 10. Xử lý ngoại lệ (Exception Handling)
- Chỉ bắt các ngoại lệ cụ thể, tránh bắt Exception chung.
- Thông điệp lỗi phải rõ ràng khi throw exception.

```csharp
try
{
    var user = await _userRepository.GetByIdAsync(userId);
    if (user == null)
    {
        throw new NotFoundException($"Không tìm thấy người dùng với ID: {userId}");
    }
}
catch (DbException dbEx)
{
    _logger.LogError(dbEx, "Lỗi kết nối database khi tìm người dùng");
    throw new ServiceException("Không thể kết nối với cơ sở dữ liệu", dbEx);
}
```

## 11. Kiểm tra giá trị null (Null Checking)
- Sử dụng toán tử `?.` (null conditional) và `??` (null coalescing) để xử lý giá trị null.

```csharp
// Toán tử ?. - Chỉ gọi GetName nếu user không null
string name = user?.GetName();

// Toán tử ?? - Sử dụng giá trị mặc định nếu null
string displayName = user?.GetName() ?? "Khách";
```

## 12. File Encoding và Line Endings
- Sử dụng **UTF-8** cho mã hóa file.
- Sử dụng **CRLF** (Windows) cho việc kết thúc dòng.
- Thiết lập trong .editorconfig để đảm bảo nhất quán.

```
# .editorconfig
[*.cs]
charset = utf-8
end_of_line = crlf
```

## 13. Tránh số và chuỗi ma thuật
- Sử dụng const hoặc enum thay vì số hoặc chuỗi cứng trong mã.

```csharp
// Tốt
private const int MaxRetryAttempts = 3;
if (retryCount >= MaxRetryAttempts)

// Không tốt
if (retryCount >= 3)
```

## 14. Thành phần dạng biểu thức
- Sử dụng cú pháp `=>` cho method và property ngắn.

```csharp
// Expression-bodied method
public string GetFullName() => $"{FirstName} {LastName}";

// Expression-bodied property
public bool IsValid => Age >= 18 && !string.IsNullOrEmpty(Name);
```

## 15. Readonly fields
- Sử dụng `readonly` cho các biến chỉ được gán giá trị một lần (thường tại constructor).

```csharp
public class UserService
{
    private readonly IUserRepository _userRepository;
    private readonly ILogger<UserService> _logger;
    
    public UserService(IUserRepository userRepository, ILogger<UserService> logger)
    {
        _userRepository = userRepository;
        _logger = logger;
    }
}
```

## 16. Dependency Injection
- Tiêm các phụ thuộc qua constructor.
- Tránh sử dụng service locator pattern.

```csharp
// Tốt - Constructor injection
public class UserService
{
    private readonly IUserRepository _userRepository;
    
    public UserService(IUserRepository userRepository)
    {
        _userRepository = userRepository;
    }
}

// Không tốt - Service locator
public class UserService
{
    private IUserRepository _userRepository;
    
    public void SomeMethod()
    {
        _userRepository = ServiceLocator.Resolve<IUserRepository>();
    }
}
```

## 17. Quy tắc sử dụng var
- Sử dụng `var` khi kiểu dữ liệu có thể suy ra rõ ràng từ ngữ cảnh.

```csharp
// Tốt - Kiểu rõ ràng từ kết quả phương thức
var user = GetUser(id);

// Tốt - Kiểu rõ ràng từ khởi tạo
var users = new List<User>();

// Không nên - Không rõ kiểu dữ liệu trả về
var result = GetSomething();
```

## 18. Async/Await Naming
- Thêm hậu tố `Async` cho các method bất đồng bộ.

```csharp
// Phương thức bất đồng bộ
public async Task<User> GetUserByIdAsync(int userId)
{
    return await _userRepository.GetByIdAsync(userId);
}

// Gọi phương thức bất đồng bộ
public async Task<IActionResult> GetUserDetails(int id)
{
    var user = await _userService.GetUserByIdAsync(id);
    return Ok(user);
}
```

## 19. Using directives
- Sắp xếp theo thứ tự: System → Third-party → Project.
- Nhóm theo namespace gốc.

```csharp
// System namespaces
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

// Third-party libraries
using Microsoft.EntityFrameworkCore;
using Newtonsoft.Json;

// Project namespaces
using Project.Domain.Entities;
using Project.Domain.Exceptions;
```

## 20. Logging
- Sử dụng cấp độ logging phù hợp: Information, Debug, Error.
- Cung cấp thông tin hữu ích và ngữ cảnh trong log.

```csharp
// Debug - Thông tin chi tiết hữu ích khi phát triển
_logger.LogDebug("Bắt đầu xử lý đơn hàng {OrderId}", orderId);

// Information - Thông tin quan trọng về hoạt động
_logger.LogInformation("Đơn hàng {OrderId} đã được xử lý thành công", orderId);

// Error - Lỗi xảy ra cần xử lý
_logger.LogError(ex, "Lỗi khi xử lý đơn hàng {OrderId}", orderId);
```

## 21. LINQ Usage
- Ưu tiên query syntax cho truy vấn phức tạp.
- Sử dụng method syntax cho các truy vấn đơn giản.

```csharp
// Query syntax cho truy vấn phức tạp
var results = from u in users
              join o in orders on u.Id equals o.UserId
              where u.IsActive && o.Status == OrderStatus.Completed
              select new { u.Name, o.OrderDate, o.Total };

// Method syntax cho truy vấn đơn giản
var activeUsers = users.Where(u => u.IsActive).ToList();
```

## 22. Unit Tests
- Đặt tên test rõ ràng theo mẫu: [MethodName_StateUnderTest_ExpectedBehavior].

```csharp
[Fact]
public async Task GetUserById_UserExists_ReturnsUser()
{
    // Arrange
    var userId = 1;
    var mockUser = new User { Id = userId, Name = "Test User" };
    _mockRepository.Setup(r => r.GetByIdAsync(userId)).ReturnsAsync(mockUser);
    
    // Act
    var result = await _userService.GetUserByIdAsync(userId);
    
    // Assert
    Assert.NotNull(result);
    Assert.Equal(userId, result.Id);
}

[Fact]
public async Task GetUserById_UserDoesNotExist_ThrowsNotFoundException()
{
    // Arrange
    var userId = 999;
    _mockRepository.Setup(r => r.GetByIdAsync(userId)).ReturnsAsync((User)null);
    
    // Act & Assert
    await Assert.ThrowsAsync<NotFoundException>(() => 
        _userService.GetUserByIdAsync(userId));
}
```

## 23. Access Modifiers Order
- Sắp xếp theo thứ tự: public → protected → internal → private.

```csharp
public class Example
{
    // Public members
    public void PublicMethod() { }
    
    // Protected members
    protected void ProtectedMethod() { }
    
    // Internal members
    internal void InternalMethod() { }
    
    // Private members
    private void PrivateMethod() { }
}
```

## 24. Khởi tạo đối tượng
- Ưu tiên object initializer khi khởi tạo đối tượng.

```csharp
// Tốt - Sử dụng object initializer
var user = new User
{
    Id = 1,
    FirstName = "John",
    LastName = "Doe",
    Email = "john.doe@example.com"
};

// Không tốt
var user = new User();
user.Id = 1;
user.FirstName = "John";
user.LastName = "Doe";
user.Email = "john.doe@example.com";
```

## 25. Tự động format code
- Sử dụng StyleCop, dotnet format và .editorconfig để đảm bảo nhất quán.
- Thiết lập file .editorconfig trong dự án.

```
# .editorconfig
root = true

[*.cs]
indent_style = space
indent_size = 4
charset = utf-8
end_of_line = crlf
insert_final_newline = true
trim_trailing_whitespace = true
```

## 26. Điều kiện và vòng lặp
- Luôn sử dụng dấu ngoặc nhọn cho blocks điều kiện và vòng lặp, ngay cả khi chỉ có một dòng.

```csharp
// Tốt
if (condition)
{
    DoSomething();
}

// Không tốt
if (condition)
    DoSomething();
```

## 27. Sử dụng đối tượng bất biến
- Ưu tiên thiết kế bất biến (immutable) cho Value Objects.
- Khởi tạo giá trị qua constructor và không cho phép thay đổi sau khi tạo.

```csharp
public class Money
{
    public decimal Amount { get; }
    public string Currency { get; }
    
    public Money(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency;
    }
    
    // Tạo đối tượng mới thay vì thay đổi giá trị hiện tại
    public Money Add(Money other)
    {
        if (other.Currency != Currency)
        {
            throw new InvalidOperationException("Không thể cộng tiền với đơn vị khác nhau");
        }
        
        return new Money(Amount + other.Amount, Currency);
    }
}
```

## 28. Sử dụng nameof
- Dùng `nameof` thay vì chuỗi cứng cho tên biến/method.

```csharp
// Tốt
public void ProcessOrder(Order order)
{
    if (order == null)
    {
        throw new ArgumentNullException(nameof(order));
    }
}

// Không tốt
public void ProcessOrder(Order order)
{
    if (order == null)
    {
        throw new ArgumentNullException("order");
    }
}
```

## 29. Chuyển đổi kiểu
- Ưu tiên sử dụng `as` + kiểm tra null thay vì cast trực tiếp.

```csharp
// Tốt - Sử dụng as + kiểm tra null
var customer = entity as Customer;
if (customer != null)
{
    // Xử lý customer
}

// Không tốt - Có thể gây ra exception
var customer = (Customer)entity;
```

## 30. Grouping Related Variables
- Nhóm các biến liên quan gần nhau.

```csharp
// Các biến liên quan đến user
var userId = GetUserId();
var userName = GetUserName();
var userRole = GetUserRole();

// Các biến liên quan đến order
var orderId = GetOrderId();
var orderDate = GetOrderDate();
var orderStatus = GetOrderStatus();
```

## 31. Tránh Nested Conditionals
- Sử dụng early returns để giảm độ phức tạp thay vì các điều kiện lồng nhau.

```csharp
// Tốt - Sử dụng early returns
public void ProcessOrder(Order order)
{
    if (order == null)
    {
        throw new ArgumentNullException(nameof(order));
    }
    
    if (!order.IsValid)
    {
        throw new InvalidOperationException("Đơn hàng không hợp lệ");
    }
    
    if (order.IsCancelled)
    {
        return;
    }
    
    // Xử lý đơn hàng
}

// Không tốt - Nested conditionals
public void ProcessOrder(Order order)
{
    if (order != null)
    {
        if (order.IsValid)
        {
            if (!order.IsCancelled)
            {
                // Xử lý đơn hàng
            }
        }
        else
        {
            throw new InvalidOperationException("Đơn hàng không hợp lệ");
        }
    }
    else
    {
        throw new ArgumentNullException(nameof(order));
    }
}
```

## 32. Method Parameters
- Tránh quá nhiều tham số, sử dụng object khi cần nhiều tham số.

```csharp
// Tốt - Sử dụng object để nhóm các tham số
public void CreateUser(UserCreateRequest request)
{
    // Xử lý
}

public class UserCreateRequest
{
    public string Username { get; set; }
    public string Email { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string PhoneNumber { get; set; }
}

// Không tốt - Quá nhiều tham số
public void CreateUser(string username, string email, string firstName, 
    string lastName, string phoneNumber)
{
    // Xử lý
}
```

## 33. Error Handling Pattern
- Sử dụng Result Pattern thay vì exception khi phù hợp.

```csharp
public class Result<T>
{
    public T Data { get; }
    public bool IsSuccess { get; }
    public string Error { get; }
    
    private Result(T data, bool isSuccess, string error = null)
    {
        Data = data;
        IsSuccess = isSuccess;
        Error = error;
    }
    
    public static Result<T> Success(T data) => new Result<T>(data, true);
    public static Result<T> Failure(string error) => new Result<T>(default, false, error);
}

// Sử dụng
public Result<User> GetUser(int id)
{
    var user = _repository.GetById(id);
    if (user == null)
    {
        return Result<User>.Failure($"Không tìm thấy người dùng với ID: {id}");
    }
    
    return Result<User>.Success(user);
}

// Xử lý kết quả
var result = _userService.GetUser(userId);
if (result.IsSuccess)
{
    var user = result.Data;
    // Xử lý user
}
else
{
    _logger.LogWarning(result.Error);
    // Xử lý lỗi
}
```

## 34. Sử dụng C# patterns mới
- Tận dụng các tính năng hiện đại như pattern matching, using declarations.

```csharp
// Pattern matching
public string GetDiscountType(Customer customer) => customer switch
{
    PremiumCustomer p when p.YearsOfMembership > 5 => "VIP",
    PremiumCustomer _ => "Premium",
    RegularCustomer r when r.PurchaseCount > 10 => "Loyal",
    RegularCustomer _ => "Regular",
    _ => "New"
};

// Using declarations
public async Task<string> ReadFileAsync(string path)
{
    using var fileStream = new FileStream(path, FileMode.Open);
    using var reader = new StreamReader(fileStream);
    return await reader.ReadToEndAsync();
}
```

## 35. Extension Methods
- Tổ chức các extension method trong các class riêng biệt.
- Đặt tên class kết thúc bằng "Extensions".

```csharp
// StringExtensions.cs
public static class StringExtensions
{
    public static bool IsNullOrEmpty(this string value)
    {
        return string.IsNullOrEmpty(value);
    }
    
    public static bool IsValidEmail(this string email)
    {
        // Kiểm tra email hợp lệ
        return Regex.IsMatch(email, @"^[^@\s]+@[^@\s]+\.[^@\s]+$");
    }
}

// DateTimeExtensions.cs
public static class DateTimeExtensions
{
    public static bool IsWeekend(this DateTime date)
    {
        return date.DayOfWeek == DayOfWeek.Saturday || date.DayOfWeek == DayOfWeek.Sunday;
    }
}
```

## 36. Tránh comment vô thời hạn
- Không sử dụng TODO, HACK, FIXME - sử dụng task management tool thay thế.

```csharp
// Thay vì viết 
// TODO: Cần cải thiện hiệu suất ở đây
// HACK: Tạm thời giải quyết lỗi X
// FIXME: Sửa lỗi này trước phiên bản tiếp theo

// Tạo task trong hệ thống quản lý công việc và tham chiếu ID
// VD: JIRA-123: Cải thiện hiệu suất xử lý đơn hàng
```

## 37. Sử dụng Attributes phù hợp
- Đánh dấu API endpoints rõ ràng với attributes.

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    [HttpGet("{id}")]
    [ProducesResponseType(typeof(UserDto), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> GetById(int id)
    {
        // code
    }
    
    [HttpPost]
    [Authorize(Roles = "Admin")]
    [ProducesResponseType(typeof(UserDto), StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<IActionResult> Create([FromBody] CreateUserRequest request)
    {
        // code
    }
}
```

## 38. Interface Naming
- Sử dụng tiền tố "I" và đặt tên rõ nghĩa.
- Tên interface nên mô tả khả năng chứ không phải triển khai.

```csharp
// Tốt
public interface IUserRepository
{
    Task<User> GetByIdAsync(int id);
    Task<IEnumerable<User>> GetAllAsync();
    Task AddAsync(User user);
}

// Tốt
public interface INotificationSender
{
    Task SendAsync(Notification notification);
}

// Không tốt - không mô tả khả năng
public interface IUserClass
{
    // code
}
```

## 39. Constants vs Static Readonly
- Sử dụng `const` cho giá trị compile-time.
- Sử dụng `static readonly` cho giá trị runtime.

```csharp
// Compile-time constants
public class PaymentSettings
{
    public const int MaxRetryAttempts = 3;
    public const string PaymentProviderName = "PayPal";
}

// Runtime constants
public class ApiSettings
{
    public static readonly TimeSpan DefaultTimeout = TimeSpan.FromSeconds(30);
    public static readonly Uri ApiEndpoint = new Uri("https://api.example.com");
}
```

## 40. Options Pattern
- Sử dụng Options Pattern cho cấu hình.

```csharp
// Define options class
public class EmailOptions
{
    public const string SectionName = "Email";
    
    public string SmtpServer { get; set; }
    public int Port { get; set; }
    public string Username { get; set; }
    public string Password { get; set; }
    public bool EnableSsl { get; set; }
}

// Register in Startup
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<EmailOptions>(Configuration.GetSection(EmailOptions.SectionName));
}

// Inject in service
public class EmailService
{
    private readonly EmailOptions _options;
    
    public EmailService(IOptions<EmailOptions> options)
    {
        _options = options.Value;
    }
    
    public async Task SendEmailAsync(string to, string subject, string body)
    {
        using var client = new SmtpClient(_options.SmtpServer, _options.Port)
        {
            EnableSsl = _options.EnableSsl,
            Credentials = new NetworkCredential(_options.Username, _options.Password)
        };
        
        // Gửi email
    }
}
```
