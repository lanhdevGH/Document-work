# Code Convention cho C# ASP.NET Clean Architecture

## 1. Nguyên tắc chung

### 1.1. Viết test dễ hiểu, độc lập
- Mỗi test cần rõ ràng về mục đích kiểm tra
- Không phụ thuộc vào test case khác
- Sử dụng các biến có tên mô tả đúng mục đích

```csharp
// Không tốt
[Fact]
public void Test1()
{
    var x = 5;
    var y = service.Calculate(x);
    Assert.Equal(10, y);
}

// Tốt
[Fact]
public void Calculate_InputIsFive_ReturnsTen()
{
    // Arrange
    var input = 5;
    
    // Act
    var result = _calculationService.Calculate(input);
    
    // Assert
    Assert.Equal(10, result);
}
```

### 1.2. Mỗi test chỉ kiểm tra một chức năng
- Test nên tập trung vào một hành vi hoặc kịch bản
- Tránh kiểm tra nhiều chức năng trong một test case
- Đảm bảo mỗi test chỉ có một lý do để fail

```csharp
// Không tốt
[Fact]
public void UserServiceTest()
{
    var user = _userService.GetById(1);
    Assert.NotNull(user);
    Assert.Equal("admin", user.Role);
    
    var result = _userService.UpdateUser(user);
    Assert.True(result);
    
    _userService.DeleteUser(1);
    Assert.Null(_userService.GetById(1));
}

// Tốt
[Fact]
public void GetById_ExistingUserId_ReturnsUser()
{
    var user = _userService.GetById(1);
    Assert.NotNull(user);
}

[Fact]
public void GetById_ExistingUser_HasCorrectRole()
{
    var user = _userService.GetById(1);
    Assert.Equal("admin", user.Role);
}
```

### 1.3. Đặt tên test theo MethodName_Scenario_ExpectedBehavior
- Tên test phải mô tả đầy đủ về phương thức được test
- Mô tả kịch bản test (input, điều kiện...)
- Mô tả kết quả mong đợi

```csharp
[Fact]
public void Login_InvalidCredentials_ReturnsUnauthorized()

[Fact] 
public void GetUserById_UserDoesNotExist_ReturnsNull()

[Fact]
public void TransferMoney_InsufficientFunds_ThrowsInsufficientFundsException()
```

### 1.4. Sử dụng Arrange-Act-Assert (AAA)
- Arrange: Chuẩn bị dữ liệu, khởi tạo đối tượng
- Act: Thực hiện hành động cần test
- Assert: Kiểm tra kết quả so với mong đợi

```csharp
[Fact]
public void CalculateDiscount_PremiumCustomer_Returns20Percent()
{
    // Arrange
    var customer = new Customer { Type = CustomerType.Premium };
    var service = new DiscountService();
    
    // Act
    var discount = service.CalculateDiscount(customer);
    
    // Assert
    Assert.Equal(0.2m, discount);
}
```

### 1.5. Không phụ thuộc vào thứ tự chạy
- Test phải chạy độc lập bất kể thứ tự
- Tránh trạng thái chia sẻ giữa các test
- Không test nên phụ thuộc vào kết quả của test khác

```csharp
// Không tốt (phụ thuộc vào biến static)
public static User testUser;

[Fact]
public void Test1_CreateUser()
{
    testUser = _userService.Create(new UserDto());
    Assert.NotNull(testUser);
}

[Fact]
public void Test2_GetUser() 
{
    var user = _userService.GetById(testUser.Id);
    Assert.Equal(testUser.Name, user.Name);
}

// Tốt (mỗi test độc lập)
[Fact]
public void Create_ValidUserData_ReturnsCreatedUser()
{
    var userDto = new UserDto { Name = "Test" };
    var user = _userService.Create(userDto);
    Assert.NotNull(user);
}

[Fact]
public void GetById_ExistingUserId_ReturnsCorrectUser()
{
    // Arrange - tạo dữ liệu test riêng
    var testUser = _fixture.Create<User>();
    _userRepository.Setup(r => r.GetById(testUser.Id)).Returns(testUser);
    
    // Act
    var result = _userService.GetById(testUser.Id);
    
    // Assert
    Assert.Equal(testUser.Name, result.Name);
}
```

