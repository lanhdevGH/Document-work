# Code Convention cho C# ASP.NET với Clean Architecture

## 1. Nguyên tắc chung
- **Tên bằng tiếng Anh**: Đặt tên tất cả các thành phần code bằng tiếng Anh.
- **Tên có ý nghĩa**: Đảm bảo tên mô tả đúng mục đích của thành phần.
- **Không dùng số/ký tự đặc biệt**: Tránh dùng số hoặc ký tự đặc biệt trong tên biến, lớp, phương thức.
  ```csharp
  // Tốt
  public class UserManager
  
  // Không tốt
  public class User123
  public class User_Manager
  ```
- **Tránh viết tắt trừ khi phổ biến**: Chỉ sử dụng viết tắt khi đã phổ biến (như "id", "url", "http").
  ```csharp
  // Tốt
  public int userId;
  
  // Không tốt
  public int usrId;
  ```
- **Dùng tên mô tả rõ ràng**: Tên phải mô tả chính xác chức năng hoặc dữ liệu.
  ```csharp
  // Tốt
  public decimal CalculateTotalPrice()
  
  // Không tốt
  public decimal Calculate()
  ```
- **Tên phải dễ đoán**: Đảm bảo tên dễ hiểu trong ngữ cảnh sử dụng.
- **Tên ngắn gọn nhưng đủ ý nghĩa**: Tránh tên quá dài hoặc quá ngắn.

## 2. Lớp (Classes)
- **Dùng PascalCase**: Viết hoa chữ cái đầu mỗi từ.
  ```csharp
  public class OrderManager
  public class ProductService
  ```
- **Phản ánh trách nhiệm theo Clean Architecture**: Tên lớp phải phản ánh vai trò trong kiến trúc.
  ```csharp
  // Domain Layer
  public class Order
  
  // Application Layer
  public class OrderService
  
  // Infrastructure Layer
  public class OrderRepository
  
  // Presentation Layer
  public class OrderController
  ```
- **Tên lớp là danh từ**: Sử dụng danh từ hoặc cụm danh từ.
  ```csharp
  public class CustomerProfile
  public class PaymentProcessor
  ```
- **Dùng danh từ hoặc cụm danh từ**: Đặt tên mô tả đối tượng.
  ```csharp
  public class ShoppingCart
  public class InvoiceGenerator
  ```
- **Tránh tiền tố không cần thiết**: Không thêm tiền tố không cần thiết vào tên lớp.
  ```csharp
  // Tốt
  public class User
  
  // Không tốt
  public class CUser
  ```
- **Tránh hậu tố thừa**: Không sử dụng hậu tố không cần thiết.
  ```csharp
  // Tốt
  public class Order
  
  // Không tốt
  public class OrderObject
  ```

## 3. Phương thức (Methods)
- **Dùng PascalCase**: Viết hoa chữ cái đầu mỗi từ.
  ```csharp
  public void ProcessOrder()
  public User GetUserById(int id)
  ```
- **Bắt đầu bằng động từ**: Luôn bắt đầu tên phương thức bằng động từ.
  ```csharp
  public void Save()
  public bool Validate()
  ```
- **Ngắn gọn, đủ ý nghĩa**: Tên phải mô tả chức năng một cách ngắn gọn.
- **Động từ + danh từ**: Sử dụng cấu trúc động từ + danh từ.
  ```csharp
  public Order GetOrder(int id)
  public void UpdateUserProfile(UserProfile profile)
  ```
- **Động từ mô tả hành động**: Động từ phải thể hiện đúng hành động thực hiện.
  ```csharp
  public decimal CalculateDiscount()
  public void SendNotification()
  ```
- **Tên mô tả hành động cụ thể**: Tên phải mô tả chính xác chức năng.
  ```csharp
  public User FetchUserData()
  public void ValidatePaymentDetails()
  ```
