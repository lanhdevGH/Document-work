# Code Convention cho C# ASP.NET Core với Clean Architecture

## 1. Xác thực & Ủy quyền

### 1.1 Sử dụng IdentityServer hoặc ASP.NET Core Identity

- **Yêu cầu**: Áp dụng giải pháp xác thực chuẩn để quản lý người dùng
- **Cách triển khai**:
  - Sử dụng ASP.NET Core Identity cho các ứng dụng đơn giản
  - Sử dụng IdentityServer4 cho hệ thống microservices hoặc nhiều client khác nhau
- **Ví dụ**:

```csharp
// Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));
        
    services.AddIdentity<ApplicationUser, IdentityRole>()
        .AddEntityFrameworkStores<ApplicationDbContext>()
        .AddDefaultTokenProviders();
}
```

### 1.2 Luôn băm mật khẩu bằng bcrypt hoặc PBKDF2

- **Yêu cầu**: Không lưu mật khẩu dạng plain text, chỉ lưu hash an toàn
- **Cách triển khai**:
  - ASP.NET Core Identity đã tích hợp PBKDF2 mặc định
  - Có thể cấu hình để tăng độ phức tạp
- **Ví dụ**:

```csharp
// Startup.cs
services.Configure<PasswordHasherOptions>(options =>
{
    options.IterationCount = 12000; // Tăng số vòng lặp để tăng độ an toàn
});
```

### 1.3 Dùng OAuth 2.0 / OpenID Connect / JWT

- **Yêu cầu**: Tránh sử dụng session-based auth, ưu tiên token-based auth
- **Cách triển khai**:
  - Cấu hình JWT Bearer authentication
  - Thiết lập các claims phù hợp
- **Ví dụ**:

```csharp
// Startup.cs
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
        ValidIssuer = Configuration["JwtIssuer"],
        ValidAudience = Configuration["JwtAudience"],
        IssuerSigningKey = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(Configuration["JwtSecretKey"]))
    };
});
```

### 1.4 Không lưu JWT trong local storage

- **Yêu cầu**: JWT dễ bị tấn công XSS nếu lưu trong local storage
- **Cách triển khai**:
  - Lưu token trong HTTP-Only cookies
  - Thiết lập thêm cookie options để tăng bảo mật
- **Ví dụ**:

```csharp
// Trong LoginController hoặc AuthService
public async Task<IActionResult> Login(LoginViewModel model)
{
    // Xác thực user và tạo token
    var token = GenerateJwtToken(user);
    
    // Lưu vào HTTP-only cookie thay vì trả về để lưu ở localStorage
    Response.Cookies.Append("AuthToken", token, new CookieOptions
    {
        HttpOnly = true,
        Secure = true, // Yêu cầu HTTPS
        SameSite = SameSiteMode.Strict,
        Expires = DateTime.UtcNow.AddMinutes(60)
    });
    
    return Ok();
}
```

### 1.5 Giới hạn thời gian sống của token

- **Yêu cầu**: Access token nên có TTL ngắn, sử dụng refresh token rotation
- **Cách triển khai**:
  - Set thời gian ngắn cho JWT (15-30 phút)
  - Triển khai refresh token với rotation
- **Ví dụ**:

```csharp
// Trong TokenService
public string GenerateAccessToken(ApplicationUser user)
{
    var claims = new[]
    {
        new Claim(JwtRegisteredClaimNames.Sub, user.Id),
        new Claim(JwtRegisteredClaimNames.Email, user.Email),
        new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
    };

    var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_configuration["JwtSecretKey"]));
    var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

    var token = new JwtSecurityToken(
        issuer: _configuration["JwtIssuer"],
        audience: _configuration["JwtAudience"],
        claims: claims,
        expires: DateTime.UtcNow.AddMinutes(30), // Chỉ 30 phút
        signingCredentials: creds
    );

    return new JwtSecurityTokenHandler().WriteToken(token);
}
```

### 1.6 Dùng PKCE để bảo vệ authorization code

- **Yêu cầu**: Ngăn chặn interception attack khi dùng OAuth 2.0
- **Cách triển khai**:
  - Kích hoạt PKCE trong OAuth flow
  - Yêu cầu code_challenge trong authorization request
- **Ví dụ**:

```csharp
// Cấu hình IdentityServer
services.AddIdentityServer()
    .AddDeveloperSigningCredential()
    .AddInMemoryApiResources(Config.GetApiResources())
    .AddInMemoryClients(new Client[] 
    {
        new Client 
        {
            ClientId = "spa_client",
            ClientName = "SPA Client",
            AllowedGrantTypes = GrantTypes.Code,
            RequirePkce = true, // Bắt buộc PKCE
            RequireClientSecret = false,
            RedirectUris = { "https://localhost:5002/callback" },
            PostLogoutRedirectUris = { "https://localhost:5002/" },
            AllowedCorsOrigins = { "https://localhost:5002" },
            AllowedScopes = { "openid", "profile", "api1" }
        }
    });
```