### 1.6. Đảm bảo môi trường test nhất quán
- Reset trạng thái trước và sau khi test
- Sử dụng TestInitialize và TestCleanup
- Tránh phụ thuộc vào cấu hình hệ thống

```csharp
public class UserServiceTests : IDisposable
{
    private readonly UserService _userService;
    private readonly Mock<IUserRepository> _mockRepository;
    
    public UserServiceTests()
    {
        // Setup chung cho mọi test
        _mockRepository = new Mock<IUserRepository>();
        _userService = new UserService(_mockRepository.Object);
    }
    
    public void Dispose()
    {
        // Cleanup sau mỗi test
        _mockRepository.Reset();
    }
    
    [Fact]
    public void GetActiveUsers_ReturnsOnlyActiveUsers()
    {
        // Test implementation
    }
}
```

### 1.7. Đảm bảo môi trường test nhất quán
- Đảm bảo test không bị ảnh hưởng bởi yếu tố ngoại cảnh
- Sử dụng dữ liệu test có thể tái tạo lại
- Chuẩn bị và dọn dẹp dữ liệu trước và sau khi test

```csharp
// Sử dụng xUnit fixture để đảm bảo nhất quán
public class DatabaseFixture : IDisposable
{
    public DatabaseFixture()
    {
        DbContext = new TestDbContext();
        DbContext.Database.EnsureCreated();
        SeedTestData(DbContext);
    }
    
    public TestDbContext DbContext { get; }
    
    public void Dispose()
    {
        DbContext.Database.EnsureDeleted();
        DbContext.Dispose();
    }
    
    private void SeedTestData(TestDbContext context)
    {
        // Tạo dữ liệu test
    }
}

public class UserRepositoryTests : IClassFixture<DatabaseFixture>
{
    private readonly DatabaseFixture _fixture;
    
    public UserRepositoryTests(DatabaseFixture fixture)
    {
        _fixture = fixture;
    }
    
    [Fact]
    public void GetUsers_ReturnsAllUsers()
    {
        // Test với dữ liệu từ fixture
    }
}
```

## 2. Unit Test

### 2.1. Không test trực tiếp database, repository
- Sử dụng mock hoặc stub cho repository
- Tránh phụ thuộc vào database thật
- Test logic nghiệp vụ thay vì tương tác database

```csharp
[Fact]
public void GetActiveUsers_ReturnsOnlyUsersWithActiveStatus()
{
    // Arrange
    var mockRepo = new Mock<IUserRepository>();
    var users = new List<User>
    {
        new User { Id = 1, Status = UserStatus.Active },
        new User { Id = 2, Status = UserStatus.Inactive },
        new User { Id = 3, Status = UserStatus.Active }
    };
    
    mockRepo.Setup(r => r.GetAllUsers()).Returns(users);
    var service = new UserService(mockRepo.Object);
    
    // Act
    var result = service.GetActiveUsers();
    
    // Assert
    Assert.Equal(2, result.Count());
    Assert.All(result, user => Assert.Equal(UserStatus.Active, user.Status));
}
```

### 2.2. Sử dụng Moq hoặc Fake Data
- Sử dụng Moq để mock các dependency
- Tạo fake data cho test thay vì dữ liệu thật
- Sử dụng AutoFixture để tạo test data tự động

