# Code Convention ASP.NET Core API

## 1. HTTP Methods chuẩn RESTful

- **Sử dụng đúng HTTP methods cho mục đích tương ứng**:
  - `GET`: Lấy dữ liệu (không thay đổi trạng thái server)
  - `POST`: Tạo mới resource
  - `PUT`: Cập nhật toàn bộ resource
  - `PATCH`: Cập nhật một phần resource
  - `DELETE`: Xóa resource

- **Ví dụ**:
```csharp
// Controller
[ApiController]
[Route("api/v1/users")]
public class UsersController : ControllerBase
{
    [HttpGet]
    public async Task<IActionResult> GetAll() { ... }
    
    [HttpGet("{id}")]
    public async Task<IActionResult> GetById(int id) { ... }
    
    [HttpPost]
    public async Task<IActionResult> Create([FromBody] UserCreateDto dto) { ... }
    
    [HttpPut("{id}")]
    public async Task<IActionResult> Update(int id, [FromBody] UserUpdateDto dto) { ... }
    
    [HttpPatch("{id}")]
    public async Task<IActionResult> PartialUpdate(int id, [FromBody] JsonPatchDocument<UserUpdateDto> patchDoc) { ... }
    
    [HttpDelete("{id}")]
    public async Task<IActionResult> Delete(int id) { ... }
}
```

## 2. Đặt tên endpoint

- **Sử dụng danh từ số nhiều và kebab-case**:
  - Đúng: `/api/v1/user-profiles`, `/api/v1/order-items`
  - Sai: `/api/v1/getUsers`, `/api/v1/user_profile`

- **Tên ngắn gọn, tránh động từ**:
  - Đúng: `/api/v1/users`
  - Sai: `/api/v1/get-all-users`

- **Ví dụ**:
```csharp
// Đúng
[Route("api/v1/product-categories")]
public class ProductCategoriesController : ControllerBase { ... }

// Sai
[Route("api/v1/get_product_categories")]
public class GetProductCategoryController : ControllerBase { ... }
```

## 3. Versioning API

- **Thêm phiên bản vào URL**:
  - Mẫu: `/api/v{major}/resource`
  - Ví dụ: `/api/v1/users`, `/api/v2/users`

- **Cấu hình versioning trong Startup.cs**:
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddApiVersioning(config =>
    {
        config.DefaultApiVersion = new ApiVersion(1, 0);
        config.AssumeDefaultVersionWhenUnspecified = true;
        config.ReportApiVersions = true;
    });
}
```

- **Định nghĩa version trong controller**:
```csharp
[ApiController]
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/users")]
public class UsersV1Controller : ControllerBase { ... }

[ApiController]
[ApiVersion("2.0")]
[Route("api/v{version:apiVersion}/users")]
public class UsersV2Controller : ControllerBase { ... }
```

## 4. Mã trạng thái HTTP

- **Trả về mã HTTP chính xác theo ngữ cảnh**:
  - `200 OK`: Request thành công
  - `201 Created`: Resource đã được tạo thành công
  - `204 No Content`: Request thành công nhưng không có nội dung trả về
  - `400 Bad Request`: Lỗi từ phía client
  - `401 Unauthorized`: Chưa xác thực
  - `403 Forbidden`: Không có quyền truy cập
  - `404 Not Found`: Resource không tồn tại
  - `500 Internal Server Error`: Lỗi server

- **Ví dụ**:
```csharp
// GET - Lấy thành công
return Ok(result); // 200

// POST - Tạo mới thành công
return CreatedAtAction(nameof(GetById), new { id = entity.Id }, entity); // 201

// DELETE - Xóa thành công
return NoContent(); // 204

// Validation lỗi
return BadRequest(ModelState); // 400