- **Động từ + mục tiêu**: Cấu trúc tên theo dạng hành động và đối tượng tác động.
  ```csharp
  public void SaveOrder(Order order)
  public bool VerifyCredentials(string username, string password)
  ```

## 4. Biến (Variables)
- **Dùng camelCase**: Viết thường chữ cái đầu từ đầu tiên, viết hoa chữ cái đầu từ tiếp theo.
  ```csharp
  int userId;
  string fullName;
  ```
- **Tên mô tả nội dung**: Đặt tên biến mô tả chính xác dữ liệu chứa.
  ```csharp
  decimal totalAmount;
  bool isActive;
  ```
- **Tránh viết tắt không phổ biến**: Chỉ dùng viết tắt khi đã phổ biến.
  ```csharp
  // Tốt
  int id;
  string url;
  
  // Không tốt
  string addr; // address
  int qnt; // quantity
  ```
- **Tên biến ngắn, tránh lặp ngữ cảnh**: Không lặp lại ngữ cảnh nếu đã rõ.
  ```csharp
  // Trong lớp Order
  // Tốt
  public decimal Total;
  
  // Không tốt
  public decimal OrderTotal;
  ```
- **Tên biến rõ ràng, không trùng lặp**: Tránh trùng lặp trong tên.
  ```csharp
  // Tốt
  int age;
  
  // Không tốt
  int userAgeValue;
  ```
- **Tên biến dễ hiểu theo ngữ cảnh**: Tên biến phải dễ hiểu trong ngữ cảnh.
  ```csharp
  // Trong UserService
  User current; // rõ ràng là current user
  ```
- **Tên biến phản ánh dữ liệu**: Tên phải phản ánh loại dữ liệu chứa.
  ```csharp
  decimal price;
  DateTime createdAt;
  ```

## 5. Tham số (Parameters)
- **Dùng camelCase**: Viết thường chữ cái đầu từ đầu tiên, viết hoa chữ cái đầu từ tiếp theo.
  ```csharp
  public void AddItem(int itemId, string itemName)
  ```
- **Tên mô tả giá trị**: Tên tham số phải mô tả giá trị mà nó đại diện.
  ```csharp
  public User FindUser(string email, bool includeInactive)
  ```
- **Tên tham số ngắn gọn**: Không đặt tên quá dài.
  ```csharp
  public void SetId(int id)
  ```
- **Tên tham số mô tả ý nghĩa**: Tên phải thể hiện rõ mục đích.
  ```csharp
  public void ProcessPayment(decimal amount, string currency)
  ```
- **Tên tham số rõ ràng**: Đảm bảo tên tham số dễ hiểu.
  ```csharp
  public void RegisterUser(string username, string password)
  ```
- **Tên tham số cụ thể**: Tên phải đủ cụ thể để phân biệt.
  ```csharp
  public Order GetOrderById(int orderId)
  ```

## 6. Hằng số (Constants)
- **Dùng UPPER_CASE với dấu _ phân cách**: Viết hoa toàn bộ và phân cách bằng dấu gạch dưới.
  ```csharp
  private const int MAX_RETRY_COUNT = 3;
  ```
- **Toàn bộ chữ cái in hoa**: Viết hoa toàn bộ chữ cái.
  ```csharp
  private const string ERROR_MESSAGE = "Operation failed.";
  ```
- **Dùng PascalCase cho hằng public**: Sử dụng PascalCase cho hằng số public.
  ```csharp
  public const int MaxRetries = 5;
  ```
- **Dùng UPPER_CASE cho hằng**: Ưu tiên UPPER_CASE cho hằng số nội bộ.
  ```csharp
  private const int API_TIMEOUT = 30;
  ```
- **Dùng UPPER_CASE**: Đảm bảo nhất quán trong việc đặt tên.
  ```csharp
  private const int DEFAULT_LIMIT = 10;
  ```