```csharp
[Fact]
public void ProcessOrder_ValidOrder_ReturnsOrderConfirmation()
{
    // Arrange
    var fixture = new Fixture();
    var order = fixture.Create<Order>();
    
    var mockOrderRepo = new Mock<IOrderRepository>();
    mockOrderRepo.Setup(r => r.Save(It.IsAny<Order>()))
                .Returns(true);
                
    var mockNotification = new Mock<INotificationService>();
    
    var service = new OrderService(mockOrderRepo.Object, mockNotification.Object);
    
    // Act
    var result = service.ProcessOrder(order);
    
    // Assert
    Assert.NotNull(result);
    Assert.Equal(OrderStatus.Confirmed, result.Status);
    mockNotification.Verify(n => n.SendConfirmation(It.IsAny<Order>()), Times.Once);
}
```

### 2.3. Đảm bảo test chạy nhanh
- Unit test phải thực thi nhanh (<100ms)
- Tránh tác vụ I/O (đọc file, network)
- Sử dụng in-memory objects thay vì external services

```csharp
// Không tốt - chậm do đọc file
[Fact]
public void ImportData_ValidCsv_ImportsAllRecords()
{
    var service = new ImportService();
    var result = service.ImportFromCsv("data.csv");
    Assert.Equal(100, result.Count);
}

// Tốt - sử dụng dữ liệu trong bộ nhớ
[Fact]
public void ImportData_ValidCsvContent_ImportsAllRecords()
{
    // Arrange
    var csvContent = "id,name,age\n1,John,30\n2,Jane,25";
    var mockReader = new Mock<ICsvReader>();
    mockReader.Setup(r => r.ReadCsv(It.IsAny<string>()))
             .Returns(csvContent);
             
    var service = new ImportService(mockReader.Object);
    
    // Act
    var result = service.ImportFromCsv("dummy.csv");
    
    // Assert
    Assert.Equal(2, result.Count);
}
```

### 2.4. Viết test có tính deterministic
- Test phải luôn cho kết quả giống nhau khi chạy nhiều lần
- Tránh sử dụng Random, DateTime.Now trực tiếp
- Mock các giá trị không xác định

```csharp
// Không tốt - kết quả phụ thuộc thời gian
[Fact]
public void IsUserAdult_ReturnsCorrectResult()
{
    var user = new User { BirthDate = DateTime.Now.AddYears(-20) };
    var result = _userService.IsAdult(user);
    Assert.True(result);
}

// Tốt - kiểm soát thời gian
[Fact]
public void IsUserAdult_UserOlderThan18_ReturnsTrue()
{
    // Arrange
    var mockTimeProvider = new Mock<ITimeProvider>();
    mockTimeProvider.Setup(t => t.Now).Returns(new DateTime(2023, 1, 1));
    
    var user = new User { BirthDate = new DateTime(2000, 1, 1) };
    var service = new UserService(mockTimeProvider.Object);
    
    // Act
    var result = service.IsAdult(user);
    
    // Assert
    Assert.True(result);
}
```

### 2.5. Kiểm tra xử lý exception
- Test các trường hợp exception được ném ra
- Kiểm tra loại exception và thông điệp
- Đảm bảo exception được xử lý đúng cách

```csharp
[Fact]
public void Withdraw_InsufficientFunds_ThrowsInsufficientFundsException()
{
    // Arrange
    var account = new BankAccount { Balance = 100 };
    var service = new AccountService();
    
    // Act & Assert
    var exception = Assert.Throws<InsufficientFundsException>(() => 
        service.Withdraw(account, 200));
        
    Assert.Equal("Insufficient funds for withdrawal", exception.Message);
}

[Fact]
public void TryGetUser_UserNotFound_ReturnsFalseAndNull()
{
    // Arrange
    var mockRepo = new Mock<IUserRepository>();
    mockRepo.Setup(r => r.GetById(999)).Returns((User)null);
    var service = new UserService(mockRepo.Object);
    
    // Act
    var success = service.TryGetUser(999, out var user);
    
    // Assert
    Assert.False(success);
    Assert.Null(user);
}
```

