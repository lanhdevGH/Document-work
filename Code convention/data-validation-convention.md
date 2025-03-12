# Code Convention: Nguyên tắc Validation trong ASP.NET Clean Architecture

## 1. Xác thực dữ liệu gần nguồn nhất

### Mô tả
Luôn kiểm tra tính hợp lệ của dữ liệu càng gần với nguồn nhập liệu càng tốt, thường là ở tầng API hoặc Controller.

### Lý do
- Phát hiện lỗi sớm giúp ngăn chặn dữ liệu không hợp lệ đi sâu vào hệ thống
- Tiết kiệm tài nguyên xử lý
- Bảo vệ Domain Model khỏi trạng thái không hợp lệ

### Ví dụ
```csharp
// Tốt
[ApiController]
public class UserController : ControllerBase
{
    [HttpPost]
    public IActionResult Create(CreateUserRequest request)
    {
        // ASP.NET tự động kiểm tra ModelState.IsValid trước khi vào action
        // nhờ [ApiController]
        var command = new CreateUserCommand(request.Name, request.Email);
        _mediator.Send(command);
        return Ok();
    }
}
```

## 2. Tách biệt Validation và Business Rules

### Mô tả
Validation chỉ kiểm tra tính hợp lệ cơ bản của dữ liệu (định dạng, giới hạn,...), trong khi Business Rules là các quy tắc nghiệp vụ phức tạp hơn.

### Lý do
- Giúp code dễ hiểu và bảo trì hơn
- Tách biệt các khía cạnh khác nhau của kiểm tra dữ liệu
- Dễ dàng thay đổi quy tắc nghiệp vụ mà không ảnh hưởng đến validation

### Ví dụ
```csharp
// Validation - Kiểm tra tính hợp lệ cơ bản
public class CreateOrderRequestValidator : AbstractValidator<CreateOrderRequest>
{
    public CreateOrderRequestValidator()
    {
        RuleFor(x => x.ProductId).NotEmpty();
        RuleFor(x => x.Quantity).GreaterThan(0);
    }
}

// Business Rule - Kiểm tra nghiệp vụ phức tạp
public class CheckProductAvailabilityRule
{
    private readonly IProductRepository _productRepository;
    
    public CheckProductAvailabilityRule(IProductRepository productRepository)
    {
        _productRepository = productRepository;
    }
    
    public bool IsSatisfiedBy(int productId, int quantity)
    {
        var product = _productRepository.GetById(productId);
        return product != null && product.AvailableStock >= quantity;
    }
}
```

## 3. Sử dụng Data Annotation cho validation đơn giản

### Mô tả
Dùng các attribute có sẵn của .NET như [Required], [StringLength], [Range], [EmailAddress] để kiểm tra dữ liệu đơn giản trên DTO hoặc ViewModel.

### Lý do
- Dễ áp dụng, ít code
- Tích hợp sẵn với ASP.NET ModelState
- Tự động tạo các thông báo lỗi mặc định

### Ví dụ
```csharp
public class RegisterUserRequest
{
    [Required(ErrorMessage = "Tên không được để trống")]
    [StringLength(100, MinimumLength = 2, ErrorMessage = "Tên phải từ 2-100 ký tự")]
    public string Name { get; set; }
    
    [Required(ErrorMessage = "Email không được để trống")]
    [EmailAddress(ErrorMessage = "Email không đúng định dạng")]
    public string Email { get; set; }
    
    [Required(ErrorMessage = "Mật khẩu không được để trống")]
    [StringLength(100, MinimumLength = 6, ErrorMessage = "Mật khẩu phải từ 6-100 ký tự")]
    public string Password { get; set; }
}
```

## 4. Dùng FluentValidation cho validation phức tạp

### Mô tả
Khi validation có nhiều điều kiện hoặc quy tắc phức tạp, hãy sử dụng thư viện FluentValidation thay vì Data Annotation.

