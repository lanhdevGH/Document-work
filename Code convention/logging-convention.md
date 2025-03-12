# Code Convention cho Backend C# ASP.NET - Clean Architecture

## Nguyên tắc chung về Logging

### Sử dụng ILogger thay vì tự viết
- Luôn sử dụng `ILogger` của Microsoft thay vì các thư viện tự phát triển
- Lợi ích: Tận dụng các tính năng có sẵn, dễ bảo trì, hỗ trợ tốt trong hệ sinh thái .NET

```csharp
// ✅ Đúng
public class UserService
{
    private readonly ILogger<UserService> _logger;
    
    public UserService(ILogger<UserService> logger)
    {
        _logger = logger;
    }
}

// ❌ Sai
public class CustomLogger 
{
    public void WriteLog(string message) { /* ... */ }
}
```

### Phân tầng Logging
- Không sử dụng log trong Domain Layer
- Chỉ sử dụng trong Application và Infrastructure Layer
- Domain Layer nên thuần túy logic nghiệp vụ, không phụ thuộc vào hạ tầng

```csharp
// ✅ Đúng - ApplicationLayer
public class CreateUserCommandHandler
{
    private readonly ILogger<CreateUserCommandHandler> _logger;
    
    public async Task Handle(CreateUserCommand command)
    {
        _logger.LogInformation("Creating user with email: {Email}", command.Email);
        // ...
    }
}

// ❌ Sai - Domain Layer
public class User
{
    private readonly ILogger _logger; // Không nên có logger ở đây
    
    public void UpdateEmail(string email)
    {
        _logger.LogInformation("Email updated"); // Không đặt log trong domain
    }
}
```

### Chọn mức độ log phù hợp
- Sử dụng đúng mức độ log (Debug, Information, Warning, Error, Critical)
- Không lạm dụng mức độ cao cho thông tin thông thường

### Bảo mật thông tin
- Không log thông tin nhạy cảm: mật khẩu, token, dữ liệu cá nhân
- Nếu cần thiết, phải sử dụng mask hoặc hash

```csharp
// ✅ Đúng
_logger.LogInformation("User {UserId} attempted login", user.Id);

// ❌ Sai
_logger.LogInformation("User {Email} attempted login with password {Password}", 
                      user.Email, password);
```

### Định dạng log
- Log có định dạng rõ ràng, dễ đọc
- Phải chứa đủ ngữ cảnh để hiểu được tình huống

## Quy ước đặt tên và cấu trúc

### Quy ước biến logger
- Biến ILogger đặt tên là `_logger`
- Sử dụng generic logger với tên class hiện tại

```csharp
private readonly ILogger<UserController> _logger;
```

### Cấu trúc message
- Bắt đầu với từ khóa mô tả hành động hoặc sự kiện
- Viết rõ ràng, ngắn gọn và đủ thông tin

```csharp
// ✅ Đúng
_logger.LogInformation("User created successfully with ID: {UserId}", user.Id);

// ❌ Sai
_logger.LogInformation("Done"); // Thiếu ngữ cảnh
```

### Structured logging
- Sử dụng tham số hóa, không nối chuỗi thủ công
- Dùng template với các placeholder

```csharp
// ✅ Đúng
_logger.LogInformation("Processing order {OrderId} for customer {CustomerId}", 
                       order.Id, order.CustomerId);

// ❌ Sai
_logger.LogInformation("Processing order " + order.Id + " for customer " + order.CustomerId);
```

## Mức độ log

### LogDebug
- Dùng cho thông tin chi tiết, chỉ hữu ích khi debug
- Chỉ nên hiển thị trong môi trường development

```csharp
_logger.LogDebug("Querying database with parameters: {Params}", queryParams);
```

### LogInformation
- Ghi lại các sự kiện bình thường nhưng quan trọng trong ứng dụng
- Ví dụ: người dùng đăng nhập, tạo tài nguyên, hoàn tất quy trình

```csharp
_logger.LogInformation("User {UserId} successfully logged in", userId);
```

### LogWarning
- Ghi lại tình huống bất thường nhưng chưa gây lỗi
- Ví dụ: retry operation, deprecation warning

```csharp
_logger.LogWarning("API call to payment gateway timed out, retrying ({RetryCount}/3)", retryCount);
```

### LogError
- Ghi lại lỗi cùng với exception
- Cung cấp đủ thông tin để tìm hiểu nguyên nhân

```csharp
try {
    // Code gây lỗi
}
catch (Exception ex) {
    _logger.LogError(ex, "Failed to process payment for order {OrderId}", orderId);
}
```

### LogCritical
- Chỉ dùng cho lỗi nghiêm trọng ảnh hưởng đến toàn bộ hệ thống
- Ví dụ: không thể kết nối database, service không khởi động được