### 2.6. Kiểm tra các trường hợp biên
- Test các giá trị biên như min, max, null, empty
- Kiểm tra các tình huống edge case
- Đảm bảo ứng dụng xử lý đúng các trường hợp đặc biệt

```csharp
[Theory]
[InlineData(0, true)]  // Min
[InlineData(100, true)]  // Max
[InlineData(-1, false)]  // < Min
[InlineData(101, false)]  // > Max
public void IsValidPercentage_ChecksBoundaries_ReturnsExpectedResult(
    int percentage, bool expected)
{
    // Act
    var result = _validationService.IsValidPercentage(percentage);
    
    // Assert
    Assert.Equal(expected, result);
}

[Theory]
[InlineData("", false)]  // Empty
[InlineData(null, false)]  // Null
[InlineData("a@b.com", true)]  // Valid
[InlineData("invalid", false)]  // Invalid format
public void IsValidEmail_ValidatesEmail_ReturnsExpectedResult(
    string email, bool expected)
{
    var result = _validationService.IsValidEmail(email);
    Assert.Equal(expected, result);
}
```

### 2.7. Sử dụng test parameterized
- Sử dụng [Theory] và [InlineData] trong xUnit
- Test nhiều trường hợp với một method
- Tránh lặp code test

```csharp
[Theory]
[InlineData(10, 20, 30)]
[InlineData(0, 0, 0)]
[InlineData(-5, 5, 0)]
[InlineData(int.MaxValue, 1, int.MinValue)]  // Overflow case
public void Add_DifferentInputs_ReturnsExpectedSum(int a, int b, int expected)
{
    // Arrange
    var calculator = new Calculator();
    
    // Act
    var result = calculator.Add(a, b);
    
    // Assert
    Assert.Equal(expected, result);
}

[Theory]
[InlineData("admin@example.com", true)]
[InlineData("user@domain.co.uk", true)]
[InlineData("invalid", false)]
[InlineData("user@", false)]
[InlineData("", false)]
[InlineData(null, false)]
public void ValidateEmail_VariousInputs_ReturnsExpectedResult(
    string email, bool expected)
{
    var result = _validator.ValidateEmail(email);
    Assert.Equal(expected, result);
}
```

## 3. Integration Test

### 3.1. Dùng WebApplicationFactory để test API
- Sử dụng WebApplicationFactory để khởi tạo ứng dụng
- Test toàn bộ pipeline từ request tới response
- Kiểm tra kết hợp tất cả các layer

```csharp
public class UserApiTests : IClassFixture<WebApplicationFactory<Startup>>
{
    private readonly WebApplicationFactory<Startup> _factory;
    private readonly HttpClient _client;
    
    public UserApiTests(WebApplicationFactory<Startup> factory)
    {
        _factory = factory.WithWebHostBuilder(builder =>
        {
            builder.ConfigureServices(services =>
            {
                // Thay thế service thật bằng test double
                services.RemoveAll(typeof(IUserRepository));
                services.AddScoped<IUserRepository, TestUserRepository>();
            });
        });
        
        _client = _factory.CreateClient();
    }
    
    [Fact]
    public async Task GetUsers_ReturnsSuccessAndUsers()
    {
        // Act
        var response = await _client.GetAsync("/api/users");
        
        // Assert
        response.EnsureSuccessStatusCode();
        var users = await response.Content.ReadFromJsonAsync<List<UserDto>>();
        Assert.NotEmpty(users);
    }
}
```

### 3.2. Kiểm tra cả request và response
- Kiểm tra status code, headers, và content
- Validate cấu trúc và nội dung response
- Test các trường hợp lỗi và xử lý lỗi