### Lý do
- Tách biệt logic validation khỏi model
- Validation có thể phức tạp và có điều kiện
- Dễ bảo trì và test hơn
- Code dễ đọc và trực quan hơn

### Ví dụ
```csharp
public class CreateProductValidator : AbstractValidator<CreateProductCommand>
{
    public CreateProductValidator(IProductRepository productRepository)
    {
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Tên sản phẩm không được để trống")
            .MaximumLength(200).WithMessage("Tên sản phẩm không được quá 200 ký tự")
            .MustAsync(async (name, cancellation) => 
                !(await productRepository.ExistsByNameAsync(name)))
            .WithMessage("Tên sản phẩm đã tồn tại");
            
        RuleFor(x => x.Price)
            .GreaterThan(0).WithMessage("Giá phải lớn hơn 0")
            .When(x => x.IsActive);
            
        RuleFor(x => x.CategoryId)
            .MustAsync(async (id, cancellation) => 
                await productRepository.CategoryExistsAsync(id))
            .WithMessage("Danh mục không tồn tại");
    }
}
```

## 5. Không phụ thuộc vào Client-side validation

### Mô tả
Client-side validation chỉ để cải thiện trải nghiệm người dùng, không phải là biện pháp bảo vệ dữ liệu chính.

### Lý do
- Client-side validation có thể bị vô hiệu hóa hoặc bypass
- Không đủ tin cậy để bảo vệ hệ thống
- API có thể được gọi từ nhiều nguồn khác nhau, không chỉ từ client chính thức

### Ví dụ
```csharp
// Luôn kiểm tra dữ liệu ở Backend, dù đã validate ở Frontend
[HttpPost]
public async Task<IActionResult> Register(RegisterUserRequest request)
{
    // Kiểm tra ở server side ngay cả khi client đã có validation
    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }
    
    // Tiến hành xử lý dữ liệu
    var result = await _userService.RegisterAsync(request);
    // ...
}
```

## 6. Xác thực dữ liệu đầu vào ở Application Layer

### Mô tả
Dữ liệu từ API nên được kiểm tra ngay trong Application Layer (Use Case / Service Layer) trước khi được chuyển đến Domain Layer.

### Lý do
- Đảm bảo dữ liệu hợp lệ trước khi thực hiện logic nghiệp vụ
- Bảo vệ Domain Model khỏi dữ liệu không hợp lệ
- Tách biệt trách nhiệm validation khỏi Domain

### Ví dụ
```csharp
public class CreateOrderHandler : IRequestHandler<CreateOrderCommand, Result<int>>
{
    private readonly IValidator<CreateOrderCommand> _validator;
    private readonly IOrderRepository _orderRepository;
    
    public CreateOrderHandler(
        IValidator<CreateOrderCommand> validator,
        IOrderRepository orderRepository)
    {
        _validator = validator;
        _orderRepository = orderRepository;
    }
    
    public async Task<Result<int>> Handle(CreateOrderCommand command, CancellationToken cancellationToken)
    {
        // Validation ở Application Layer
        var validationResult = await _validator.ValidateAsync(command, cancellationToken);
        if (!validationResult.IsValid)
        {
            return Result.Failure<int>(validationResult.Errors
                .Select(e => e.ErrorMessage)
                .ToList());
        }
        
        // Dữ liệu đã valid, tiến hành xử lý domain logic
        var order = new Order(command.CustomerId, command.Items);
        await _orderRepository.AddAsync(order);
        
        return Result.Success(order.Id);
    }
}
```

## 7. Tránh validation trong Controller

### Mô tả
Controller chỉ nên gọi Validation Middleware hoặc kiểm tra ModelState.IsValid, không chứa logic validation thủ công.

### Lý do
- Giữ Controller gọn nhẹ, chỉ làm nhiệm vụ điều hướng
- Dễ dàng tái sử dụng logic validation ở nhiều endpoint
- Tách biệt trách nhiệm giữa các lớp