```csharp
_logger.LogCritical(ex, "Application failed to start due to database connection error");
```

## Triển khai trong Clean Architecture

### Dependency Injection
- Inject `_logger` qua constructor trong các lớp ở Application và Infrastructure Layer
- Sử dụng generic logger với tên class

```csharp
public class UserService
{
    private readonly ILogger<UserService> _logger;
    private readonly IUserRepository _userRepository;
    
    public UserService(ILogger<UserService> logger, IUserRepository userRepository)
    {
        _logger = logger;
        _userRepository = userRepository;
    }
}
```

### Domain Entities
- Không đặt logic logging trong Domain Entities
- Domain Layer nên độc lập với concerns của infrastructure

### Cấu hình
- Cấu hình logging trong `Program.cs`
- Thiết lập các providers (Console, File, etc.) và log levels

```csharp
// Program.cs
builder.Logging.ClearProviders();
builder.Logging.AddConsole();
builder.Logging.AddDebug();
builder.Logging.AddFile("logs/app-{Date}.log");
```

## Xử lý Exception

### Ghi log exception
- Luôn truyền exception object vào phương thức log
- Cung cấp thêm thông tin ngữ cảnh

```csharp
try {
    await _userRepository.UpdateAsync(user);
}
catch (DbUpdateException ex) {
    _logger.LogError(ex, "Database error occurred while updating user {UserId}", user.Id);
    throw; // Rethrow để xử lý ở tầng cao hơn nếu cần
}
```

### Tách biệt logging và logic
- Không để logic logging làm rối code chính
- Chỉ ghi log và rethrow nếu cần thiết

## Tối ưu hóa

### Tránh log thừa thãi
- Chỉ log những thông tin thực sự cần thiết
- Tránh logging trong vòng lặp hoặc code thực thi thường xuyên

```csharp
// ❌ Sai - Log trong vòng lặp
foreach (var item in items) {
    _logger.LogDebug("Processing item {ItemId}", item.Id);
    // xử lý item
}

// ✅ Đúng
_logger.LogDebug("Start processing {Count} items", items.Count);
foreach (var item in items) {
    // xử lý item
}
_logger.LogDebug("Completed processing {Count} items", items.Count);
```

### Kiểm tra trước khi log
- Sử dụng `_logger.IsEnabled()` để tránh tạo message không cần thiết

```csharp
if (_logger.IsEnabled(LogLevel.Debug)) {
    var complexData = GenerateComplexDebugData(); // Chỉ thực hiện nếu Debug được bật
    _logger.LogDebug("Complex debug data: {Data}", complexData);
}
```

### Thời gian
- Không tự thêm thời gian vào nội dung log
- Để hệ thống tự động thêm timestamp

```csharp
// ❌ Sai
_logger.LogInformation($"[{DateTime.Now}] User created");

// ✅ Đúng
_logger.LogInformation("User created");
```

## Kiểm tra trước khi commit

### Kiểm tra Dependency Injection
- Đảm bảo `_logger` không null (đã inject đúng)
- Validate constructor injection

### Kiểm tra ngữ cảnh
- Log phải có đủ ngữ cảnh để debug mà không cần đọc code
- Mỗi log entry phải có thông tin định danh (ID, mã giao dịch...)

### Kiểm tra mức độ
- Đã tuân thủ mức độ log phù hợp với tình huống
- Không dùng Error cho tình huống bình thường

## Bổ sung từ các hãng lớn

### Structured Logging
- Sử dụng JSON hoặc định dạng chuẩn để dễ phân tích
- Hỗ trợ tốt cho các công cụ phân tích log

```csharp
// Trong Program.cs
builder.Logging.AddJsonConsole(options => {
    options.IncludeScopes = true;
    options.TimestampFormat = "yyyy-MM-dd HH:mm:ss ";
});
```

### Tracing
- Thêm request ID hoặc trace ID để theo dõi luồng xử lý
- Sử dụng correlation IDs xuyên suốt các service

```csharp
// Trong Middleware
app.Use(async (context, next) => {
    var correlationId = context.Request.Headers["X-Correlation-ID"].FirstOrDefault() 
                        ?? Guid.NewGuid().ToString();
    
    using (LogContext.PushProperty("CorrelationId", correlationId))
    {
        await next();
    }
});
```

### Phân loại log
- Tách biệt log nghiệp vụ (business) và log kỹ thuật (technical)
- Sử dụng category hoặc tags

### Format key-value
- Ghi log ở dạng key-value để hỗ trợ tìm kiếm và phân tích

```csharp
_logger.LogInformation("Order processed. OrderId={OrderId}, CustomerId={CustomerId}, Amount={Amount}", 
                      order.Id, order.CustomerId, order.Amount);
```