```csharp
[Fact]
public async Task CreateUser_ValidInput_ReturnsCreatedAndLocation()
{
    // Arrange
    var newUser = new CreateUserDto { Name = "Test User", Email = "test@example.com" };
    
    // Act
    var response = await _client.PostAsJsonAsync("/api/users", newUser);
    
    // Assert
    Assert.Equal(HttpStatusCode.Created, response.StatusCode);
    Assert.NotNull(response.Headers.Location);
    
    var createdUser = await response.Content.ReadFromJsonAsync<UserDto>();
    Assert.Equal(newUser.Name, createdUser.Name);
    Assert.Equal(newUser.Email, createdUser.Email);
}

[Fact]
public async Task GetUser_InvalidId_ReturnsNotFound()
{
    // Act
    var response = await _client.GetAsync("/api/users/999");
    
    // Assert
    Assert.Equal(HttpStatusCode.NotFound, response.StatusCode);
    var problem = await response.Content.ReadFromJsonAsync<ProblemDetails>();
    Assert.Equal("User not found", problem.Title);
}
```

### 3.3. Không dùng database thật trong integration test
- Sử dụng in-memory database hoặc SQLite
- Cấu hình test environment riêng
- Reset database state sau mỗi test

```csharp
public class ApiTests : IClassFixture<WebApplicationFactory<Startup>>
{
    private readonly WebApplicationFactory<Startup> _factory;
    
    public ApiTests(WebApplicationFactory<Startup> factory)
    {
        _factory = factory.WithWebHostBuilder(builder =>
        {
            builder.ConfigureServices(services =>
            {
                // Thay database thật bằng in-memory
                services.RemoveAll(typeof(DbContextOptions<AppDbContext>));
                services.AddDbContext<AppDbContext>(options =>
                {
                    options.UseInMemoryDatabase("TestDatabase");
                });
                
                // Seed test data
                var sp = services.BuildServiceProvider();
                using var scope = sp.CreateScope();
                var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
                db.Database.EnsureCreated();
                SeedTestData(db);
            });
        });
    }
    
    private void SeedTestData(AppDbContext context)
    {
        // Add test data
        context.Users.Add(new User { Id = 1, Name = "Test User" });
        context.SaveChanges();
    }
    
    [Fact]
    public async Task GetUser_ExistingId_ReturnsUser()
    {
        // Arrange
        var client = _factory.CreateClient();
        
        // Act
        var response = await client.GetAsync("/api/users/1");
        
        // Assert
        response.EnsureSuccessStatusCode();
        var user = await response.Content.ReadFromJsonAsync<UserDto>();
        Assert.Equal("Test User", user.Name);
    }
}
```

### 3.4. Kiểm tra bảo mật (SQL injection, XSS)
- Test các lỗ hổng bảo mật phổ biến
- Kiểm tra xử lý input không tin cậy
- Đảm bảo sanitize dữ liệu đầu vào

```csharp
[Fact]
public async Task UserSearch_SqlInjectionAttempt_ReturnsEmptyAndDoesNotFail()
{
    // Arrange
    var client = _factory.CreateClient();
    var injectionString = "'; DROP TABLE Users; --";
    
    // Act
    var response = await client.GetAsync($"/api/users/search?name={injectionString}");
    
    // Assert
    response.EnsureSuccessStatusCode();
    var users = await response.Content.ReadFromJsonAsync<List<UserDto>>();
    Assert.Empty(users);
    
    // Kiểm tra database vẫn còn nguyên
    var dbClient = _factory.CreateClient();
    var dbCheckResponse = await dbClient.GetAsync("/api/users");
    var allUsers = await dbCheckResponse.Content.ReadFromJsonAsync<List<UserDto>>();
    Assert.NotEmpty(allUsers);
}

[Fact]
public async Task CreateUser_XssAttempt_SanitizesInput()
{
    // Arrange
    var xssPayload = "<script>alert('XSS')</script>";
    var newUser = new CreateUserDto { Name = xssPayload, Email = "test@example.com" };
    
    // Act
    var response = await _client.PostAsJsonAsync("/api/users", newUser);
    
    // Assert
    response.EnsureSuccessStatusCode();
    var createdUser = await response.Content.ReadFromJsonAsync<UserDto>();
    Assert.DoesNotContain("<script>", createdUser.Name);
}
```