### 1.7 Áp dụng RBAC (Role-Based Access Control) hoặc ABAC

- **Yêu cầu**: Phân quyền theo vai trò hoặc thuộc tính để kiểm soát truy cập
- **Cách triển khai**:
  - Sử dụng Roles và Policy trong ASP.NET Core
  - Định nghĩa rõ ràng các quyền và vai trò
- **Ví dụ**:

```csharp
// Startup.cs - Cấu hình Policy
services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", policy => policy.RequireRole("Admin"));
    options.AddPolicy("UserManager", policy => 
        policy.RequireAssertion(context => 
            context.User.IsInRole("Admin") || 
            context.User.IsInRole("Manager")));
    options.AddPolicy("CanManageUsers", policy =>
        policy.RequireClaim("Permission", "Users.Manage"));
});

// Trong controller hoặc action
[Authorize(Policy = "AdminOnly")]
public IActionResult AdminDashboard()
{
    return View();
}
```

## 2. Bảo vệ API

### 2.1 Không trả thông tin lỗi chi tiết

- **Yêu cầu**: Tránh tiết lộ thông tin nội bộ khi xảy ra lỗi
- **Cách triển khai**:
  - Sử dụng Global Exception Handler
  - Log lỗi chi tiết nhưng trả về thông báo chung
- **Ví dụ**:

```csharp
// GlobalExceptionMiddleware.cs
public class GlobalExceptionMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<GlobalExceptionMiddleware> _logger;

    public GlobalExceptionMiddleware(RequestDelegate next, ILogger<GlobalExceptionMiddleware> logger)
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
        catch (Exception ex)
        {
            _logger.LogError(ex, "Lỗi không xử lý được: {Message}", ex.Message);
            
            // Trả về lỗi chung, không kèm chi tiết
            context.Response.StatusCode = StatusCodes.Status500InternalServerError;
            context.Response.ContentType = "application/json";
            
            var response = new { 
                error = "Đã xảy ra lỗi khi xử lý yêu cầu. Vui lòng thử lại sau." 
            };
            
            await context.Response.WriteAsJsonAsync(response);
        }
    }
}

// Startup.cs
public void Configure(IApplicationBuilder app)
{
    // Thêm middleware xử lý lỗi
    app.UseMiddleware<GlobalExceptionMiddleware>();
    
    // Các middleware khác...
}
```

### 2.2 Bắt buộc HTTPS cho tất cả request

- **Yêu cầu**: Bảo mật dữ liệu truyền tải, tránh MITM attack
- **Cách triển khai**:
  - Cấu hình HTTPS Redirection và HSTS
  - Đảm bảo mọi request đều qua HTTPS
- **Ví dụ**:

```csharp
// Startup.cs
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // Các middleware khác...
    
    app.UseHttpsRedirection();
    
    // Thêm HSTS - HTTP Strict Transport Security
    app.UseHsts();
    
    // Các middleware khác...
}

// Có thể cấu hình thêm trong appsettings.json
"HttpsRedirection": {
    "RedirectStatusCode": 307,
    "HttpsPort": 443
},
"Hsts": {
    "Enabled": true,
    "MaxAge": 30, // Thời gian tính bằng ngày
    "IncludeSubDomains": true,
    "Preload": true
}
```

### 2.3 Giới hạn số lần đăng nhập thất bại

- **Yêu cầu**: Chống brute force attack bằng cách giới hạn số lần thử
- **Cách triển khai**:
  - Sử dụng IDistributedCache để theo dõi
  - Khóa tài khoản tạm thời khi vượt quá ngưỡng
- **Ví dụ**:

```csharp
// Trong AuthService hoặc LoginController
public async Task<IActionResult> Login(LoginViewModel model)
{
    // Kiểm tra số lần đăng nhập thất bại
    string loginAttemptsKey = $"login-attempts:{model.Username}";
    string ipAttemptsKey = $"login-attempts-ip:{HttpContext.Connection.RemoteIpAddress}";
    
    int userAttempts = await GetLoginAttempts(loginAttemptsKey);
    int ipAttempts = await GetLoginAttempts(ipAttemptsKey);
    
    if (userAttempts >= 5 || ipAttempts >= 10)
    {
        // Khóa tạm thời
        return BadRequest(new { error = "Tài khoản tạm thời bị khóa do đăng nhập thất bại nhiều lần. Vui lòng thử lại sau 15 phút." });
    }
    
    // Xác thực người dùng
    var result = await _signInManager.PasswordSignInAsync(model.Username, model.Password, model.RememberMe, lockoutOnFailure: true);
    
    if (!result.Succeeded)
    {
        // Tăng số lần thất bại
        await IncrementLoginAttempts(loginAttemptsKey);
        await IncrementLoginAttempts(ipAttemptsKey);
        return BadRequest(new { error = "Thông tin đăng nhập không chính xác" });
    }
    
    // Xóa bộ đếm khi đăng nhập thành công
    await _cache.RemoveAsync(loginAttemptsKey);
    
    // Trả về token và thông tin khác
    return Ok(new { token = GenerateToken(user) });
}

private async Task<int> GetLoginAttempts(string key)
{
    var attemptsStr = await _cache.GetStringAsync(key);
    return attemptsStr != null ? int.Parse(attemptsStr) : 0;
}

private async Task IncrementLoginAttempts(string key)
{
    var attempts = await GetLoginAttempts(key);
    await _cache.SetStringAsync(key, (attempts + 1).ToString(), 
        new DistributedCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(15)
        });
}
```