## 7. Interface
- **Dùng PascalCase, bắt đầu bằng I**: Luôn bắt đầu tên interface bằng chữ "I".
  ```csharp
  public interface IOrderService
  public interface IRepository<T>
  public interface IUserRepository
  ```

## 8. Thuộc tính (Properties)
- **Dùng PascalCase**: Viết hoa chữ cái đầu mỗi từ.
  ```csharp
  public int UserId { get; set; }
  public string FullName { get; set; }
  ```
- **Tên mô tả dữ liệu**: Đặt tên mô tả đúng dữ liệu.
  ```csharp
  public bool IsActive { get; set; }
  public DateTime CreatedAt { get; set; }
  ```
- **Tên thuộc tính là danh từ**: Sử dụng danh từ hoặc cụm danh từ.
  ```csharp
  public int OrderId { get; set; }
  public int ItemCount { get; set; }
  ```
- **Tên thuộc tính rõ ràng**: Đảm bảo tên thuộc tính dễ hiểu.
  ```csharp
  public string UserName { get; set; }
  ```
- **Tên thuộc tính ngắn gọn**: Không đặt tên quá dài.
  ```csharp
  public decimal Total { get; set; }
  ```

## 9. Enum
- **Dùng PascalCase cho tên và giá trị**: Viết hoa chữ cái đầu mỗi từ cho cả tên enum và giá trị.
  ```csharp
  public enum OrderStatus
  {
      Pending,
      Processing,
      Completed,
      Cancelled
  }
  ```
- **Tên enum là danh từ số nhiều hoặc số ít**: Sử dụng danh từ.
  ```csharp
  public enum PaymentMethods
  {
      CreditCard,
      BankTransfer,
      PayPal
  }
  
  public enum UserRole
  {
      Admin,
      User,
      Guest
  }
  ```
- **Tên enum mô tả tập hợp**: Tên phải mô tả đúng tập hợp giá trị.
  ```csharp
  public enum ShippingMethod
  {
      Standard,
      Express,
      Overnight
  }
  ```

## 10. Tránh trùng lặp không cần thiết
- **Không lặp lại thông tin ngữ cảnh**: Tránh lặp lại ngữ cảnh đã rõ.
  ```csharp
  // Trong lớp User
  // Tốt
  public string Name { get; set; }
  
  // Không tốt
  public string UserName { get; set; }
  ```
- **Tránh lặp tên lớp trong phương thức**: Không lặp lại tên lớp trong phương thức.
  ```csharp
  // Trong OrderController
  // Tốt
  public Order Get(int id)
  
  // Không tốt
  public Order GetOrder(int id)
  ```
- **Tên ngắn gọn, không dư thừa**: Đảm bảo tên không có thông tin thừa.
  ```csharp
  // Tốt
  public User Fetch(int id)
  
  // Không tốt
  public User FetchUserById(int id)
  ```
- **Tránh lặp ngữ cảnh**: Tránh lặp lại ngữ cảnh khi không cần thiết.
  ```csharp
  // Trong OrderService
  // Tốt
  public void Save(Order order)
  
  // Không tốt
  public void SaveOrder(Order order)
  ```

## 11. Comments (Bình luận)
- **Viết comment rõ ràng, ngắn gọn và có ý nghĩa**: Đảm bảo comment cung cấp thông tin hữu ích.
  ```csharp
  // Tính tổng giá trị đơn hàng sau khi áp dụng giảm giá
  public decimal CalculateFinalPrice()
  ```
- **Giải thích logic phức tạp, tránh ghi chú lại những gì hiển nhiên từ code**: Chỉ comment những điểm cần giải thích.
  ```csharp
  // Kiểm tra tính hợp lệ của mã giảm giá theo quy tắc: 
  // 1. Mã chưa hết hạn
  // 2. Mã chưa được sử dụng
  // 3. Mã áp dụng cho loại sản phẩm này
  public bool ValidateDiscountCode(string code, Product product)
  ```