// Resource không tồn tại
return NotFound(); // 404
```

## 5. Định dạng JSON

- **Sử dụng camelCase cho property trong JSON**:
  - Đúng: `{ "userId": 1, "firstName": "John" }`
  - Sai: `{ "UserId": 1, "FirstName": "John" }`

- **Cấu hình trong Startup.cs**:
```csharp
services.AddControllers()
    .AddJsonOptions(options =>
    {
        options.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
        options.JsonSerializerOptions.DictionaryKeyPolicy = JsonNamingPolicy.CamelCase;
        options.JsonSerializerOptions.WriteIndented = true;
    });
```

- **Sử dụng Data Transfer Objects (DTOs)**:
```csharp
public class UserDto
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    
    // Sẽ trở thành "emailAddress" trong JSON
    public string EmailAddress { get; set; }
}
```

## 6. Tham số đầu vào

- **Sử dụng đúng attribute cho tham số**:
  - `[FromBody]`: Dữ liệu từ body request
  - `[FromQuery]`: Dữ liệu từ query string
  - `[FromRoute]`: Dữ liệu từ route template
  
- **Validate dữ liệu sử dụng Data Annotations**:
```csharp
public class CreateUserDto
{
    [Required(ErrorMessage = "Tên là bắt buộc")]
    [StringLength(50, ErrorMessage = "Tên không được vượt quá 50 ký tự")]
    public string Name { get; set; }
    
    [Required(ErrorMessage = "Email là bắt buộc")]
    [EmailAddress(ErrorMessage = "Email không hợp lệ")]
    public string Email { get; set; }
    
    [Required(ErrorMessage = "Mật khẩu là bắt buộc")]
    [MinLength(8, ErrorMessage = "Mật khẩu phải có ít nhất 8 ký tự")]
    public string Password { get; set; }
}
```

- **Ví dụ sử dụng**:
```csharp
[HttpGet]
public async Task<IActionResult> Search([FromQuery] UserSearchParams searchParams) { ... }

[HttpPost]
public async Task<IActionResult> Create([FromBody] CreateUserDto dto) { ... }

[HttpGet("{id}")]
public async Task<IActionResult> GetById([FromRoute] int id) { ... }
```

## 7. Xử lý lỗi

- **Trả về JSON với errorCode và message**:
```csharp
public class ApiError
{
    public string ErrorCode { get; set; }
    public string Message { get; set; }
    public object Details { get; set; }
}
```

- **Tạo middleware xử lý exception**:
```csharp
public class GlobalExceptionHandlerMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<GlobalExceptionHandlerMiddleware> _logger;

    public GlobalExceptionHandlerMiddleware(RequestDelegate next, ILogger<GlobalExceptionHandlerMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (NotFoundException ex)
        {
            await HandleExceptionAsync(context, ex, HttpStatusCode.NotFound, "RESOURCE_NOT_FOUND");
        }
        catch (ValidationException ex)
        {
            await HandleExceptionAsync(context, ex, HttpStatusCode.BadRequest, "VALIDATION_FAILED");
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Unhandled exception");
            await HandleExceptionAsync(context, ex, HttpStatusCode.InternalServerError, "INTERNAL_SERVER_ERROR");
        }
    }

    private static async Task HandleExceptionAsync(HttpContext context, Exception exception, HttpStatusCode statusCode, string errorCode)
    {
        context.Response.ContentType = "application/json";
        context.Response.StatusCode = (int)statusCode;

        var error = new ApiError
        {
            ErrorCode = errorCode,
            Message = exception.Message,
            Details = exception.InnerException?.Message
        };

        await context.Response.WriteAsJsonAsync(error);
    }
}
```

- **Đăng ký middleware trong Startup.cs**:
```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.UseMiddleware<GlobalExceptionHandlerMiddleware>();
    
    // ...
}
```

## 8. Tách logic khỏi controller

- **Controller chỉ xử lý request/response, không chứa business logic**:
```csharp
[ApiController]
[Route("api/v1/users")]
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;
    
    public UsersController(IUserService userService)
    {
        _userService = userService;
    }
    
    [HttpGet]
    public async Task<IActionResult> GetAll()
    {
        var users = await _userService.GetAllAsync();
        return Ok(users);
    }
    
    [HttpPost]
    public async Task<IActionResult> Create([FromBody] CreateUserDto dto)
    {
        var user = await _userService.CreateAsync(dto);
        return CreatedAtAction(nameof(GetById), new { id = user.Id }, user);
    }
}
```

- **Đặt business logic trong service layer**:
```csharp
public class UserService : IUserService
{
    private readonly IUserRepository _userRepository;
    private readonly IMapper _mapper;
    