### 2.4 Cấu hình CORS hợp lý

- **Yêu cầu**: Chỉ cho phép domain tin cậy truy cập API
- **Cách triển khai**:
  - Cấu hình CORS với danh sách origins cụ thể
  - Không sử dụng AllowAnyOrigin() trong production
- **Ví dụ**:

```csharp
// Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddCors(options =>
    {
        options.AddPolicy("TrustedOrigins", policy =>
        {
            policy.WithOrigins(
                "https://trusted-frontend.com",
                "https://admin.mycompany.com")
                .AllowCredentials()
                .WithMethods("GET", "POST", "PUT", "DELETE")
                .WithHeaders("Authorization", "Content-Type");
        });
    });
    
    // Dịch vụ khác...
}

public void Configure(IApplicationBuilder app)
{
    // Các middleware khác...
    
    app.UseCors("TrustedOrigins");
    
    // Các middleware khác...
}
```

## 3. Bảo vệ dữ liệu

### 3.1 Không log dữ liệu nhạy cảm

- **Yêu cầu**: Không ghi log password, token, thông tin cá nhân
- **Cách triển khai**:
  - Sử dụng lớp trung gian để lọc thông tin nhạy cảm
  - Tùy chỉnh Serilog hoặc ILogger
- **Ví dụ**:

```csharp
// SensitiveDataFilter.cs
public class SensitiveDataFilter
{
    private static readonly string[] SensitiveProperties = new[] 
    { 
        "password", "token", "secret", "creditcard", "ssn", "email", "phone"
    };
    
    public static string FilterSensitiveData(string logMessage)
    {
        if (string.IsNullOrEmpty(logMessage))
            return logMessage;
        
        foreach (var prop in SensitiveProperties)
        {
            // Tìm và thay thế các pattern như "password": "abc123"
            var pattern = $"\"{prop}\"\\s*:\\s*\"[^\"]*\"";
            logMessage = Regex.Replace(logMessage, pattern, $"\"{prop}\":\"[REDACTED]\"", RegexOptions.IgnoreCase);
        }
        
        return logMessage;
    }
}

// Trong Program.cs
Log.Logger = new LoggerConfiguration()
    .Enrich.FromLogContext()
    .WriteTo.Console()
    .WriteTo.File(new CustomLogFormatter())
    .CreateLogger();

// CustomLogFormatter.cs
public class CustomLogFormatter : ITextFormatter
{
    public void Format(LogEvent logEvent, TextWriter output)
    {
        // Lọc dữ liệu nhạy cảm trước khi log
        var filteredMessage = SensitiveDataFilter.FilterSensitiveData(logEvent.MessageTemplate.Render(logEvent.Properties));
        
        // Tiếp tục định dạng log bình thường
        output.WriteLine($"[{logEvent.Timestamp:yyyy-MM-dd HH:mm:ss}] [{logEvent.Level}] {filteredMessage}");
    }
}
```

### 3.2 Mã hóa dữ liệu quan trọng

- **Yêu cầu**: Sử dụng AES hoặc RSA để bảo vệ dữ liệu lưu trữ và truyền tải
- **Cách triển khai**:
  - Tạo service mã hóa/giải mã
  - Áp dụng cho dữ liệu nhạy cảm
- **Ví dụ**:

```csharp
// EncryptionService.cs
public interface IEncryptionService
{
    string Encrypt(string plainText);
    string Decrypt(string cipherText);
}

public class AesEncryptionService : IEncryptionService
{
    private readonly byte[] _key;
    private readonly byte[] _iv;
    
    public AesEncryptionService(IConfiguration configuration)
    {
        // Lấy key từ cấu hình hoặc Azure Key Vault
        var encryptionKey = configuration["Encryption:Key"];
        var encryptionIv = configuration["Encryption:IV"];
        
        _key = Convert.FromBase64String(encryptionKey);
        _iv = Convert.FromBase64String(encryptionIv);
    }
    
    public string Encrypt(string plainText)
    {
        using var aes = Aes.Create();
        aes.Key = _key;
        aes.IV = _iv;
        
        var encryptor = aes.CreateEncryptor(aes.Key, aes.IV);
        
        using var memoryStream = new MemoryStream();
        using var cryptoStream = new CryptoStream(memoryStream, encryptor, CryptoStreamMode.Write);
        using (var streamWriter = new StreamWriter(cryptoStream))
        {
            streamWriter.Write(plainText);
        }
        
        return Convert.ToBase64String(memoryStream.ToArray());
    }
    
    public string Decrypt(string cipherText)
    {
        var cipher = Convert.FromBase64String(cipherText);
        
        using var aes = Aes.Create();
        aes.Key = _key;
        aes.IV = _iv;
        
        var decryptor = aes.CreateDecryptor(aes.Key, aes.IV);
        
        using var memoryStream = new MemoryStream(cipher);
        using var cryptoStream = new CryptoStream(memoryStream, decryptor, CryptoStreamMode.Read);
        using var streamReader = new StreamReader(cryptoStream);
        
        return streamReader.ReadToEnd();
    }
}

// Đăng ký dịch vụ trong Startup.cs
services.AddSingleton<IEncryptionService, AesEncryptionService>();

// Sử dụng trong Repository hoặc Service
public class UserRepository
{
    private readonly IEncryptionService _encryptionService;
    
    public UserRepository(IEncryptionService encryptionService)
    {
        _encryptionService = encryptionService;
    }
    
    public async Task SavePersonalInfo(UserInfo userInfo)
    {
        // Mã hóa thông tin nhạy cảm trước khi lưu
        userInfo.SocialSecurityNumber = _encryptionService.Encrypt(userInfo.SocialSecurityNumber);
        userInfo.CreditCardNumber = _encryptionService.Encrypt(userInfo.CreditCardNumber);
        
        // Lưu vào database
        await _dbContext.UserInfos.AddAsync(userInfo);
        await _dbContext.SaveChangesAsync();
    }
}
```

### 3.3 Cache dữ liệu nhạy cảm có thời gian sống ngắn

- **Yêu cầu**: Chỉ giữ cache trong thời gian cần thiết để tránh rò rỉ
- **Cách triển khai**:
  - Cấu hình TTL cho cache
  - Xóa cache khi không còn cần thiết
- **Ví dụ**:

```csharp
// Cấu hình trong Startup.cs
services.AddDistributedMemoryCache(); // Trong development
// Hoặc
services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = Configuration.GetConnectionString("Redis");
    options.InstanceName = "SampleInstance_";
});

// CacheService.cs
public class CacheService : ICacheService
{
    private readonly IDistributedCache _cache;
    
    public CacheService(IDistributedCache cache)
    {
        _cache = cache;
    }
    
    public async Task SetSensitiveDataAsync<T>(string key, T value, int timeToLiveMinutes = 5)
    {
        var options = new DistributedCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(timeToLiveMinutes)
        };
        
        var jsonData = JsonSerializer.Serialize(value);
        await _cache.SetStringAsync(key, jsonData, options);
    }
    
    public async Task<T> GetSensitiveDataAsync<T>(string key)
    {
        var jsonData = await _cache.GetStringAsync(key);
        if (string.IsNullOrEmpty(jsonData))
            return default;
            
        return JsonSerializer.Deserialize<T>(jsonData);
    }
    
    public async Task RemoveSensitiveDataAsync(string key)
    {
        await _cache.RemoveAsync(key);
    }
}

// Sử dụng
public class PaymentService
{
    private readonly ICacheService _cacheService;
    
    public async Task ProcessPayment(string userId, PaymentInfo paymentInfo)
    {
        // Cache thông tin thanh toán với thời gian sống ngắn
        await _cacheService.SetSensitiveDataAsync(
            $"payment_{userId}",
            paymentInfo,
            timeToLiveMinutes: 2); // Chỉ lưu 2 phút
            
        // Xử lý thanh toán
        
        // Xóa cache khi đã xử lý xong
        await _cacheService.RemoveSensitiveDataAsync($"payment_{userId}");
    }
}
```

## 4. Bảo vệ SQL & NoSQL Injection

### 4.1 Luôn dùng parameterized queries

- **Yêu cầu**: Ngăn SQL Injection bằng cách dùng tham số thay vì query động
- **Cách triển khai**:
  - Sử dụng Entity Framework hoặc Dapper
  - Luôn dùng tham số hóa khi tương tác với DB
- **Ví dụ**:

```csharp
// KHÔNG BAO GIỜ làm thế này:
// string query = $"SELECT * FROM Users WHERE Username = '{username}' AND Password = '{password}'";

// Cách đúng với Entity Framework Core
public async Task<User> GetUserByUsername(string username)
{
    return await _dbContext.Users
        .FirstOrDefaultAsync(u => u.Username == username);
}

// Cách đúng với Dapper
public async Task<User> GetUserByUsername(string username)
{
    var parameters = new { Username = username };
    
    const string sql = "SELECT * FROM Users WHERE Username = @Username";
    
    using var connection = new SqlConnection(_connectionString);
    return await connection.QueryFirstOrDefaultAsync<User>(sql, parameters);
}
```