## 12. Formatting (Định dạng)
- **Sử dụng indent hợp lý**: Sử dụng 4 khoảng trắng cho mỗi cấp.
  ```csharp
  public void Process()
  {
      if (isValid)
      {
          DoSomething();
      }
  }
  ```
- **Dùng khoảng trắng giữa toán tử và sau dấu phẩy**: Đảm bảo code dễ đọc.
  ```csharp
  // Tốt
  int sum = a + b;
  GetUser(id, includeOrders);
  
  // Không tốt
  int sum=a+b;
  GetUser(id,includeOrders);
  ```
- **Tuân thủ định dạng code nhất quán trong toàn dự án**: Sử dụng .editorconfig để đảm bảo tính nhất quán.

## 13. Documentation (Tài liệu)
- **Viết documentation cho lớp, phương thức và thuộc tính**: Cung cấp mô tả đầy đủ.
  ```csharp
  /// <summary>
  /// Dịch vụ xử lý các thao tác liên quan đến đơn hàng
  /// </summary>
  public class OrderService
  {
      /// <summary>
      /// Tạo đơn hàng mới từ giỏ hàng
      /// </summary>
      /// <param name="cartId">ID của giỏ hàng</param>
      /// <param name="userId">ID của người dùng</param>
      /// <returns>Đơn hàng đã được tạo</returns>
      public Order CreateFromCart(int cartId, int userId)
      {
          // Thực hiện logic
      }
  }
  ```
- **Sử dụng XML comments trong C#**: Tận dụng XML comments để tạo tài liệu tự động.

## 14. Unit Testing (Kiểm thử đơn vị)
- **Viết unit tests cho các phương thức quan trọng**: Đảm bảo các phương thức chính được kiểm thử.
  ```csharp
  [Fact]
  public void CalculateTotalPrice_WithValidItems_ReturnsCorrectTotal()
  {
      // Arrange
      var service = new OrderService();
      var items = new List<OrderItem>
      {
          new OrderItem { Price = 10, Quantity = 2 },
          new OrderItem { Price = 15, Quantity = 1 }
      };
      
      // Act
      var result = service.CalculateTotalPrice(items);
      
      // Assert
      Assert.Equal(35, result);
  }
  ```
- **Đảm bảo bao phủ các tình huống chính**: Kiểm thử các trường hợp cơ bản và ngoại lệ.

## 15. Exception Handling (Xử lý ngoại lệ)
- **Xử lý ngoại lệ một cách cụ thể và hợp lý**: Bắt từng loại ngoại lệ riêng biệt.
  ```csharp
  try
  {
      ProcessOrder(order);
  }
  catch (DatabaseException ex)
  {
      _logger.LogError(ex, "Database error while processing order");
      throw new OrderProcessingException("Failed to process order due to database error", ex);
  }
  catch (PaymentException ex)
  {
      _logger.LogError(ex, "Payment error while processing order");
      throw new OrderProcessingException("Failed to process order due to payment error", ex);
  }
  ```
- **Tránh sử dụng catch-all, ghi log lỗi đầy đủ để dễ dàng debug**: Không bắt Exception chung nếu có thể tránh.

## 16. Design Patterns (Mẫu thiết kế)
- **Áp dụng các design patterns khi phù hợp**: Sử dụng pattern khi giải quyết vấn đề đặc thù.
  ```csharp
  // Singleton pattern
  public class ConfigurationManager
  {
      private static ConfigurationManager _instance;
      private static readonly object _lock = new object();
      
      private ConfigurationManager() { }
      
      public static ConfigurationManager Instance
      {
          get
          {
              if (_instance == null)
              {
                  lock (_lock)
                  {
                      if (_instance == null)
                      {
                          _instance = new ConfigurationManager();
                      }
                  }
              }
              return _instance;
          }
      }
  }
  ```
- **Tránh lạm dụng pattern không cần thiết**: Chỉ áp dụng khi thực sự cần thiết.