### Ví dụ
```csharp
// Không tốt
[HttpPost]
public IActionResult Create(UserDto user)
{
    if (string.IsNullOrEmpty(user.Name))
    {
        ModelState.AddModelError("Name", "Tên không được để trống");
    }
    
    if (user.Age < 18)
    {
        ModelState.AddModelError("Age", "Tuổi phải lớn hơn hoặc bằng 18");
    }
    
    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }
    
    // Xử lý tiếp
}

// Tốt
[HttpPost]
public async Task<IActionResult> Create(CreateUserCommand command)
{
    var result = await _mediator.Send(command);
    return result.IsSuccess ? Ok(result.Value) : BadRequest(result.Error);
}
```

## 8. Không dùng Data Annotations trong Domain Model

### Mô tả
Để đảm bảo Domain Model độc lập, không sử dụng Data Annotations trong các Entity. Thay vào đó, hãy dùng Value Objects và validation trong constructor.

### Lý do
- Giữ Domain Model độc lập với infrastructure
- Đảm bảo Entity luôn ở trạng thái hợp lệ
- Tuân thủ nguyên tắc của Domain-Driven Design

### Ví dụ
```csharp
// Không tốt
public class User
{
    [Required]
    [StringLength(100)]
    public string Name { get; set; }
    
    [Required]
    [EmailAddress]
    public string Email { get; set; }
}

// Tốt
public class User
{
    public string Name { get; }
    public Email Email { get; }
    
    public User(string name, string email)
    {
        if (string.IsNullOrWhiteSpace(name))
            throw new DomainException("Tên không được để trống");
            
        Name = name;
        Email = new Email(email); // Email là một Value Object tự validate
    }
}

// Value Object
public class Email
{
    public string Value { get; }
    
    public Email(string value)
    {
        if (string.IsNullOrWhiteSpace(value))
            throw new DomainException("Email không được để trống");
            
        if (!IsValidEmail(value))
            throw new DomainException("Email không đúng định dạng");
            
        Value = value;
    }
    
    private bool IsValidEmail(string email)
    {
        // Logic kiểm tra email
        return Regex.IsMatch(email, @"^[^@\s]+@[^@\s]+\.[^@\s]+$");
    }
}
```

## 9. Chuẩn hóa lỗi validation trả về

### Mô tả
Khi dữ liệu không hợp lệ, API nên trả về lỗi theo format JSON thống nhất.

### Lý do
- Giúp client dễ dàng xử lý lỗi
- Cải thiện trải nghiệm người dùng
- Đảm bảo tính nhất quán trong API

### Ví dụ
```csharp
// Cấu trúc lỗi trả về
public class ValidationErrorResponse
{
    public Dictionary<string, string[]> Errors { get; set; } = new Dictionary<string, string[]>();
}

// Trong controller hoặc middleware
[HttpPost]
public IActionResult Create(CreateProductRequest request)
{
    if (!ModelState.IsValid)
    {
        var errors = ModelState
            .Where(x => x.Value.Errors.Count > 0)
            .ToDictionary(
                kvp => kvp.Key,
                kvp => kvp.Value.Errors.Select(e => e.ErrorMessage).ToArray()
            );
            
        return BadRequest(new ValidationErrorResponse { Errors = errors });
    }
    
    // Xử lý tiếp
}
```

## 10. Dùng Middleware để xử lý lỗi validation tự động

### Mô tả
Cấu hình Middleware để tự động bắt lỗi validation và trả về phản hồi chuẩn mà không cần code thủ công trong Controller.

### Lý do
- Giảm code lặp lại trong các controller
- Đảm bảo xử lý lỗi nhất quán
- Dễ bảo trì và thay đổi format lỗi

### Ví dụ
```csharp
// Trong Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers()
        .ConfigureApiBehaviorOptions(options =>
        {
            options.InvalidModelStateResponseFactory = context =>
            {
                var errors = context.ModelState
                    .Where(e => e.Value.Errors.Count > 0)
                    .ToDictionary(
                        kvp => kvp.Key,
                        kvp => kvp.Value.Errors.Select(e => e.ErrorMessage).ToArray()
                    );

                return new BadRequestObjectResult(new
                {
                    errors = errors
                });
            };
        });
        
    // Cấu hình FluentValidation
    services.AddFluentValidationAutoValidation();
    services.AddValidatorsFromAssembly(Assembly.GetExecutingAssembly());
}
```