### 4.2 Không bao giờ truyền SQL query trực tiếp từ client

- **Yêu cầu**: Không cho phép client gửi truy vấn SQL thô đến server
- **Cách triển khai**:
  - Tạo các endpoint API cụ thể
  - Xác thực và lọc dữ liệu đầu vào
- **Ví dụ**:

```csharp
// Thiết kế API an toàn
[HttpGet("search")]
public async Task<IActionResult> SearchProducts([FromQuery] ProductSearchModel model)
{
    // Giới hạn và kiểm tra tham số
    if (model.PageSize > 100)
        model.PageSize = 100;
        
    // Sử dụng specification pattern cho truy vấn
    var specification = new ProductSpecification(model);
    var products = await _repository.ListAsync(specification);
    
    return Ok(products);
}

// Specification.cs
public class ProductSpecification : BaseSpecification<Product>
{
    public ProductSpecification(ProductSearchModel model)
    {
        // Xây dựng truy vấn an toàn dựa trên tham số đã kiểm tra
        if (!string.IsNullOrEmpty(model.Category))
            AddCriteria(p => p.Category == model.Category);
            
        if (!string.IsNullOrEmpty(model.Name))
            AddCriteria(p => p.Name.Contains(model.Name));
            
        if (model.MinPrice.HasValue)
            AddCriteria(p => p.Price >= model.MinPrice.Value);
            
        // Phân trang an toàn
        ApplyPaging((model.Page - 1) * model.PageSize, model.PageSize);
        
        // Sắp xếp
        if (model.OrderBy == "price")
            ApplyOrderBy(p => p.Price);
        else
            ApplyOrderBy(p => p.Name);
    }
}
```

### 4.3 Với MongoDB, không cho phép client gửi raw query

- **Yêu cầu**: Tránh NoSQL Injection bằng cách validate dữ liệu đầu vào
- **Cách triển khai**:
  - Xác thực và lọc tham số truy vấn
  - Sử dụng typed builders thay vì raw queries
- **Ví dụ**:

```csharp
// MongoDbRepository.cs
public class ProductRepository : IProductRepository
{
    private readonly IMongoCollection<Product> _products;
    
    public ProductRepository(IMongoClient client, IConfiguration config)
    {
        var database = client.GetDatabase(config["MongoDB:DatabaseName"]);
        _products = database.GetCollection<Product>(config["MongoDB:CollectionName"]);
    }
    
    public async Task<IEnumerable<Product>> SearchProducts(ProductSearchModel model)
    {
        // Xây dựng filter an toàn
        var filterBuilder = Builders<Product>.Filter;
        var filter = filterBuilder.Empty;
        
        if (!string.IsNullOrEmpty(model.Category))
        {
            // Validate input trước khi sử dụng
            if (IsValidCategory(model.Category))
                filter &= filterBuilder.Eq(p => p.Category, model.Category);
        }
        
        if (!string.IsNullOrEmpty(model.Name))
        {
            var sanitizedName = SanitizeInput(model.Name);
            filter &= filterBuilder.Regex(p => p.Name, new BsonRegularExpression(sanitizedName, "i"));
        }
        
        if (model.MinPrice.HasValue)
            filter &= filterBuilder.Gte(p => p.Price, model.MinPrice.Value);
            
        // Phân trang và sắp xếp an toàn
        var sort = Builders<Product>.Sort.Ascending(p => p.Name);
        if (model.OrderBy == "price")
            sort = Builders<Product>.Sort.Ascending(p => p.Price);
            
        return await _products
            .Find(filter)
            .Sort(sort)
            .Skip((model.Page - 1) * model.PageSize)
            .Limit(model.PageSize)
            .ToListAsync();
    }
    
    // Phương thức validate input
    private bool IsValidCategory(string category)
    {
        var validCategories = new[] { "electronics", "clothing", "books" };
        return validCategories.Contains(category.ToLower());
    }
    
    private string SanitizeInput(string input)
    {
        // Loại bỏ các ký tự đặc biệt có thể gây lỗi
        return Regex.Replace(input, @"[^\w\s]", "");
    }
}
```

## 5. Bảo vệ chống XSS & CSRF

### 5.1 XSS: Escape output khi render dữ liệu từ user

- **Yêu cầu**: Không hiển thị HTML không được kiểm soát từ input người dùng
- **Cách triển khai**:
  - Luôn escape dữ liệu trong API responses
  - Sử dụng thư viện như HtmlSanitizer
- **Ví dụ**:

```csharp
// ApiResponseHelper.cs
public static class ApiResponseHelper
{
    public static string SanitizeContent(string content)
    {
        if (string.IsNullOrEmpty(content))
            return content;
            
        // Sử dụng thư viện HtmlSanitizer để lọc nội dung
        var sanitizer = new HtmlSanitizer();
        return sanitizer.Sanitize(content);
    }
}

// CommentController.cs
[HttpGet("comments")]
public async Task<IActionResult> GetComments(int postId)
{
    var comments = await _commentService.GetCommentsByPostId(postId);
    
    // Escape nội dung trước khi trả về
    var sanitizedComments = comments.Select(c => new
    {
        c.Id,
        c.UserId,
        c.CreatedAt,
        Content = ApiResponseHelper.SanitizeContent(c.Content)
    });
    
    return Ok(sanitizedComments);
}

// DTO trước khi serialize thành JSON
public class CommentDto
{
    public int Id { get; set; }
    public string UserId { get; set; }
    public DateTime CreatedAt { get; set; }
    
    // Tự động sanitize khi gán giá trị
    private string _content;
    public string Content
    {
        get => _content;
        set => _content = ApiResponseHelper.SanitizeContent(value);
    }
}

### 5.2 CSRF: Dùng AntiForgeryToken hoặc SameSite Cookies

- **Yêu cầu**: Bảo vệ API và form khỏi Cross-Site Request Forgery attack
- **Cách triển khai**:
  - Sử dụng AntiForgeryToken trong ASP.NET MVC
  - Cấu hình SameSite cho cookies
- **Ví dụ**:

```csharp
// Trong controller MVC
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> UpdateProfile(ProfileViewModel model)
{
    if (!ModelState.IsValid)
        return View(model);
        
    // Cập nhật profile
    
    return RedirectToAction("Index", "Home");
}

// Trong Startup.cs - Cấu hình cookies
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.Cookie.HttpOnly = true;
        options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
        options.Cookie.SameSite = SameSiteMode.Strict;
    });

// Bảo vệ API Web với AutoValidateAntiforgeryToken
[AutoValidateAntiforgeryToken]
public class PaymentApiController : ApiController
{
    [HttpPost]
    public async Task<IActionResult> ProcessPayment(PaymentModel model)
    {
        // Xử lý thanh toán
        return Ok();
    }
}
```

## 6. Logging & Monitoring

### 6.1 Dùng Serilog hoặc ILogger để log hành vi bất thường

- **Yêu cầu**: Ghi nhận các hành vi đáng ngờ để phân tích bảo mật
- **Cách triển khai**:
  - Cấu hình Serilog với nhiều sink
  - Log đủ thông tin để phân tích sự cố
- **Ví dụ**:

```csharp
// Program.cs
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .UseSerilog((context, services, configuration) => configuration
            .ReadFrom.Configuration(context.Configuration)
            .ReadFrom.Services(services)
            .Enrich.FromLogContext()
            .WriteTo.Console()
            .WriteTo.File("logs/app-.log", rollingInterval: RollingInterval.Day)
            .WriteTo.Seq("http://seq-server:5341")
        )
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });

// SecurityService.cs
public class SecurityService
{
    private readonly ILogger<SecurityService> _logger;
    
    public SecurityService(ILogger<SecurityService> logger)
    {
        _logger = logger;
    }
    
    public async Task<bool> ValidateRequest(HttpContext context, string userId)
    {
        if (IsUnusualRequest(context))
        {
            _logger.LogWarning(
                "Phát hiện hành vi đáng ngờ: UserId={UserId}, IP={IP}, UserAgent={UserAgent}",
                userId,
                context.Connection.RemoteIpAddress,
                context.Request.Headers["User-Agent"]);
                
            return false;
        }
        
        return true;
    }
    
    private bool IsUnusualRequest(HttpContext context)
    {
        // Logic phát hiện hành vi bất thường
        return false;
    }
}
```

### 6.2 Không log thông tin nhạy cảm

- **Yêu cầu**: Đảm bảo log không chứa dữ liệu cá nhân hoặc thông tin bí mật
- **Cách triển khai**:
  - Sử dụng destructuring và redaction
  - Cấu hình Serilog Destructuring Options
- **Ví dụ**:

```csharp
// Program.cs - Cấu hình log không chứa dữ liệu nhạy cảm
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .UseSerilog((context, services, configuration) => configuration
            .Destructure.ByTransforming<UserDto>(user => new
            {
                user.Id,
                user.Username,
                Email = "***@***", // Ẩn email
                HasPassword = !string.IsNullOrEmpty(user.Password),
                Password = "******" // Ẩn mật khẩu
            })
            .Destructure.With(new RedactingDestructuringPolicy())
        );