## 17. Separation of Concerns (Phân tách mối quan tâm)
- **Tách biệt rõ ràng các tầng logic của ứng dụng**: Tuân thủ nguyên tắc Clean Architecture.
  ```csharp
  // Domain Layer
  public class Order { }
  
  // Application Layer
  public class OrderService
  {
      private readonly IOrderRepository _repository;
      
      public OrderService(IOrderRepository repository)
      {
          _repository = repository;
      }
      
      public Order GetById(int id)
      {
          return _repository.GetById(id);
      }
  }
  
  // Infrastructure Layer
  public class OrderRepository : IOrderRepository
  {
      private readonly DbContext _context;
      
      public OrderRepository(DbContext context)
      {
          _context = context;
      }
      
      public Order GetById(int id)
      {
          return _context.Orders.Find(id);
      }
  }
  
  // Presentation Layer
  public class OrderController : Controller
  {
      private readonly IOrderService _service;
      
      public OrderController(IOrderService service)
      {
          _service = service;
      }
      
      public IActionResult Get(int id)
      {
          var order = _service.GetById(id);
          return Ok(order);
      }
  }
  ```
- **Tuân thủ nguyên tắc Single Responsibility Principle (SRP)**: Mỗi lớp chỉ có một trách nhiệm.

## 18. Extensibility (Khả năng mở rộng)
- **Viết code dễ mở rộng và bảo trì**: Thiết kế code để dễ dàng mở rộng trong tương lai.
  ```csharp
  // Interface cho dịch vụ thanh toán
  public interface IPaymentService
  {
      Task<PaymentResult> ProcessPayment(Payment payment);
  }
  
  // Các triển khai cụ thể
  public class CreditCardPaymentService : IPaymentService { }
  public class PayPalPaymentService : IPaymentService { }
  ```
- **Sử dụng dependency injection để tăng khả năng mở rộng và kiểm thử**: Đăng ký các dịch vụ trong IoC container.
  ```csharp
  // Đăng ký dịch vụ trong Startup.cs
  services.AddScoped<IOrderRepository, OrderRepository>();
  services.AddScoped<IOrderService, OrderService>();
  services.AddScoped<IPaymentService, CreditCardPaymentService>();
  ```

## 19. Event Naming (Đặt tên sự kiện)
- **Đặt tên sự kiện bắt đầu bằng "On"**: Sử dụng tiền tố "On" cho tên sự kiện.
  ```csharp
  public event EventHandler OnOrderPlaced;
  public event EventHandler<PaymentCompletedEventArgs> OnPaymentCompleted;
  ```
- **Tên sự kiện phải rõ ràng, diễn tả hành động xảy ra**: Tên phải mô tả đúng sự kiện.
  ```csharp
  public event EventHandler OnUserRegistered;
  public event EventHandler<ProductAddedEventArgs> OnProductAddedToCart;
  ```

## 20. Security (Bảo mật)
- **Xem xét các vấn đề bảo mật trong code**: Luôn tính đến yếu tố bảo mật khi viết code.
  ```csharp
  // Sử dụng phương thức bảo mật để lưu mật khẩu
  public void SaveUser(User user)
  {
      user.PasswordHash = _passwordHasher.HashPassword(user.Password);
      user.Password = null; // Không lưu mật khẩu gốc
      _repository.Save(user);
  }
  ```
- **Tránh lộ thông tin quan trọng**: Không lưu thông tin nhạy cảm vào log hoặc hiển thị lỗi.
  ```csharp
  try
  {
      ValidateUserCredentials(username, password);
  }
  catch (Exception ex)
  {
      // Không log password
      _logger.LogError($"Lỗi xác thực cho người dùng {username}");
      throw new AuthenticationException("Đăng nhập không thành công");
  }
  ```
- **Tuân thủ các nguyên tắc bảo mật theo tiêu chuẩn ngành**: Tuân thủ các tiêu chuẩn và quy tắc bảo mật của ngành.