## 11. Dùng Custom Validator cho logic kiểm tra đặc thù

### Mô tả
Khi cần kiểm tra phức tạp, hãy tạo Validator riêng thay vì viết trong Controller hoặc Service.

### Lý do
- Tách biệt logic validation phức tạp
- Dễ dàng tái sử dụng
- Dễ kiểm thử

### Ví dụ
```csharp
// Custom validator kiểm tra ID có tồn tại trong DB
public class EntityExistsValidator<TEntity> : IEntityExistsValidator<TEntity>
    where TEntity : class, IEntity
{
    private readonly IRepository<TEntity> _repository;
    
    public EntityExistsValidator(IRepository<TEntity> repository)
    {
        _repository = repository;
    }
    
    public async Task<bool> ExistsAsync(int id, CancellationToken cancellationToken = default)
    {
        return await _repository.ExistsAsync(id, cancellationToken);
    }
}

// Sử dụng trong FluentValidation
public class UpdateProductValidator : AbstractValidator<UpdateProductCommand>
{
    public UpdateProductValidator(IEntityExistsValidator<Product> productValidator)
    {
        RuleFor(x => x.Id)
            .NotEmpty()
            .MustAsync(async (id, cancellation) => 
                await productValidator.ExistsAsync(id, cancellation))
            .WithMessage("Sản phẩm không tồn tại");
            
        // Các rule khác
    }
}
```

## 12. Không ném Exception khi Validation thất bại

### Mô tả
Khi validation không đạt, chỉ trả về HTTP 400 Bad Request, không ném Exception để tránh rủi ro bảo mật và hiệu suất kém.

### Lý do
- Exception nên dành cho lỗi không mong đợi
- Trả về mã lỗi 400 là đúng chuẩn HTTP cho validation lỗi
- Không gây ra log exception không cần thiết

### Ví dụ
```csharp
// Không tốt
public async Task<Product> CreateProductAsync(CreateProductDto dto)
{
    if (string.IsNullOrEmpty(dto.Name))
    {
        throw new ValidationException("Tên sản phẩm không được để trống");
    }
    
    // Xử lý tiếp
}

// Tốt
public async Task<Result<Product>> CreateProductAsync(CreateProductDto dto)
{
    var validator = new CreateProductValidator();
    var validationResult = await validator.ValidateAsync(dto);
    
    if (!validationResult.IsValid)
    {
        return Result.Failure<Product>(validationResult.Errors
            .Select(x => x.ErrorMessage)
            .ToList());
    }
    
    // Xử lý tiếp
    var product = new Product(dto.Name, dto.Price);
    await _repository.AddAsync(product);
    
    return Result.Success(product);
}
```

## 13. Tạo lớp Validation Service riêng nếu cần

### Mô tả
Nếu một model có nhiều validation phức tạp, tách riêng thành lớp ValidationService để dễ tái sử dụng và kiểm soát.

### Lý do
- Tách biệt logic validation phức tạp
- Dễ tái sử dụng giữa các handler
- Dễ test hơn

