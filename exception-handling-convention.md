# Code Convention cho Backend C# & ASP.NET Core

## Nguyên tắc xử lý lỗi (Exception Handling)

### 1. Không sử dụng try-catch dư thừa
- Chỉ sử dụng try-catch khi thực sự cần xử lý hoặc log lỗi
- **Không nên làm**:
```csharp
public int Add(int a, int b)
{
    try
    {
        return a + b; // Không cần try-catch cho phép tính đơn giản
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Lỗi khi thực hiện phép cộng");
        throw;
    }
}
```

- **Nên làm**:
```csharp
public int Add(int a, int b)
{
    return a + b;
}
```

### 2. Không catch Exception chung chung
- Tránh catch (Exception ex), chỉ bắt các loại lỗi cụ thể mà có thể xử lý được
- **Không nên làm**:
```csharp
try
{
    var result = _userRepository.GetById(id);
    return result;
}
catch (Exception ex) // Quá chung chung
{
    _logger.LogError(ex, "Lỗi khi lấy thông tin người dùng");
    throw;
}
```

- **Nên làm**:
```csharp
try
{
    var result = _userRepository.GetById(id);
    return result;
}
catch (SqlException ex)
{
    _logger.LogError(ex, "Lỗi kết nối database khi lấy thông tin người dùng");
    throw new RepositoryException("Không thể truy cập dữ liệu người dùng", ex);
}
catch (DbUpdateException ex)
{
    _logger.LogError(ex, "Lỗi cập nhật dữ liệu khi lấy thông tin người dùng");
    throw new RepositoryException("Không thể xử lý dữ liệu người dùng", ex);
}
```

### 3. Luôn log exception
- Sử dụng hệ thống logging chuẩn như ILogger, ELK, Datadog, etc.
- **Không nên làm**:
```csharp
try
{
    // Thực hiện một thao tác
}
catch (Exception ex)
{
    // Không log gì cả
    throw new CustomException("Đã xảy ra lỗi");
}
```

- **Nên làm**:
```csharp
try
{
    // Thực hiện một thao tác
}
catch (Exception ex)
{
    _logger.LogError(ex, "Lỗi khi thực hiện thao tác X với đối tượng {ObjectId}", objectId);
    throw new CustomException("Đã xảy ra lỗi khi xử lý yêu cầu", ex);
}
```

### 4. Không swallow lỗi
- Không được bắt lỗi mà không xử lý hoặc log
- **Không nên làm**:
```csharp
try
{
    // Thực hiện thao tác
    DeleteFile(path);
}
catch (Exception)
{
    // "Swallowing" exception - không log, không xử lý
}
```

- **Nên làm**:
```csharp
try
{
    DeleteFile(path);
}
catch (FileNotFoundException ex)
{
    _logger.LogWarning(ex, "File không tồn tại tại đường dẫn {FilePath}", path);
    // Xử lý tình huống file không tồn tại
}
catch (UnauthorizedAccessException ex)
{
    _logger.LogError(ex, "Không có quyền xóa file tại {FilePath}", path);
    throw new BusinessException("Không thể xóa tệp do thiếu quyền truy cập", ex);
}
```

### 5. Không trả về lỗi chi tiết cho client
- Tránh lộ thông tin hệ thống như stack trace
- **Không nên làm**:
```csharp
catch (Exception ex)
{
    return StatusCode(500, new
    {
        error = ex.ToString(), // Lộ stack trace và thông tin nhạy cảm
        stackTrace = ex.StackTrace
    });
}
```

- **Nên làm**:
```csharp
catch (Exception ex)
{
    _logger.LogError(ex, "Lỗi khi xử lý yêu cầu");
    return StatusCode(500, new
    {
        error = "Đã xảy ra lỗi hệ thống. Vui lòng thử lại sau.",
        referenceId = Guid.NewGuid().ToString() // ID để tra cứu log
    });
}
```