### Async logging
- Đảm bảo log không blocking ứng dụng
- Sử dụng async logging nếu có thể

### Log Schema
- Cung cấp tài liệu hướng dẫn log cho team
- Mô tả rõ các event, format và context cần có

## Các mục bổ sung

### Tuân thủ quy định (GDPR, HIPAA)
- Đảm bảo log không vi phạm các tiêu chuẩn bảo mật
- Không lưu thông tin cá nhân nếu không cần thiết

```csharp
// ✅ Đúng
_logger.LogInformation("User with ID {UserId} requested personal data", userId);

// ❌ Sai
_logger.LogInformation("User {Name} ({Email}, {Phone}) requested personal data", 
                      userName, userEmail, userPhone);
```

### Xử lý dữ liệu nhạy cảm
- Tự động mask/mã hóa thông tin nhạy cảm trong log
- Sử dụng các extension method để xử lý

```csharp
// Extension method
public static string MaskEmail(this string email)
{
    if (string.IsNullOrEmpty(email) || !email.Contains("@"))
        return email;
        
    var parts = email.Split('@');
    return parts[0].Substring(0, 2) + "***@" + parts[1];
}

// Sử dụng
_logger.LogInformation("Password reset requested for {Email}", userEmail.MaskEmail());
```

### Logging scopes
- Nhóm các log liên quan bằng `BeginScope`
- Hữu ích cho các operation phức tạp

```csharp
using (_logger.BeginScope("Processing order {OrderId}", order.Id))
{
    _logger.LogInformation("Validating order items");
    // Xử lý
    _logger.LogInformation("Processing payment");
    // Xử lý
    _logger.LogInformation("Order completed");
}
```

### Log rotation & retention
- Cấu hình tự động xóa/xoay vòng file log
- Đặt chính sách lưu trữ hợp lý

```csharp
// Trong Program.cs
builder.Logging.AddFile("logs/app-{Date}.log", fileSizeLimitBytes: 10 * 1024 * 1024, 
                        retainedFileCountLimit: 30);
```

### Tích hợp hệ thống tập trung
- Gửi log đến ELK, Splunk, hoặc dịch vụ logging chuyên dụng
- Sử dụng các provider phù hợp

```csharp
// Trong Program.cs
builder.Logging.AddElasticsearch(options => {
    options.ElasticsearchEndpoint = new Uri("http://localhost:9200");
    options.IndexName = "app-logs";
});
```

### Monitoring & alert
- Thiết lập cảnh báo khi có lỗi Critical/Error vượt ngưỡng
- Tích hợp với hệ thống monitoring

### Chuẩn hóa ngôn ngữ log
- Sử dụng tiếng Anh để dễ dàng phân tích toàn hệ thống
- Đảm bảo tính nhất quán trong toàn bộ codebase

### Ngăn chặn log injection
- Validate input trước khi ghi vào log
- Cẩn thận với dữ liệu người dùng nhập vào

```csharp
// Hàm helper để sanitize input
public static string SanitizeForLog(this string input)
{
    if (string.IsNullOrEmpty(input))
        return input;
        
    // Loại bỏ các ký tự đặc biệt, xuống dòng,...
    return Regex.Replace(input, @"[\r\n\t]", " ").Trim();
}

// Sử dụng
_logger.LogInformation("User comment: {Comment}", userComment.SanitizeForLog());
```

### Thông tin người dùng/session
- Ghi kèm user ID/session ID vào log
- Hỗ trợ theo dõi hành động người dùng

```csharp
// Middleware để thêm thông tin user vào log context
app.Use(async (context, next) => {
    if (context.User.Identity.IsAuthenticated)
    {
        var userId = context.User.FindFirstValue(ClaimTypes.NameIdentifier);
        using (LogContext.PushProperty("UserId", userId))
        {
            await next();
        }
    }
    else
    {
        await next();
    }
});
```

### Kiểm thử logging
- Viết test kiểm tra hành vi logging
- Đảm bảo log đúng trong các tình huống khác nhau

```csharp
[Fact]
public void CreateUser_Success_ShouldLogInformation()
{
    // Arrange
    var loggerMock = new Mock<ILogger<UserService>>();
    var userService = new UserService(loggerMock.Object, _userRepositoryMock.Object);
    
    // Act
    userService.CreateUser(new UserDto { /* ... */ });
    
    // Assert
    loggerMock.Verify(
        x => x.Log(
            LogLevel.Information,
            It.IsAny<EventId>(),
            It.Is<It.IsAnyType>((v, t) => v.ToString().Contains("User created successfully")),
            It.IsAny<Exception>(),
            It.IsAny<Func<It.IsAnyType, Exception, string>>()),
        Times.Once);
}
```