### Ví dụ
```csharp
// Validation Service
public interface IUserValidationService
{
    Task<ValidationResult> ValidateRegistrationAsync(RegisterUserCommand command);
    Task<ValidationResult> ValidateUpdateAsync(UpdateUserCommand command);
}

public class UserValidationService : IUserValidationService
{
    private readonly IUserRepository _userRepository;
    
    public UserValidationService(IUserRepository userRepository)
    {
        _userRepository = userRepository;
    }
    
    public async Task<ValidationResult> ValidateRegistrationAsync(RegisterUserCommand command)
    {
        var errors = new List<string>();
        
        // Kiểm tra email đã tồn tại
        if (await _userRepository.ExistsByEmailAsync(command.Email))
        {
            errors.Add("Email đã được sử dụng");
        }
        
        // Kiểm tra username đã tồn tại
        if (await _userRepository.ExistsByUsernameAsync(command.Username))
        {
            errors.Add("Username đã được sử dụng");
        }
        
        return errors.Any() 
            ? ValidationResult.Fail(errors) 
            : ValidationResult.Success();
    }
    
    public async Task<ValidationResult> ValidateUpdateAsync(UpdateUserCommand command)
    {
        // Logic kiểm tra cập nhật
    }
}
```

## 14. Dùng Dependency Injection để inject Validator

### Mô tả
Khi sử dụng FluentValidation, đăng ký Validator trong DI Container để dễ dàng sử dụng.

### Lý do
- Tuân thủ nguyên tắc Dependency Inversion
- Dễ dàng mock validator trong unit test
- Tự động inject dependency cho validator

### Ví dụ
```csharp
// Trong Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    // Đăng ký tất cả validator trong assembly
    services.AddValidatorsFromAssemblyContaining<Program>();
    
    // Hoặc đăng ký từng validator cụ thể
    services.AddScoped<IValidator<CreateProductCommand>, CreateProductValidator>();
    services.AddScoped<IValidator<UpdateProductCommand>, UpdateProductValidator>();
    
    // Đăng ký validation service
    services.AddScoped<IUserValidationService, UserValidationService>();
}

// Sử dụng trong handler
public class CreateProductHandler : IRequestHandler<CreateProductCommand, Result<int>>
{
    private readonly IValidator<CreateProductCommand> _validator;
    
    public CreateProductHandler(IValidator<CreateProductCommand> validator)
    {
        _validator = validator;
    }
    
    public async Task<Result<int>> Handle(CreateProductCommand command, CancellationToken cancellationToken)
    {
        var validationResult = await _validator.ValidateAsync(command, cancellationToken);
        if (!validationResult.IsValid)
        {
            return Result.Failure<int>(validationResult.Errors
                .Select(e => e.ErrorMessage)
                .ToList());
        }
        
        // Xử lý tiếp
    }
}
```

## 15. Tạo unit test cho Validator

### Mô tả
Mỗi Validator nên có unit test để đảm bảo tất cả điều kiện kiểm tra dữ liệu đều hoạt động đúng.

### Lý do
- Đảm bảo validator hoạt động chính xác
- Phát hiện lỗi sớm khi thay đổi logic
- Làm tài liệu cho các điều kiện validation

### Ví dụ
```csharp
[TestClass]
public class CreateProductValidatorTests
{
    private CreateProductValidator _validator;
    private Mock<IProductRepository> _repositoryMock;
    
    [TestInitialize]
    public void Setup()
    {
        _repositoryMock = new Mock<IProductRepository>();
        _validator = new CreateProductValidator(_repositoryMock.Object);
    }
    
    [TestMethod]
    public async Task Should_Fail_When_Name_Is_Empty()
    {
        // Arrange
        var command = new CreateProductCommand { Price = 100 };
        
        // Act
        var result = await _validator.ValidateAsync(command);
        
        // Assert
        Assert.IsFalse(result.IsValid);
        Assert.IsTrue(result.Errors.Any(e => e.PropertyName == "Name"));
    }
    
    [TestMethod]
    public async Task Should_Fail_When_Product_Name_Already_Exists()
    {
        // Arrange
        var command = new CreateProductCommand 
        { 
            Name = "Existing Product", 
            Price = 100 
        };
        
        _repositoryMock.Setup(r => r.ExistsByNameAsync("Existing Product"))
            .ReturnsAsync(true);
            
        // Act
        var result = await _validator.ValidateAsync(command);
        
        // Assert
        Assert.IsFalse(result.IsValid);
        Assert.IsTrue(result.Errors.Any(e => e.PropertyName == "Name" && 
                                          e.ErrorMessage.Contains("đã tồn tại")));
    }
}
```