    public UserService(IUserRepository userRepository, IMapper mapper)
    {
        _userRepository = userRepository;
        _mapper = mapper;
    }
    
    public async Task<IEnumerable<UserDto>> GetAllAsync()
    {
        var users = await _userRepository.GetAllAsync();
        return _mapper.Map<IEnumerable<UserDto>>(users);
    }
    
    public async Task<UserDto> CreateAsync(CreateUserDto dto)
    {
        // Validation logic
        
        // Business logic
        var user = _mapper.Map<User>(dto);
        
        // Persistence
        await _userRepository.AddAsync(user);
        
        return _mapper.Map<UserDto>(user);
    }
}
```

## 9. Tính nhất quán trong tên

- **Tránh viết tắt, sử dụng tên mô tả rõ ràng**:
  - Đúng: `/api/v1/user-accounts`, `/api/v1/product-categories`
  - Sai: `/api/v1/ua`, `/api/v1/pc`

- **Tên endpoint tự giải thích**:
  - Đúng: `/api/v1/orders/{orderId}/items`
  - Sai: `/api/v1/o/{id}/i`

- **Áp dụng quy tắc này trong tất cả các layer**:
```csharp
// Đúng
public interface IOrderRepository { ... }
public class OrderService : IOrderService { ... }
public class OrdersController : ControllerBase { ... }

// Sai
public interface IOrdRepo { ... }
public class OrdSvc : IOrdSvc { ... }
public class OrdController : ControllerBase { ... }
```

## 10. Hỗ trợ phân trang

- **Sử dụng query parameters cho phân trang**:
  - Format: `?page=1&size=10`
  - Mặc định: page=1, size=10 nếu không được cung cấp

- **Tạo class Pagination Request**:
```csharp
public class PaginationParams
{
    private const int MaxPageSize = 50;
    private int _pageSize = 10;
    
    [FromQuery(Name = "page")]
    public int PageNumber { get; set; } = 1;
    
    [FromQuery(Name = "size")]
    public int PageSize
    {
        get => _pageSize;
        set => _pageSize = (value > MaxPageSize) ? MaxPageSize : value;
    }
}
```

- **Tạo class Pagination Response**:
```csharp
public class PagedResponse<T>
{
    public IEnumerable<T> Items { get; set; }
    public int PageNumber { get; set; }
    public int PageSize { get; set; }
    public int TotalPages { get; set; }
    public int TotalItems { get; set; }
    public bool HasPrevious => PageNumber > 1;
    public bool HasNext => PageNumber < TotalPages;
}
```

- **Ví dụ sử dụng**:
```csharp
[HttpGet]
public async Task<IActionResult> GetAll([FromQuery] PaginationParams pagingParams)
{
    var pagedResult = await _userService.GetPagedListAsync(pagingParams);
    return Ok(pagedResult);
}
```

## 11. Tránh nested resources sâu

- **Giới hạn độ sâu URL**:
  - Đúng: `/api/v1/users/{id}/orders`
  - Sai: `/api/v1/users/{userId}/orders/{orderId}/items/{itemId}/details`

- **Sử dụng query parameters thay vì nested resources quá sâu**:
  - Thay vì: `/api/v1/users/{userId}/orders/{orderId}/items`
  - Sử dụng: `/api/v1/order-items?userId={userId}&orderId={orderId}`

- **Ví dụ**:
```csharp
// Độ sâu hợp lý
[HttpGet("users/{userId}/orders")]
public async Task<IActionResult> GetUserOrders(int userId) { ... }