### 3.5. Kiểm tra cross-browser cho ứng dụng web
- Test với các browser engine khác nhau
- Sử dụng Selenium hoặc Playwright
- Kiểm tra responsive và compatibility

```csharp
[Theory]
[InlineData("chrome")]
[InlineData("firefox")]
[InlineData("edge")]
public async Task LoginPage_DifferentBrowsers_WorksCorrectly(string browser)
{
    // Arrange
    var playwright = await Playwright.CreateAsync();
    IBrowser playwrightBrowser;
    
    switch (browser)
    {
        case "firefox":
            playwrightBrowser = await playwright.Firefox.LaunchAsync();
            break;
        case "edge":
            playwrightBrowser = await playwright.Chromium.LaunchAsync(
                new BrowserTypeLaunchOptions { Channel = "msedge" });
            break;
        default:
            playwrightBrowser = await playwright.Chromium.LaunchAsync();
            break;
    }
    
    // Act
    var page = await playwrightBrowser.NewPageAsync();
    await page.GotoAsync("https://your-test-app/login");
    await page.FillAsync("#email", "test@example.com");
    await page.FillAsync("#password", "password");
    await page.ClickAsync("#login-button");
    
    // Assert
    await page.WaitForSelectorAsync(".dashboard-container");
    var title = await page.TitleAsync();
    Assert.Contains("Dashboard", title);
    
    // Cleanup
    await playwrightBrowser.CloseAsync();
}
```

## 4. Performance Testing

### 4.1. Dùng BenchmarkDotNet để đo hiệu suất
- Sử dụng BenchmarkDotNet để đo thời gian thực hiện
- So sánh hiệu suất các phương pháp
- Phân tích memory allocation

```csharp
[MemoryDiagnoser]
public class StringConcatenationBenchmark
{
    private const int Iterations = 10000;
    private readonly string[] _parts = { "Hello", "World", "How", "Are", "You" };
    
    [Benchmark(Baseline = true)]
    public string UsingStringConcat()
    {
        string result = string.Empty;
        for (int i = 0; i < Iterations; i++)
        {
            result = string.Empty;
            foreach (var part in _parts)
            {
                result += part;
            }
        }
        return result;
    }
    
    [Benchmark]
    public string UsingStringBuilder()
    {
        StringBuilder sb = new();
        for (int i = 0; i < Iterations; i++)
        {
            sb.Clear();
            foreach (var part in _parts)
            {
                sb.Append(part);
            }
        }
        return sb.ToString();
    }
    
    [Benchmark]
    public string UsingStringJoin()
    {
        string result = string.Empty;
        for (int i = 0; i < Iterations; i++)
        {
            result = string.Join("", _parts);
        }
        return result;
    }
}

// Trong một method:
// BenchmarkRunner.Run<StringConcatenationBenchmark>();
```

### 4.2. Tránh tối ưu hóa quá sớm
- Tập trung vào code đúng trước, tối ưu sau
- Đo lường trước khi tối ưu
- Tối ưu dựa trên bottleneck thực tế

```csharp
// Quy trình tối ưu hóa:
// 1. Viết code đơn giản, dễ đọc
public List<User> GetActiveUsers(List<User> users)
{
    return users.Where(u => u.IsActive).ToList();
}

// 2. Đo lường hiệu suất nếu cần thiết
[Benchmark]
public List<User> StandardLinqFilter()
{
    return _users.Where(u => u.IsActive).ToList();
}

// 3. Tối ưu chỉ khi cần
[Benchmark]
public List<User> OptimizedFilter()
{
    var result = new List<User>(_users.Count / 2); // Pre-allocate capacity
    foreach (var user in _users)
    {
        if (user.IsActive)
        {
            result.Add(user);
        }
    }
    return result;
}
```