## 16. Kiểm tra tính duy nhất khi cần

### Mô tả
Nếu một trường (như Email, SKU) cần duy nhất, kiểm tra trước khi lưu vào Database.

### Lý do
- Tránh lỗi unique constraint ở DB
- Trả về thông báo lỗi thân thiện với người dùng
- Đảm bảo tính toàn vẹn dữ liệu

### Ví dụ
```csharp
public class CreateUserValidator : AbstractValidator<CreateUserCommand>
{
    private readonly IUserRepository _userRepository;
    
    public CreateUserValidator(IUserRepository userRepository)
    {
        _userRepository = userRepository;
        
        RuleFor(x => x.Email)
            .NotEmpty().WithMessage("Email không được để trống")
            .EmailAddress().WithMessage("Email không đúng định dạng")
            .MustAsync(async (email, cancellation) => 
                !(await _userRepository.ExistsByEmailAsync(email)))
            .WithMessage("Email đã được sử dụng");
            
        RuleFor(x => x.Username)
            .NotEmpty().WithMessage("Username không được để trống")
            .MustAsync(async (username, cancellation) => 
                !(await _userRepository.ExistsByUsernameAsync(username)))
            .WithMessage("Username đã được sử dụng");
    }
}
```

## 17. Sử dụng Regex cho kiểm tra định dạng phức tạp

### Mô tả
Nếu cần kiểm tra định dạng (ví dụ: số điện thoại, mã khách hàng), dùng Regex để đảm bảo dữ liệu hợp lệ.

### Lý do
- Kiểm tra chính xác các định dạng phức tạp
- Đảm bảo tính nhất quán trong dữ liệu
- Tái sử dụng được các pattern

### Ví dụ
```csharp
public class PhoneNumberValidator : AbstractValidator<UpdatePhoneNumberCommand>
{
    private const string PhonePattern = @"^(0|\+84)(\d{9,10})$";
    
    public PhoneNumberValidator()
    {
        RuleFor(x => x.PhoneNumber)
            .NotEmpty().WithMessage("Số điện thoại không được để trống")
            .Matches(PhonePattern).WithMessage("Số điện thoại không đúng định dạng Việt Nam");
    }
}

// Hoặc tạo custom validator
public static class CustomValidators
{
    public static IRuleBuilderOptions<T, string> PhoneNumber<T>(
        this IRuleBuilder<T, string> ruleBuilder)
    {
        return ruleBuilder
            .NotEmpty().WithMessage("Số điện thoại không được để trống")
            .Matches(@"^(0|\+84)(\d{9,10})$").WithMessage("Số điện thoại không đúng định dạng Việt Nam");
    }
    
    public static IRuleBuilderOptions<T, string> CustomerCode<T>(
        this IRuleBuilder<T, string> ruleBuilder)
    {
        return ruleBuilder
            .NotEmpty().WithMessage("Mã khách hàng không được để trống")
            .Matches(@"^KH\d{6}$").WithMessage("Mã khách hàng phải có dạng KH + 6 số");
    }
}

// Sử dụng
public class CustomerValidator : AbstractValidator<CustomerDto>
{
    public CustomerValidator()
    {
        RuleFor(x => x.PhoneNumber).PhoneNumber();
        RuleFor(x => x.CustomerCode).CustomerCode();
    }
}
```

## 18. Hạn chế validation bằng Exception

### Mô tả
Không sử dụng Exception (throw new Exception) để kiểm tra dữ liệu đầu vào, mà nên dùng cơ chế validation hợp lý.

### Lý do
- Exception nên dùng cho lỗi không mong đợi, không phải lỗi validation
- Sử dụng Exception cho validation gây hiệu năng kém
- Phù hợp với thiết kế RESTful API