### 6. Sử dụng middleware xử lý lỗi tập trung
- Centralize error handling bằng middleware để xử lý nhất quán
- **Nên làm**:
```csharp
// Trong Startup.cs hoặc Program.cs
app.UseExceptionHandler(appBuilder =>
{
    appBuilder.Run(async context =>
    {
        var exceptionHandlerFeature = context.Features.Get<IExceptionHandlerFeature>();
        var exception = exceptionHandlerFeature.Error;
        
        _logger.LogError(exception, "Unhandled exception");
        
        context.Response.StatusCode = exception switch
        {
            BusinessException => StatusCodes.Status400BadRequest,
            NotFoundException => StatusCodes.Status404NotFound,
            _ => StatusCodes.Status500InternalServerError
        };
        
        // Trả về response chuẩn
        var result = JsonSerializer.Serialize(new
        {
            error = GetUserFriendlyMessage(exception),
            referenceId = Activity.Current?.Id ?? context.TraceIdentifier
        });
        
        context.Response.ContentType = "application/json";
        await context.Response.WriteAsync(result);
    });
});
```

### 7. Không bắt exception trong Application Layer
- Để lỗi propagate lên middleware xử lý tập trung
- **Không nên làm**:
```csharp
// Trong Application Service
public async Task<UserDto> GetUserAsync(int id)
{
    try 
    {
        var user = await _userRepository.GetByIdAsync(id);
        return _mapper.Map<UserDto>(user);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Lỗi khi lấy thông tin user {UserId}", id);
        throw;
    }
}
```

- **Nên làm**:
```csharp
// Trong Application Service
public async Task<UserDto> GetUserAsync(int id)
{
    var user = await _userRepository.GetByIdAsync(id);
    if (user == null)
    {
        throw new NotFoundException($"Không tìm thấy user với id {id}");
    }
    return _mapper.Map<UserDto>(user);
}
```

### 8. Không trả về message lỗi trực tiếp từ Exception
- Tránh lộ thông tin nhạy cảm như connection string
- **Không nên làm**:
```csharp
catch (SqlException ex)
{
    return BadRequest(new
    {
        error = ex.Message // Có thể chứa connection string
    });
}
```

- **Nên làm**:
```csharp
catch (SqlException ex)
{
    _logger.LogError(ex, "Lỗi database khi xử lý yêu cầu");
    return BadRequest(new
    {
        error = "Không thể xử lý yêu cầu do lỗi kết nối cơ sở dữ liệu"
    });
}
```

### 9. Ưu tiên dùng Result<T> thay vì exception
- Tránh dùng exception để điều khiển luồng logic
- **Không nên làm**:
```csharp
public User GetUser(int id)
{
    var user = _dbContext.Users.Find(id);
    if (user == null)
    {
        throw new NotFoundException("User không tồn tại");
    }
    return user;
}
```

- **Nên làm**:
```csharp
public Result<User> GetUser(int id)
{
    var user = _dbContext.Users.Find(id);
    if (user == null)
    {
        return Result<User>.Failure("User không tồn tại");
    }
    return Result<User>.Success(user);
}

// Sử dụng
var result = _userService.GetUser(id);
if (result.IsSuccess)
{
    var user = result.Value;
    // Xử lý user
}
else
{
    // Xử lý lỗi
}
```

### 10. Không ném Exception trong Domain Model
- Dùng DomainException hoặc pattern phù hợp
- **Không nên làm**:
```csharp
public class User
{
    public void ChangeEmail(string email)
    {
        if (string.IsNullOrEmpty(email))
        {
            throw new Exception("Email không được để trống");
        }
        if (!email.Contains("@"))
        {
            throw new Exception("Email không hợp lệ");
        }
        Email = email;
    }
}
```

- **Nên làm**:
```csharp
public class User
{
    public void ChangeEmail(string email)
    {
        if (string.IsNullOrEmpty(email))
        {
            throw new DomainException("Email không được để trống");
        }
        if (!email.Contains("@"))
        {
            throw new DomainException("Email không hợp lệ");
        }
        Email = email;
    }
}
```

### 11. Không throw Exception chung chung
- Dùng exception cụ thể
- **Không nên làm**:
```csharp
if (amount <= 0)
{
    throw new Exception("Số tiền không hợp lệ");
}
```

- **Nên làm**:
```csharp
if (amount <= 0)
{
    throw new ArgumentException("Số tiền phải lớn hơn 0", nameof(amount));
}
```

### 12. Bắt lỗi cụ thể trong Repository
- Chỉ catch lỗi liên quan đến database
- **Nên làm**:
```csharp
public async Task<User> GetByIdAsync(int id)
{
    try
    {
        return await _dbContext.Users.FindAsync(id);
    }
    catch (SqlException ex)
    {
        _logger.LogError(ex, "Lỗi SQL khi truy vấn user {UserId}", id);
        throw new RepositoryException("Lỗi khi truy cập dữ liệu user", ex);
    }
    catch (DbUpdateException ex)
    {
        _logger.LogError(ex, "Lỗi cập nhật dữ liệu user {UserId}", id);
        throw new RepositoryException("Lỗi khi cập nhật dữ liệu user", ex);
    }
}
```