### 4.3. Kiểm tra tải và stress test
- Sử dụng JMeter, k6, hoặc NBomber
- Kiểm tra hệ thống dưới tải cao
- Kiểm tra các điểm yếu tiềm ẩn

```csharp
// Sử dụng NBomber cho load testing
public class ApiLoadTests
{
    public static void Main()
    {
        var httpFactory = ClientFactory.Create(
            name: "http_factory",
            clientCount: 100,
            initClient: (_, _) => Task.FromResult(new HttpClient())
        );

        var step = Step.Create("get_users", httpFactory, async context =>
        {
            var client = context.Client;
            try
            {
                var response = await client.GetAsync("https://api.example.com/users");
                return response.IsSuccessStatusCode 
                    ? Response.Ok() 
                    : Response.Fail();
            }
            catch
            {
                return Response.Fail();
            }
        });

        var scenario = ScenarioBuilder.CreateScenario("api_load_test", step)
            .WithWarmUpDuration(TimeSpan.FromSeconds(5))
            .WithLoadSimulations(
                Simulation.KeepConstant(copies: 50, during: TimeSpan.FromMinutes(1))
            );

        NBomberRunner
            .RegisterScenarios(scenario)
            .Run();
    }
}
```

## 5. Coverage & CI/CD

### 5.1. Đảm bảo code coverage ≥ 80%
- Thiết lập target coverage cho project
- Tập trung vào business logic
- Monitỏing coverage theo thời gian

```xml
<!-- coverlet.runsettings -->
<?xml version="1.0" encoding="utf-8" ?>
<RunSettings>
  <DataCollectionRunSettings>
    <DataCollectors>
      <DataCollector friendlyName="XPlat Code Coverage">
        <Configuration>
          <Format>cobertura</Format>
          <Exclude>[*]*.Program,[*]*.Startup</Exclude>
          <Include>[*]*.Controllers.*,[*]*.Services.*</Include>
          <ExcludeByAttribute>GeneratedCodeAttribute,ExcludeFromCodeCoverage</ExcludeByAttribute>
          <IncludeTestAssembly>false</IncludeTestAssembly>
          <SingleHit>false</SingleHit>
          <UseSourceLink>true</UseSourceLink>
          <SkipAutoProps>true</SkipAutoProps>
          <ThresholdType>line</ThresholdType>
          <ThresholdStat>minimum</ThresholdStat>
          <threshold>80</threshold>
        </Configuration>
      </DataCollector>
    </DataCollectors>
  </DataCollectionRunSettings>
</RunSettings>
```

### 5.2. Tích hợp test vào CI/CD
- Chạy test tự động trong CI pipeline
- Ngăn merge nếu test fail
- Tạo báo cáo test và coverage

```yaml
# azure-pipelines.yml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'

steps:
- task: UseDotNet@2
  inputs:
    version: '6.0.x'
    displayName: 'Install .NET Core SDK'

- task: DotNetCoreCLI@2
  displayName: 'Restore packages'
  inputs:
    command: 'restore'
    projects: '**/*.csproj'

- task: DotNetCoreCLI@2
  displayName: 'Build solution'
  inputs:
    command: 'build'
    projects: '**/*.csproj'
    arguments: '--configuration $(buildConfiguration) --no-restore'

- task: DotNetCoreCLI@2
  displayName: 'Run tests with coverage'
  inputs:
    command: 'test'
    projects: '**/*Tests.csproj'
    arguments: '--configuration $(buildConfiguration) --no-build --collect:"XPlat Code Coverage" --settings ./coverlet.runsettings'

- task: PublishCodeCoverageResults@1
  displayName: 'Publish code coverage'
  inputs:
    codeCoverageTool: 'Cobertura'
    summaryFileLocation: '$(Agent.TempDirectory)/**/coverage.cobertura.xml'
    reportDirectory: '$(Build.SourcesDirectory)/CoverageReport'
    failIfCoverageEmpty: true