// Thay vì quá nhiều cấp
// [HttpGet("users/{userId}/orders/{orderId}/items")]
[HttpGet("order-items")]
public async Task<IActionResult> GetOrderItems([FromQuery] int userId, [FromQuery] int orderId) { ... }
```

## 12. Dùng HTTPS

- **Tất cả API phải chạy trên HTTPS**

- **Cấu hình trong Program.cs**:
```csharp
public static void Main(string[] args)
{
    var builder = WebApplication.CreateBuilder(args);
    
    // Thêm cấu hình HTTPS
    builder.Services.AddHttpsRedirection(options =>
    {
        options.RedirectStatusCode = StatusCodes.Status307TemporaryRedirect;
        options.HttpsPort = 5001;
    });
    
    // ...
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // ...
    app.UseHttpsRedirection();
    // ...
}
```

- **Sử dụng HSTS (HTTP Strict Transport Security)**:
```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (!env.IsDevelopment())
    {
        app.UseHsts();
    }
    
    app.UseHttpsRedirection();
    // ...
}
```

## 13. Rate Limiting

- **Giới hạn số lượng request bằng middleware**:

- **Cấu hình Rate Limiting**:
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddRateLimiter(options =>
    {
        options.GlobalLimiter = PartitionedRateLimiter.Create<HttpContext, string>(httpContext =>
            RateLimitPartition.GetFixedWindowLimiter(
                partitionKey: httpContext.User.Identity?.Name ?? httpContext.Request.Headers.Host.ToString(),
                factory: partition => new FixedWindowRateLimiterOptions
                {
                    AutoReplenishment = true,
                    PermitLimit = 100,
                    QueueLimit = 0,
                    Window = TimeSpan.FromMinutes(1)
                }));

        options.OnRejected = async (context, token) =>
        {
            context.HttpContext.Response.StatusCode = 429; // Too Many Requests
            context.HttpContext.Response.Headers.Add("X-RateLimit-Limit", "100");
            context.HttpContext.Response.Headers.Add("X-RateLimit-Remaining", "0");
            context.HttpContext.Response.Headers.Add("X-RateLimit-Reset", DateTimeOffset.UtcNow.AddMinutes(1).ToUnixTimeSeconds().ToString());
            
            await context.HttpContext.Response.WriteAsJsonAsync(new
            {
                errorCode = "RATE_LIMIT_EXCEEDED",
                message = "API rate limit exceeded"
            });
        };
    });
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // ...
    app.UseRateLimiter();
    // ...
}
```

## 14. HATEOAS

- **Thêm hypermedia links trong response**:
```csharp
public class EntityWithLinks<T>
{
    public T Data { get; set; }
    public List<Link> Links { get; set; } = new List<Link>();
}

public class Link
{
    public string Href { get; set; }
    public string Rel { get; set; }
    public string Method { get; set; }
    
    public Link(string href, string rel, string method)
    {
        Href = href;
        Rel = rel;
        Method = method;
    }
}
```

- **Ví dụ sử dụng trong controller**:
```csharp
[HttpGet("{id}")]
public async Task<IActionResult> GetById(int id)
{
    var user = await _userService.GetByIdAsync(id);
    if (user == null)
        return NotFound();
    
    var userWithLinks = new EntityWithLinks<UserDto>
    {
        Data = user,
        Links = new List<Link>
        {
            new Link(Url.Action(nameof(GetById), new { id }), "self", "GET"),
            new Link(Url.Action(nameof(Update), new { id }), "update", "PUT"),
            new Link(Url.Action(nameof(Delete), new { id }), "delete", "DELETE"),
            new Link(Url.Action(nameof(GetUserOrders), new { userId = id }), "orders", "GET")
        }
    };
    
    return Ok(userWithLinks);
}
```

## 15. Caching

- **Sử dụng HTTP headers cho caching**:

- **Cấu hình Response Caching**:
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCaching();
    services.AddControllers(options =>
    {
        options.CacheProfiles.Add("Default30",
            new CacheProfile
            {
                Duration = 30,
                Location = ResponseCacheLocation.Any
            });
    });
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // ...
    app.UseResponseCaching();
    // ...
}
```

- **Áp dụng trong controller**:
```csharp
[HttpGet]
[ResponseCache(CacheProfileName = "Default30")]
public async Task<IActionResult> GetAll()
{
    var users = await _userService.GetAllAsync();
    return Ok(users);
}

[HttpGet("{id}")]
[ResponseCache(Duration = 60, VaryByQueryKeys = new[] { "id" })]
public async Task<IActionResult> GetById(int id)
{
    var user = await _userService.GetByIdAsync(id);
    if (user == null)
        return NotFound();
    
    return Ok(user);
}
```

- **Sử dụng ETag**:
```csharp
[HttpGet("{id}")]
public async Task<IActionResult> GetById(int id)
{
    var user = await _userService.GetByIdAsync(id);
    if (user == null)
        return NotFound();
    
    var serializedUser = JsonSerializer.Serialize(user);
    var etag = Convert.ToBase64String(System.Security.Cryptography.SHA1.Create().ComputeHash(Encoding.UTF8.GetBytes(serializedUser)));
    
    Response.Headers.Add("ETag", $"\"{etag}\"");
    
    return Ok(user);
}
```

## 16. Filtering/Sorting

- **Hỗ trợ query params cho filtering và sorting**:
  - Filtering: `?filter=status:active,role:admin`
  - Sorting: `?sort=createdAt:desc,name:asc`

- **Tạo class Filter và Sort**:
```csharp
public class FilterParams
{
    [FromQuery(Name = "filter")]
    public string FilterString { get; set; }
    
    public Dictionary<string, string> GetFilters()
    {
        var filters = new Dictionary<string, string>();
        
        if (string.IsNullOrWhiteSpace(FilterString))
            return filters;
        
        var filterParams = FilterString.Split(',');
        foreach (var param in filterParams)
        {
            var keyValue = param.Split(':');
            if (keyValue.Length == 2)
            {
                filters.Add(keyValue[0], keyValue[1]);
            }
        }
        
        return filters;
    }
}

public class SortParams
{
    [FromQuery(Name = "sort")]
    public string SortString { get; set; }
    
    public List<SortParam> GetSorts()
    {
        var sorts = new List<SortParam>();
        
        if (string.IsNullOrWhiteSpace(SortString))
            return sorts;
        
        var sortParams = SortString.Split(',');
        foreach (var param in sortParams)
        {
            var keyValue = param.Split(':');
            if (keyValue.Length == 2)
            {
                var direction = keyValue[1].ToLower() == "desc" ? SortDirection.Descending : SortDirection.Ascending;
                sorts.Add(new SortParam(keyValue[0], direction));
            }
        }
        
        return sorts;
    }
}

public class SortParam
{
    public string Field { get; set; }
    public SortDirection Direction { get; set; }
    
    public SortParam(string field, SortDirection direction)
    {
        Field = field;
        Direction = direction;
    }
}

public enum SortDirection
{
    Ascending,
    Descending
}
```

- **Sử dụng trong controller**:
```csharp
[HttpGet]
public async Task<IActionResult> GetAll([FromQuery] FilterParams filterParams, [FromQuery] SortParams sortParams)
{
    var filters = filterParams.GetFilters();
    var sorts = sortParams.GetSorts();
    
    var users = await _userService.GetFilteredAndSortedAsync(filters, sorts);
    return Ok(users);
}
```

## 17. Idempotency

- **Đảm bảo PUT/PATCH/DELETE là idempotent**:
  - Cùng một request có thể gọi nhiều lần mà không tạo ra tác dụng phụ

- **Xử lý idempotency bằng idempotency key**:
```csharp
[Produces("application/json")]
[ApiController]
public abstract class IdempotentControllerBase : ControllerBase
{
    private readonly IIdempotencyService _idempotencyService;
    
    protected IdempotentControllerBase(IIdempotencyService idempotencyService)
    {
        _idempotencyService = idempotencyService;
    }
    
