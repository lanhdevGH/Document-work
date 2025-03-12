# Quy tắc Code Convention cho Clean Architecture trong C# và ASP.NET

## Quy tắc chung

| STT | CheckList | Mô tả | Ví dụ |
|-----|-----------|-------|-------|
| 1 | Documentation ngắn gọn, đủ ý, không dài dòng | Viết comment súc tích nhưng vẫn truyền tải đầy đủ thông tin cần thiết | `/// Authenticates user and returns JWT token` thay vì `/// This method is used to authenticate a user by checking their credentials and when successful, it will create and return a JWT token that can be used for subsequent API calls` |
| 2 | Sử dụng XML Documentation (///) cho class, method, property, interface | Dùng cú pháp /// để tạo tài liệu chuẩn, hỗ trợ công cụ như Swagger hoặc IDE | `/// <summary>Manages user authentication</summary>` |
| 3 | Viết bằng tiếng Anh, câu từ đơn giản, tránh mơ hồ | Đảm bảo dễ đọc, dễ hiểu cho mọi thành viên trong team | `/// Creates new user account` thay vì `/// Thực hiện việc tạo người dùng mới` |
| 4 | Cập nhật documentation ngay khi code thay đổi | Giữ tài liệu đồng bộ với code để tránh sai lệch thông tin | Sửa comment khi thay đổi logic method |

## Namespace

| STT | CheckList | Mô tả | Ví dụ |
|-----|-----------|-------|-------|
| 5 | Namespace có dòng mô tả ngắn gọn về mục đích | Giải thích namespace dùng để làm gì | `/// <summary>Contains user management services</summary>` |

## Class

| STT | CheckList | Mô tả | Ví dụ |
|-----|-----------|-------|-------|
| 6 | Class mô tả trách nhiệm chính (Single Responsibility) | Nêu rõ class làm gì | `/// <summary>Manages user operations in Application layer</summary>` |
| 7 | Class ghi rõ thuộc tầng nào (Presentation, Application, Domain, Infrastructure) | Giúp phân biệt vai trò trong Clean Architecture | `/// <summary>Application service for processing order requests</summary>` |
| 21 | Class có ghi chú về dependencies quan trọng | Liệt kê các dependencies chính | `/// <summary>User manager service</summary>` <br> `/// <remarks>Depends on IUserRepository for data access</remarks>` |

## Method

| STT | CheckList | Mô tả | Ví dụ |
|-----|-----------|-------|-------|
| 8 | Method mô tả chức năng chính | Tập trung vào "làm gì" | `/// <summary>Retrieves a user by ID</summary>` |
| 9 | Method có thẻ `<param>` giải thích từng tham số | Mô tả ý nghĩa tham số | `/// <param name="userId">User's unique ID</param>` |
| 10 | Method có thẻ `<returns>` nếu có giá trị trả về | Giải thích giá trị trả về | `/// <returns>User entity or null if not found</returns>` |
| 11 | Method có thẻ `<exception>` nếu có ngoại lệ | Liệt kê ngoại lệ có thể xảy ra | `/// <exception cref="ArgumentException">Thrown when userId is empty</exception>` |
| 22 | Method async/await cần ghi rõ hành vi bất đồng bộ | Giải thích về tính chất bất đồng bộ | `/// <summary>Asynchronously fetches user data from database</summary>` |
| 23 | Method override cần giải thích lý do override | Nêu rõ lý do ghi đè | `/// <summary>Overrides base method to add caching logic</summary>` |

## Property

| STT | CheckList | Mô tả | Ví dụ |
|-----|-----------|-------|-------|
| 12 | Property mô tả ngắn gọn ý nghĩa và vai trò | Nêu rõ ý nghĩa | `/// <summary>Gets or sets the user's full name</summary>` |
| 24 | Property readonly ghi rõ điều kiện khởi tạo | Giải thích cách khởi tạo | `/// <summary>User's unique identifier</summary>` <br> `/// <remarks>Initialized via constructor, immutable</remarks>` |

## Interface

| STT | CheckList | Mô tả | Ví dụ |
|-----|-----------|-------|-------|
| 13 | Interface mô tả mục đích và tầng sử dụng | Giải thích tổng quan | `/// <summary>Defines user operation contract for Application layer</summary>` |
| 14 | Interface không lặp lại chi tiết của method | Chỉ mô tả chung, tránh dư thừa | `/// <summary>Repository contract for user data access</summary>` thay vì mô tả chi tiết từng method |

## Enum

| STT | CheckList | Mô tả | Ví dụ |
|-----|-----------|-------|-------|
| 25 | Enum có mô tả từng giá trị | Giải thích ý nghĩa các giá trị | `/// <summary>User account status</summary>` <br> `/// <remarks>` <br> `/// Active: User can log in normally` <br> `/// Suspended: User temporarily blocked` <br> `/// Deleted: User permanently removed` <br> `/// </remarks>` |

## Lưu ý quan trọng

| STT | CheckList | Mô tả | Ví dụ |
|-----|-----------|-------|-------|
| 15 | Không viết comment thừa thãi | Tránh giải thích điều đã rõ qua tên | Không viết `/// Sets the name` cho `SetName(string name)` |
| 16 | Tập trung vào "Tại sao" thay vì "Làm thế nào" | Nêu lý do tồn tại | `/// Ensures business rules for order approval` thay vì `/// Loops through order items` |
| 17 | Sử dụng `#region` để nhóm code dài, kèm mô tả ngắn | Tổ chức code dễ đọc | `#region Private Methods // Helper validation methods` |
| 26 | Ghi chú code deprecated (nếu có) | Cảnh báo code sắp bỏ | `/// <summary>Creates user account</summary>` <br> `/// <remarks>[Obsolete] Use RegisterUser() instead</remarks>` |
| 27 | Thêm ví dụ sử dụng trong comment (nếu phức tạp) | Minh họa cách dùng | `/// <example>var result = Calculate(5, 10);</example>` |

## Công cụ hỗ trợ

| STT | CheckList | Mô tả | Ví dụ |
|-----|-----------|-------|-------|
| 18 | Dùng Visual Studio để tự động sinh XML Documentation (///) | Tận dụng IDE để tiết kiệm thời gian | Gõ `///` trước một method để VS tự tạo template |
| 19 | Kiểm tra bằng StyleCop/SonarQube để đảm bảo tuân thủ | Dùng công cụ phân tích tĩnh | Cài đặt StyleCop rules SA1600-SA1615 để kiểm tra documentation |
| 20 | Tích hợp Swagger nếu là API để hiển thị tài liệu | Kết hợp XML comments với Swagger | Cấu hình trong Startup.cs để Swagger sử dụng XML comments |
| 28 | Sử dụng Doxygen/DocFX cho dự án lớn | Tạo tài liệu HTML/PDF chuyên nghiệp | Thêm cấu hình DocFX vào project để tạo tài liệu từ XML comments |
| 29 | Kiểm tra XML documentation warnings trong CI/CD | Đảm bảo không có lỗi thiếu comment | Thêm bước kiểm tra XML documentation trong pipeline |
| 30 | Sử dụng GhostDoc để tự động đề xuất comment | Công cụ hỗ trợ generate comment | Cài GhostDoc extension và sử dụng để tạo comment mẫu |

## Cấu trúc Clean Architecture

### Presentation Layer

```csharp
/// <summary>
/// Controller handling user authentication requests.
/// </summary>
/// <remarks>Belongs to Presentation layer</remarks>
[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly IAuthService _authService;

    /// <summary>
    /// Initializes a new instance of AuthController.
    /// </summary>
    /// <param name="authService">Service for authentication operations</param>
    public AuthController(IAuthService authService)
    {
        _authService = authService;
    }

    /// <summary>
    /// Authenticates user and returns JWT token.
    /// </summary>
    /// <param name="request">Login credentials</param>
    /// <returns>JWT token with user info</returns>
    /// <exception cref="UnauthorizedAccessException">When credentials are invalid</exception>
    [HttpPost("login")]
    public async Task<ActionResult<AuthResponse>> Login(LoginRequest request)
    {
        // Implementation
    }
}
```

### Application Layer

```csharp
/// <summary>
/// Service for user authentication operations.
/// </summary>
/// <remarks>
/// Belongs to Application layer.
/// Depends on IUserRepository and IJwtGenerator.
/// </remarks>
public class AuthService : IAuthService
{
    /// <summary>
    /// Authenticates user and generates JWT token.
    /// </summary>
    /// <param name="username">User's username or email</param>
    /// <param name="password">User's password</param>
    /// <returns>Authentication result with token</returns>
    /// <exception cref="NotFoundException">When user doesn't exist</exception>
    /// <exception cref="UnauthorizedAccessException">When password is incorrect</exception>
    public async Task<AuthResult> AuthenticateAsync(string username, string password)
    {
        // Implementation
    }
}
```

### Domain Layer

```csharp
/// <summary>
/// Represents a user account in the system.
/// </summary>
/// <remarks>Domain entity</remarks>
public class User
{
    /// <summary>
    /// User's unique identifier.
    /// </summary>
    /// <remarks>Initialized via constructor, immutable</remarks>
    public Guid Id { get; private set; }

    /// <summary>
    /// Gets or sets the user's email address.
    /// </summary>
    /// <remarks>Must be unique in the system</remarks>
    public string Email { get; set; }

    /// <summary>
    /// Verifies if provided password matches user's password.
    /// </summary>
    /// <param name="password">Password to check</param>
    /// <returns>True if password matches, otherwise false</returns>
    public bool VerifyPassword(string password)
    {
        // Implementation
    }
}
```

### Infrastructure Layer

```csharp
/// <summary>
/// Repository implementation for user data access.
/// </summary>
/// <remarks>
/// Belongs to Infrastructure layer.
/// Implements IUserRepository from Domain layer.
/// </remarks>
public class UserRepository : IUserRepository
{
    /// <summary>
    /// Asynchronously retrieves user by email.
    /// </summary>
    /// <param name="email">User's email address</param>
    /// <returns>User entity or null if not found</returns>
    public async Task<User> GetByEmailAsync(string email)
    {
        // Implementation
    }
}
```