### Ví dụ
```csharp
// Không tốt
public void CreateUser(UserDto dto)
{
    if (string.IsNullOrEmpty(dto.Email))
    {
        throw new ArgumentException("Email không được để trống");
    }
    
    if (_userRepository.ExistsByEmail(dto.Email))
    {
        throw new InvalidOperationException("Email đã tồn tại");
    }
    
    // Xử lý tiếp
}

// Tốt
public Result<User> CreateUser(UserDto dto)
{
    var validator = new CreateUserValidator(_userRepository);
    var validationResult = validator.Validate(dto);
    
    if (!validationResult.IsValid)
    {
        return Result.Failure<User>(validationResult.Errors
            .Select(e => e.ErrorMessage)
            .ToList());
    }
    
    var user = new User(dto.Name, dto.Email);
    _userRepository.Add(user);
    
    return Result.Success(user);
}
```

## 19. Kiểm tra giá trị null hoặc empty đúng cách

### Mô tả
Sử dụng .NotNull().NotEmpty() trong FluentValidation thay vì != null để kiểm tra chính xác.

### Lý do
- Kiểm tra đầy đủ cả null và empty string
- Tránh lỗi NullReferenceException
- Đảm bảo tính nhất quán trong validation

### Ví dụ
```csharp
// Không tốt - có thể bỏ sót trường hợp string empty
public bool IsValid(string value)
{
    return value != null;
}

// Tốt
public class ProductValidator : AbstractValidator<Product>
{
    public ProductValidator()
    {
        RuleFor(x => x.Name)
            .NotNull().WithMessage("Tên sản phẩm không được null")
            .NotEmpty().WithMessage("Tên sản phẩm không được để trống");
            
        // Hoặc gộp lại
        RuleFor(x => x.Description)
            .NotNull().NotEmpty().WithMessage("Mô tả sản phẩm không được để trống");
    }
}
```

## 20. Không chặn validation lỗi của nhiều field cùng lúc

### Mô tả
Nếu có nhiều lỗi validation, API nên trả về tất cả lỗi thay vì chỉ dừng lại ở lỗi đầu tiên.

### Lý do
- Giúp người dùng sửa tất cả lỗi một lần
- Tránh trải nghiệm người dùng kém khi phải sửa từng lỗi một
- Tiết kiệm thời gian và request không cần thiết

### Ví dụ
```csharp
// Trong ConfigureServices
services.AddFluentValidationAutoValidation(config =>
{
    // Đảm bảo tất cả lỗi được thu thập
    config.ValidatorOptions.CascadeMode = CascadeMode.Continue;
});

// Trong validator
public class ProductValidator : AbstractValidator<Product>
{
    public ProductValidator()
    {
        // CascadeMode.Continue đảm bảo tất cả rule được kiểm tra
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Tên không được để trống")
            .MaximumLength(200).WithMessage("Tên không được quá 200 ký tự");
            
        RuleFor(x => x.Price)
            .GreaterThan(0).WithMessage("Giá phải lớn hơn 0");
            
        RuleFor(x => x.Description)
            .NotEmpty().WithMessage("Mô tả không được để trống");
    }
}
```

## 21. Xác thực dữ liệu liên quan giữa các field

### Mô tả
Nếu có các field phụ thuộc vào nhau (ví dụ: StartDate phải nhỏ hơn EndDate), kiểm tra logic này trong Validator.

### Lý do
- Đảm bảo tính toàn vẹn của dữ liệu liên quan
- Trả về lỗi cụ thể giúp người dùng hiểu vấn đề
- Tránh lỗi logic trong hệ thống

### Ví dụ
```csharp
public class DateRangeValidator : AbstractValidator<DateRangeRequest>
{
    public DateRangeValidator()
    {
        RuleFor(x => x.StartDate)
            .NotEmpty().WithMessage("Ngày bắt đầu không được để trống");
            
        RuleFor(x => x.EndDate)
            .NotEmpty().WithMessage("Ngày kết thúc không được để trống")
            .GreaterThanOrEqualTo(x => x.StartDate)
                .WithMessage("Ngày kết thúc phải lớn hơn hoặc bằng ngày bắt đầu");
                
        // Kiểm tra liên quan nhiều field
        RuleFor(x => x)
            .Must(x => IsValidDateRange(x.StartDate, x.EndDate, x.MaxDays))
            .WithMessage("Khoảng thời gian không được vượt quá {MaxDays} ngày")
            .When(x => x.StartDate != null && x.EndDate != null);
    }
    
    private bool IsValidDateRange(DateTime start, DateTime end, int maxDays)
    {
        return (end - start).TotalDays <= maxDays;
    }
}
```