// RedactingDestructuringPolicy.cs
public class RedactingDestructuringPolicy : IDestructuringPolicy
{
    private static readonly HashSet<string> SensitiveProperties = new(StringComparer.OrdinalIgnoreCase)
    {
        "password", "secret", "token", "apikey", "connectionstring",
        "creditcard", "ssn", "socialsecuritynumber", "email", "phone"
    };

    public bool TryDestructure(object value, ILogEventPropertyValueFactory propertyValueFactory,
        out LogEventPropertyValue result)
    {
        if (value is not IEnumerable<KeyValuePair<string, object>> properties)
        {
            result = null;
            return false;
        }

        var redactedProperties = new Dictionary<string, LogEventPropertyValue>();
        
        foreach (var kvp in properties)
        {
            var propertyName = kvp.Key;
            
            if (SensitiveProperties.Contains(propertyName))
            {
                redactedProperties.Add(propertyName, 
                    propertyValueFactory.CreatePropertyValue("[REDACTED]"));
            }
            else
            {
                redactedProperties.Add(propertyName, 
                    propertyValueFactory.CreatePropertyValue(kvp.Value));
            }
        }

        result = new DictionaryValue(redactedProperties);
        return true;
    }
}
```

### 6.3 Tích hợp hệ thống giám sát bảo mật

- **Yêu cầu**: Dùng ELK Stack, Application Insights để theo dõi hoạt động API
- **Cách triển khai**:
  - Cấu hình Application Insights
  - Bắt và theo dõi các sự kiện bảo mật
- **Ví dụ**:

```csharp
// Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    // Kích hoạt Application Insights
    services.AddApplicationInsightsTelemetry(Configuration["ApplicationInsights:InstrumentationKey"]);
    
    // Thêm telemetry processor để giám sát các yêu cầu
    services.AddSingleton<ITelemetryInitializer, SecurityTelemetryInitializer>();
    
    // Dịch vụ khác...
}

// SecurityTelemetryInitializer.cs
public class SecurityTelemetryInitializer : ITelemetryInitializer
{
    public void Initialize(ITelemetry telemetry)
    {
        if (!(telemetry is RequestTelemetry requestTelemetry))
            return;
            
        // Thêm thông tin giám sát bảo mật
        requestTelemetry.Properties["Environment"] = Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT");
        
        // Kiểm tra mã trạng thái HTTP
        if (requestTelemetry.ResponseCode == "401" || requestTelemetry.ResponseCode == "403")
        {
            requestTelemetry.Properties["SecurityEvent"] = "AuthenticationFailure";
        }
    }
}

// Middleware giám sát và cảnh báo
public class SecurityMonitoringMiddleware
{
    private readonly RequestDelegate _next;
    private readonly TelemetryClient _telemetryClient;
    
    public SecurityMonitoringMiddleware(RequestDelegate next, TelemetryClient telemetryClient)
    {
        _next = next;
        _telemetryClient = telemetryClient;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            // Giám sát số lượng request từ một IP
            TrackRequestsPerIp(context);
            
            await _next(context);
            
            // Theo dõi mã trạng thái HTTP
            if (context.Response.StatusCode == StatusCodes.Status429TooManyRequests)
            {
                _telemetryClient.TrackEvent("RateLimitExceeded", new Dictionary<string, string>
                {
                    ["IP"] = context.Connection.RemoteIpAddress.ToString(),
                    ["Path"] = context.Request.Path
                });
            }
        }
        catch (Exception ex)
        {
            _telemetryClient.TrackException(ex, new Dictionary<string, string>
            {
                ["IP"] = context.Connection.RemoteIpAddress.ToString(),
                ["Path"] = context.Request.Path
            });
            
            throw;
        }
    }
    
    private void TrackRequestsPerIp(HttpContext context)
    {
        var ip = context.Connection.RemoteIpAddress.ToString();
        _telemetryClient.TrackMetric("RequestsPerIp", 1, new Dictionary<string, string>
        {
            ["IP"] = ip
        });
    }
}
```

## 7. Secure Coding Practices

### 7.1 Không hardcode API keys, passwords, connection strings

- **Yêu cầu**: Sử dụng biến môi trường hoặc Azure Key Vault để bảo mật thông tin
- **Cách triển khai**:
  - Sử dụng User Secrets trong development
  - Sử dụng Azure Key Vault trong production
- **Ví dụ**:

```csharp
// Program.cs
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureAppConfiguration((context, config) =>
        {
            var builtConfig = config.Build();
            
            // Chỉ sử dụng Key Vault trong production
            if (context.HostingEnvironment.IsProduction())
            {
                var keyVaultEndpoint = builtConfig["KeyVault:Endpoint"];
                if (!string.IsNullOrEmpty(keyVaultEndpoint))
                {
                    // Kết nối với Azure Key Vault
                    var azureServiceTokenProvider = new AzureServiceTokenProvider();
                    var keyVaultClient = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(
                        azureServiceTokenProvider.KeyVaultTokenCallback));
                        
                    config.AddAzureKeyVault(keyVaultEndpoint, keyVaultClient, new DefaultKeyVaultSecretManager());
                }
            }
            else
            {
                // Trong development, sử dụng User Secrets
                config.AddUserSecrets<Program>();
            }
        })
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });

// Startup.cs - Lấy và sử dụng cấu hình an toàn
public class Startup
{
    private readonly IConfiguration _configuration;
    
    public Startup(IConfiguration configuration)
    {
        _configuration = configuration;
    }
    
    public void ConfigureServices(IServiceCollection services)
    {
        // Lấy connection string từ cấu hình an toàn
        services.AddDbContext<ApplicationDbContext>(options =>
            options.UseSqlServer(_configuration.GetConnectionString("DefaultConnection")));
            
        // Thêm API client với key bảo mật
        services.AddHttpClient("ExternalApi", client =>
        {
            client.BaseAddress = new Uri(_configuration["ExternalApi:BaseUrl"]);
            client.DefaultRequestHeaders.Add("X-Api-Key", _configuration["ExternalApi:ApiKey"]);
        });
    }
}
```

### 7.2 Cập nhật dependencies thường xuyên

- **Yêu cầu**: Giảm thiểu rủi ro từ lỗ hổng bảo mật của thư viện bên thứ ba
- **Cách triển khai**:
  - Sử dụng công cụ kiểm tra lỗ hổng
  - Tự động cập nhật với GitHub Dependabot
- **Ví dụ**:

```csharp
// Tạo file .github/dependabot.yml trong repository
version: 2
updates:
  - package-ecosystem: "nuget"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
    target-branch: "develop"
    labels:
      - "dependencies"
      - "security"

// Kiểm tra lỗ hổng trong pipeline CI/CD (azure-pipelines.yml)
steps:
- task: DotNetCoreCLI@2
  displayName: 'dotnet restore'
  inputs:
    command: 'restore'
    projects: '**/*.csproj'

- script: |
    dotnet tool install --global dotnet-ossindex
    dotnet-ossindex -p $(Build.SourcesDirectory)/src/MyProject.csproj
  displayName: 'Check for vulnerable dependencies'
  continueOnError: true

// Thêm audit log khi cập nhật dependency
public class DependencyUpdateAuditMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger _logger;
    
    public DependencyUpdateAuditMiddleware(RequestDelegate next, ILogger<DependencyUpdateAuditMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        // Log thông tin phiên bản các thư viện quan trọng
        var assemblyVersions = AppDomain.CurrentDomain.GetAssemblies()
            .Where(a => !a.IsDynamic)
            .Select(a => new 
            { 
                Name = a.GetName().Name,
                Version = a.GetName().Version.ToString() 
            })
            .OrderBy(a => a.Name);
            
        _logger.LogInformation("Dependency versions: {@DependencyVersions}", assemblyVersions);
        
        await _next(context);
    }
}
```

### 7.3 Dùng Security Headers

- **Yêu cầu**: Thêm `Content-Security-Policy`, `X-Frame-Options`, `X-Content-Type-Options`
- **Cách triển khai**:
  - Tạo middleware thêm security headers
  - Cấu hình CSP phù hợp với ứng dụng
- **Ví dụ**:

```csharp
// SecurityHeadersMiddleware.cs
public class SecurityHeadersMiddleware
{
    private readonly RequestDelegate _next;
    
    public SecurityHeadersMiddleware(RequestDelegate next)
    {
        _next = next;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        // Thêm security headers
        
        // Không cho phép iframe từ domain khác
        context.Response.Headers.Add("X-Frame-Options", "DENY");
        
        // Ngăn chặn MIME-sniffing
        context.Response.Headers.Add("X-Content-Type-Options", "nosniff");
        
        // Strict Transport Security
        context.Response.Headers.Add("Strict-Transport-Security", "max-age=31536000; includeSubDomains");
        
        // Content Security Policy
        context.Response.Headers.Add(
            "Content-Security-Policy",
            "default-src 'self'; " +
            "script-src 'self' https://trusted.cdn.com; " +
            "style-src 'self' https://trusted.cdn.com; " +
            "img-src 'self' data:; " +
            "font-src 'self'; " +
            "connect-src 'self' https://api.myservice.com; " +
            "frame-ancestors 'none'; " +
            "form-action 'self';"
        );
        
        // Không gửi Referer header khi từ HTTPS -> HTTP
        context.Response.Headers.Add("Referrer-Policy", "strict-origin-when-cross-origin");
        
        // Bảo vệ chống XSS trên trình duyệt cũ
        context.Response.Headers.Add("X-XSS-Protection", "1; mode=block");
        
        await _next(context);
    }
}

// Startup.cs
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // Đăng ký middleware
    app.UseMiddleware<SecurityHeadersMiddleware>();
    
    // Các middleware khác...
}
```