    protected async Task<bool> IsIdempotentRequestAsync()
    {
        if (!Request.Headers.TryGetValue("X-Idempotency-Key", out var idempotencyKey))
            return false;
        
        return await _idempotencyService.HasProcessedRequestAsync(idempotencyKey);
    }
    
    protected async Task SaveIdempotentRequestAsync(object result)
    {
        if (Request.Headers.TryGetValue("X-Idempotency-Key", out var idempotencyKey))
        {
            await _idempotencyService.SaveProcessedRequestAsync(idempotencyKey, result);
        }
    }
}
```

- **Sử dụng trong controller**:
```csharp
[HttpPut("{id}")]
public async Task<IActionResult> Update(int id, [FromBody] UpdateUserDto dto)
{
    // Kiểm tra idempotency
    if (await IsIdempotentRequestAsync())
    {
        var cachedResult = await _idempotencyService.GetCachedResponseAsync(Request.Headers["X-Idempotency-Key"]);
        return Ok(cachedResult);
    }
    
    var user = await _userService.UpdateAsync(id, dto);
    if (user == null)
        return NotFound();
    
    // Lưu kết quả cho idempotency
    await SaveIdempotentRequestAsync(user);
    
    return Ok(user);
}
```

## 18. Authentication/Authorization

- **Sử dụng JWT cho xác thực**:

- **Cấu hình trong Program.cs**:
```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    var jwtSettings = Configuration.GetSection("JwtSettings");
    
    services.AddAuthentication(options =>
    {
        options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
        options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
    })
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = jwtSettings["Issuer"],
            ValidAudience = jwtSettings["Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(jwtSettings["Secret"]))
        };
    });
    
    services.AddAuthorization(options =>
    {
        options.AddPolicy("RequireAdminRole", policy => policy.RequireRole("Admin"));
        options.AddPolicy("RequireUserRole", policy => policy.RequireRole("User"));
    });
    // ...
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // ...
    app.UseAuthentication();
    app.UseAuthorization();
    // ...
}
```

- **Sử dụng trong controller**:
```csharp
[ApiController]
[Route("api/v1/users")]
[Authorize] // Yêu cầu xác thực cho tất cả endpoints
public class UsersController : ControllerBase
{
    // ...
    
    [HttpGet]
    [Authorize(Policy = "RequireAdminRole")] // Yêu cầu role Admin
    public async Task<IActionResult> GetAll()
    {
        // ...
    }
    
    [HttpGet("profile")]
    [Authorize(Policy = "RequireUserRole")] // Yêu cầu role User
    public async Task<IActionResult> GetProfile()
    {
        var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        // ...
    }
}
```

## 19. Logging/Monitoring

- **Sử dụng ILogger cho logging**:
```csharp
[ApiController]
[Route("api/v1/users")]
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;
    private readonly ILogger<UsersController> _logger;
    
    public UsersController(IUserService userService, ILogger<UsersController> logger)
    {
        _userService = userService;
        _logger = logger;
    }
    
    [HttpGet]
    public async Task<IActionResult> GetAll()
    {
        _logger.LogInformation("Getting all users");
        try
        {
            var users = await _userService.GetAllAsync();
            _logger.LogInformation("Retrieved {Count} users", users.Count());
            return Ok(users);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error occurred while getting all users");
            throw;
        }
    }
}
```

- **Cấu hình logging trong appsettings.json**:
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  }
}
```

- **Tích hợp với Application Insights**:
```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    services.AddApplicationInsightsTelemetry();
    // ...
}
```

## 20. API Documentation

- **Sử dụng Swagger/OpenAPI**:

- **Cài đặt package**:
```bash
dotnet add package Swashbuckle.AspNetCore
```

- **Cấu hình trong Program.cs**:
```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    services.AddSwaggerGen(c =>
    {
        c.SwaggerDoc("v1", new OpenApiInfo
        {
            Title = "My API",
            Version = "v1",
            Description = "API documentation for My Project",
            Contact = new OpenApiContact
            {
                Name = "Development Team",
                Email =