## 22. Không ghi đè Validation mặc định của Framework

### Mô tả
Nếu sử dụng ModelState hoặc FluentValidation, không nên ghi đè cách thức xử lý lỗi mặc định trừ khi có lý do chính đáng.

### Lý do
- Tận dụng cơ chế validation có sẵn và được tối ưu
- Giảm thiểu code tùy chỉnh
- Đảm bảo nhất quán trong xử lý lỗi

### Ví dụ
```csharp
// Không tốt - Ghi đè toàn bộ cơ chế validation mặc định
public class CustomModelBinderProvider : IModelBinderProvider
{
    public IModelBinder GetBinder(ModelBinderProviderContext context)
    {
        // Custom logic ghi đè validation mặc định
    }
}

// Tốt - Chỉ tùy chỉnh format lỗi, không ghi đè cơ chế
services.AddControllers()
    .ConfigureApiBehaviorOptions(options =>
    {
        options.InvalidModelStateResponseFactory = context =>
        {
            var errors = context.ModelState
                .Where(e => e.Value.Errors.Count > 0)
                .ToDictionary(
                    kvp => kvp.Key,
                    kvp => kvp.Value.Errors.Select(e => e.ErrorMessage).ToArray()
                );

            return new BadRequestObjectResult(new
            {
                errors = errors
            });
        };
    });
```

## 23. Đảm bảo validation hoạt động trong môi trường production

### Mô tả
Không chỉ test validation trong development, cần kiểm tra trong staging/prod để tránh lỗi runtime.

### Lý do
- Môi trường production có thể khác với development
- Đảm bảo validation hoạt động chính xác trong tất cả môi trường
- Tránh lỗi không mong muốn ảnh hưởng đến người dùng

### Ví dụ
```csharp
// Đảm bảo validation được cấu hình đúng trong tất cả môi trường
public void ConfigureServices(IServiceCollection services)
{
    // Validator luôn được đăng ký bất kể môi trường
    services.AddValidatorsFromAssemblyContaining<Program>();
    
    // Configuration theo môi trường
    if (_env.IsDevelopment())
    {
        // Chi tiết hơn cho dev
        services.AddFluentValidationAutoValidation(config =>
        {
            config.ValidatorOptions.CascadeMode = CascadeMode.Continue;
        });
    }
    else
    {
        // Cấu hình cho production
        services.AddFluentValidationAutoValidation();
        
        // Middleware logging lỗi validation
        services.AddSingleton<ValidationErrorLoggingMiddleware>();
    }
}

// Test tự động cho validation trong CI/CD
[TestClass]
public class ValidationIntegrationTests
{
    [TestMethod]
    public async Task Validation_Should_Work_With_Production_Configuration()
    {
        // Arrange
        var factory = new WebApplicationFactory<Program>()
            .WithWebHostBuilder(builder =>
            {
                // Sử dụng cấu hình production
                builder.UseEnvironment("Production");
            });
            
        var client = factory.CreateClient();
        
        // Act - Gửi request với dữ liệu không hợp lệ
        var response = await client.PostAsync("/api/products", 
            new StringContent(
                JsonSerializer.Serialize(new { }), 
                System.Text.Encoding.UTF8, 
                "application/json"));
                
        // Assert
        Assert.AreEqual(HttpStatusCode.BadRequest, response.StatusCode);
        var content = await response.Content.ReadAsStringAsync();
        Assert.IsTrue(content.Contains("errors"));
    }
}