### 13. Ném RepositoryException nếu lỗi liên quan đến database
- Phân biệt lỗi business và hạ tầng
- **Nên làm**:
```csharp
public class RepositoryException : Exception
{
    public RepositoryException(string message) : base(message)
    {
    }

    public RepositoryException(string message, Exception innerException) 
        : base(message, innerException)
    {
    }
}

// Sử dụng
catch (SqlException ex)
{
    throw new RepositoryException("Không thể kết nối đến cơ sở dữ liệu", ex);
}
```

### 14. Trả về response chuẩn (HTTP Status Code)
- Sử dụng mã lỗi HTTP phù hợp
- **Nên làm**:
```csharp
// Trong global error handling middleware
var statusCode = exception switch
{
    ValidationException => StatusCodes.Status400BadRequest,
    NotFoundException => StatusCodes.Status404NotFound,
    UnauthorizedAccessException => StatusCodes.Status403Forbidden,
    _ => StatusCodes.Status500InternalServerError
};

context.Response.StatusCode = statusCode;
```

### 15. Sử dụng khối finally/using để giải phóng tài nguyên
- Đảm bảo đóng kết nối database, file handle, etc.
- **Không nên làm**:
```csharp
public void SaveToFile(string content, string path)
{
    var writer = new StreamWriter(path);
    writer.Write(content);
    writer.Close(); // Có thể không được gọi nếu có exception
}
```

- **Nên làm** (C# 8.0+):
```csharp
public void SaveToFile(string content, string path)
{
    using var writer = new StreamWriter(path);
    writer.Write(content);
    // Tự động dispose khi kết thúc scope
}
```

- **Hoặc** (trước C# 8.0):
```csharp
public void SaveToFile(string content, string path)
{
    using (var writer = new StreamWriter(path))
    {
        writer.Write(content);
    }
}
```

### 16. Tránh log thông tin nhạy cảm
- Ngăn rủi ro bảo mật từ log
- **Không nên làm**:
```csharp
_logger.LogInformation("Đăng nhập với tài khoản: {Username}, mật khẩu: {Password}", username, password);
```

- **Nên làm**:
```csharp
_logger.LogInformation("Người dùng {Username} đã đăng nhập thành công", username);
```

### 17. Validate input đầu vào để giảm thiểu lỗi
- Kiểm tra dữ liệu trước khi xử lý
- **Nên làm**:
```csharp
public void ProcessOrder(Order order)
{
    if (order == null)
    {
        throw new ArgumentNullException(nameof(order));
    }

    if (order.Items == null || !order.Items.Any())
    {
        throw new ArgumentException("Đơn hàng phải có ít nhất một sản phẩm", nameof(order));
    }

    if (order.DeliveryDate <= DateTime.Now)
    {
        throw new ArgumentException("Ngày giao hàng phải sau ngày hiện tại", nameof(order.DeliveryDate));
    }

    // Xử lý đơn hàng
}
```

### 18. Xử lý exception trong async/await đúng cách
- Dùng await và ConfigureAwait để tránh deadlock
- **Không nên làm**:
```csharp
public async Task<User> GetUserAsync(int id)
{
    var user = _userRepository.GetByIdAsync(id).Result; // Có thể gây deadlock
    return user;
}
```

- **Nên làm**:
```csharp
public async Task<User> GetUserAsync(int id)
{
    var user = await _userRepository.GetByIdAsync(id).ConfigureAwait(false);
    return user;
}
```

### 19. Tạo custom exception hierarchy cho ứng dụng
- Phân loại lỗi rõ ràng
- **Nên làm**:
```csharp
// Base exception
public abstract class AppException : Exception
{
    protected AppException(string message) : base(message) { }
    protected AppException(string message, Exception inner) : base(message, inner) { }
}

// Business exceptions
public class BusinessException : AppException
{
    public BusinessException(string message) : base(message) { }
    public BusinessException(string message, Exception inner) : base(message, inner) { }
}

// Domain exceptions
public class DomainException : BusinessException
{
    public DomainException(string message) : base(message) { }
    public DomainException(string message, Exception inner) : base(message, inner) { }
}

// Infrastructure exceptions
public class InfrastructureException : AppException
{
    public InfrastructureException(string message) : base(message) { }
    public InfrastructureException(string message, Exception inner) : base(message, inner) { }
}

// Repository exceptions
public class RepositoryException : InfrastructureException
{
    public RepositoryException(string message) : base(message) { }
    public RepositoryException(string message, Exception inner) : base(message, inner) { }
}
```

### 20. Áp dụng nguyên tắc Fail Fast
- Validate điều kiện sớm, thất bại ngay nếu không hợp lệ
- **Nên làm**:
```csharp
public void ProcessPayment(PaymentRequest request)
{
    // Validate ngay từ đầu
    if (request == null)
    {
        throw new ArgumentNullException(nameof(request));
    }
    
    if (request.Amount <= 0)
    {
        throw new BusinessException("Số tiền thanh toán phải lớn hơn 0");
    }
    
    if (string.IsNullOrEmpty(request.PaymentMethod))
    {
        throw new BusinessException("Phương thức thanh toán không được để trống");
    }
    
    // Tiếp tục xử lý khi đã biết chắc dữ liệu hợp lệ
    _paymentService.Process(request);
}
```

### 21. Implement retry mechanism cho lỗi tạm thời
- Xử lý lỗi kết nối, timeout bằng Polly hoặc tương tự
- **Nên làm**:
```csharp
// Sử dụng Polly
public async Task<Order> GetOrderAsync(int orderId)
{
    return await Policy
        .Handle<SqlException>()
        .Or<HttpRequestException>()
        .WaitAndRetryAsync(
            retryCount: 3,
            sleepDurationProvider: retry => TimeSpan.FromSeconds(Math.Pow(2, retry)),
            onRetry: (exception, timeSpan, retryCount, context) =>
            {
                _logger.LogWarning(exception,
                    "Lỗi khi lấy thông tin đơn hàng {OrderId}. Đang thử lại lần {RetryCount}",
                    orderId, retryCount);
            })
        .ExecuteAsync(() => _orderRepository.GetByIdAsync(orderId));
}
```

### 22. Document các exception có thể throw từ method
- Dùng XML comment hoặc công cụ document
- **Nên làm**:
```csharp
/// <summary>
/// Xử lý thanh toán cho đơn hàng
/// </summary>
/// <param name="order">Đơn hàng cần thanh toán</param>
/// <param name="paymentMethod">Phương thức thanh toán</param>
/// <returns>Kết quả thanh toán</returns>
/// <exception cref="ArgumentNullException">Ném ra khi order là null</exception>
/// <exception cref="BusinessException">Ném ra khi đơn hàng không hợp lệ</exception>
/// <exception cref="PaymentException">Ném ra khi có lỗi trong quá trình thanh toán</exception>
public PaymentResult ProcessPayment(Order order, string paymentMethod)
{
    // Xử lý thanh toán
}
```

### 23. Sử dụng exception filter để xử lý điều kiện
- Lọc exception theo điều kiện
- **Nên làm**:
```csharp
try
{
    // Thực hiện một tác vụ phức tạp
}
catch (Exception ex) when (ex is SqlException sqlEx && sqlEx.Number == 1205)
{
    _logger.LogWarning(ex, "Deadlock phát hiện, đang thử lại");
    // Xử lý deadlock
}
catch (Exception ex) when (ex is TimeoutException)
{
    _logger.LogError(ex, "Timeout khi thực hiện tác vụ");
    // Xử lý timeout
}
```

### 24. Thêm correlation ID vào log
- Giúp liên kết log giữa các service
- **Nên làm**:
```csharp
// Trong middleware
app.Use(async (context, next) =>
{
    // Sử dụng correlation ID từ header hoặc tạo mới
    var correlationId = context.Request.Headers["X-Correlation-ID"].FirstOrDefault() ?? Guid.NewGuid().ToString();
    context.Response.Headers["X-Correlation-ID"] = correlationId;
    
    // Thêm vào log context
    using (LogContext.PushProperty("CorrelationId", correlationId))
    {
        await next();
    }
});

// Trong code
_logger.LogInformation("Xử lý yêu cầu {RequestPath} cho người dùng {UserId}", 
    context.Request.Path, userId);
// Log sẽ tự động bao gồm CorrelationId
```
