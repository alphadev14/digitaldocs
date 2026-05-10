# Tổng Quan Về Design Pattern

## 1. Design Pattern là gì?

Design Pattern là "mẫu giải pháp" đã được kiểm chứng để xử lý các vấn đề lặp đi lặp lại trong thiết kế phần mềm.

Design Pattern không phải là code copy-paste. Nó là cách tổ chức code để:

- Dễ mở rộng
- Dễ bảo trì
- Dễ test
- Giảm code trùng lặp
- Giảm phụ thuộc giữa các module
- Giúp team có chung một ngôn ngữ thiết kế

Ví dụ:

Thay vì viết logic tạo object ở nhiều nơi, mình dùng Factory Pattern để gom logic tạo object lại một chỗ.

Nói đơn giản:

```text
Gặp vấn đề lặp lại trong thiết kế code -> dùng pattern phù hợp -> code dễ hiểu và dễ mở rộng hơn
```

---

## 2. Cần phân biệt các nhóm pattern

Trong thực tế, mọi người thường gọi chung là "design pattern", nhưng có nhiều nhóm khác nhau.

| Nhóm | Mục đích | Ví dụ |
| ---- | -------- | ----- |
| GoF Design Patterns | Tổ chức class/object | Factory, Builder, Adapter, Strategy, Observer |
| Architectural Patterns | Tổ chức kiến trúc ứng dụng | Layered Architecture, CQRS, Event Sourcing |
| Enterprise Patterns | Xử lý nghiệp vụ và data access | Repository, Unit of Work, Specification |
| Integration Patterns | Tích hợp hệ thống | Outbox, Retry, Circuit Breaker, Saga |
| UI Patterns | Tổ chức frontend | Component, Container/Presentational, Provider |

Trong tài liệu này, phần `design-pattern` ưu tiên GoF patterns và một số pattern rất hay gặp khi làm .NET backend.

---

## 3. Các nhóm GoF Design Pattern chính

## 3.1 Creational Pattern

Dùng để xử lý việc khởi tạo object.

| Pattern | Khi nghe bài toán này thì nhớ tới | Mức độ nên học |
| ------- | --------------------------------- | -------------- |
| Factory Method | Cần chọn object theo loại/input | Rất nên học |
| Abstract Factory | Cần tạo cả họ object liên quan | Học sau Factory |
| Builder | Object nhiều tham số, nhiều bước cấu hình | Rất nên học |
| Singleton | Chỉ cần một instance dùng chung toàn app | Biết, dùng cẩn thận |
| Prototype | Cần clone object có sẵn | Ít gặp |

---

## 3.2 Structural Pattern

Dùng để tổ chức quan hệ giữa class/module.

| Pattern | Khi nghe bài toán này thì nhớ tới | Mức độ nên học |
| ------- | --------------------------------- | -------------- |
| Adapter | API/interface bên ngoài không khớp code mình | Rất nên học |
| Facade | Che hệ thống con phức tạp bằng API đơn giản | Rất nên học |
| Decorator | Bọc thêm hành vi mà không sửa class gốc | Rất nên học |
| Proxy | Kiểm soát truy cập tới object thật | Nên biết |
| Composite | Dữ liệu dạng cây, node cha/con xử lý giống nhau | Nên biết |
| Bridge | Tách abstraction khỏi implementation | Trung bình |
| Flyweight | Tối ưu bộ nhớ khi có rất nhiều object giống nhau | Ít gặp |

---

## 3.3 Behavioral Pattern

Dùng để xử lý hành vi, thuật toán và luồng xử lý.

| Pattern | Khi nghe bài toán này thì nhớ tới | Mức độ nên học |
| ------- | --------------------------------- | -------------- |
| Strategy | Nhiều cách xử lý thay đổi được | Rất nên học |
| Observer | Một sự kiện xảy ra, nhiều bên cần phản ứng | Rất nên học |
| Chain of Responsibility | Nhiều bước validate/handle nối tiếp nhau | Rất nên học |
| Command | Đóng gói một hành động để queue, retry, undo | Nên học |
| Template Method | Có khung xử lý cố định, vài bước cho class con override | Nên học |
| State | Object đổi hành vi theo trạng thái | Nên học |
| Mediator | Nhiều object giao tiếp rối, cần trung gian điều phối | Trung bình |
| Iterator | Duyệt collection mà không lộ cấu trúc bên trong | Biết là đủ |
| Visitor | Thêm hành vi mới cho cấu trúc object phức tạp | Khó |
| Memento | Lưu và khôi phục trạng thái | Ít gặp |

---

## 4. Design Pattern dùng trong thực tế như thế nào?

## Backend .NET

Ví dụ kiến trúc phổ biến:

```text
Controller -> Service -> Repository -> Database
```

Trong flow này thường có nhiều pattern:

- Dependency Injection: inject service/repository vào nơi cần dùng.
- Service Layer: gom logic nghiệp vụ vào service.
- Repository: che chi tiết truy vấn database.
- Unit of Work: gom nhiều thay đổi database vào một transaction.
- Factory: chọn implementation cần tạo theo input.
- Strategy: thay đổi thuật toán tính giá, tính phí, validate.

Ví dụ Strategy Pattern:

```csharp
public interface IPricingStrategy
{
    decimal CalculatePrice(Product product);
}

public class NormalPriceStrategy : IPricingStrategy
{
    public decimal CalculatePrice(Product product)
    {
        return product.BasePrice;
    }
}

public class PromotionPriceStrategy : IPricingStrategy
{
    public decimal CalculatePrice(Product product)
    {
        return product.BasePrice * 0.9m;
    }
}
```

---

## Frontend React

Các pattern thường gặp:

- Component Pattern
- Container/Presentational Pattern
- Custom Hook Pattern
- Provider Pattern
- Compound Component
- Render Props

Ví dụ Custom Hook:

```tsx
function useProducts() {
  const [products, setProducts] = useState([]);

  useEffect(() => {
    fetchProducts().then(setProducts);
  }, []);

  return { products };
}
```

Khi nghe bài toán "logic fetch data bị lặp ở nhiều component", thường nhớ tới Custom Hook.

---

## System Design / Microservices

Một số pattern hiện đại:

- CQRS
- Event Sourcing
- Saga Pattern
- Outbox Pattern
- Circuit Breaker
- Retry Pattern
- Cache Aside
- API Gateway
- Background Job Pattern

Ví dụ:

Client gọi API tạo đơn hàng:

- API trả response nhanh.
- Gửi email, gửi Telegram, sync dữ liệu sang hệ thống khác chạy sau.
- Nếu gửi message thất bại, cần retry an toàn.

Có thể dùng:

- Background Job
- Queue
- Outbox Pattern
- Retry Pattern

---

## 5. Học theo bài toán thực tế

Đừng học pattern theo kiểu nhớ tên trước rồi cố nhét vào code.

Cách học tốt hơn:

```text
Gặp vấn đề trong code -> nhận diện dấu hiệu -> chọn pattern phù hợp
```

Bảng dưới đây là phần quan trọng nhất của bài overview.

| Bài toán thực tế | Dấu hiệu trong code | Pattern nên nhớ |
| ---------------- | ------------------- | --------------- |
| Có nhiều loại object cần tạo theo input | `switch`, `if else` tạo `new A()`, `new B()` rải rác | Factory |
| Object có quá nhiều constructor parameters | Constructor dài, nhiều optional field, đọc khó hiểu | Builder |
| Cần đúng một instance dùng chung | Config, logger, cache dùng toàn app | Singleton |
| Tích hợp API/thư viện bên ngoài không giống interface nội bộ | Code chính phải biết chi tiết API ngoài | Adapter |
| Một nghiệp vụ phải gọi nhiều service con | Controller/service phải gọi quá nhiều bước | Facade |
| Muốn thêm logging/cache/validation quanh hành vi cũ | Không muốn sửa class gốc | Decorator |
| Cần kiểm soát quyền, cache lazy load, remote call | Muốn đại diện object thật | Proxy |
| Nhiều cách tính giá/phí/discount | `if type == ...` để chọn thuật toán | Strategy |
| Một event xảy ra kéo theo nhiều side effects | Tạo đơn -> gửi email, trừ kho, notify admin | Observer |
| Validate nhiều rule theo chuỗi | Nhiều đoạn `if (...) return error` | Chain of Responsibility |
| Cần đóng gói hành động để queue/retry/undo | Lưu action như một object | Command |
| Quy trình cố định nhưng vài bước thay đổi | Import file nào cũng parse, validate, save nhưng chi tiết khác | Template Method |
| Object đổi hành vi theo trạng thái | Order pending/paid/cancelled xử lý khác nhau | State |
| Nhiều object gọi qua lại quá rối | Module A gọi B, B gọi C, C gọi lại A | Mediator |
| Truy vấn nghiệp vụ có nhiều điều kiện reusable | Rule lọc sản phẩm, đơn hàng, user bị lặp | Specification |
| API cần trả nhanh, việc nặng chạy sau | Gửi email, sync CRM, tạo report | Background Job |
| Cần publish message chắc chắn sau khi lưu DB | Lưu DB thành công nhưng publish event có thể fail | Outbox |
| Gọi service ngoài hay fail tạm thời | Timeout, 503, rate limit | Retry / Circuit Breaker |
| Query và command có nhu cầu tách riêng | Read model khác write model, query cần tối ưu riêng | CQRS |
| Nghiệp vụ nhiều service cần rollback bù | Đặt hàng, thanh toán, giao hàng ở nhiều service | Saga |
| Dữ liệu đọc nhiều, ít thay đổi | Query DB lặp lại nhiều | Cache Aside |

---

## 6. Ví dụ áp dụng thực tế

## Case 1 - Validate nhiều rule khi import Excel

Không nên viết:

```csharp
if (row.Name == null) return error;
if (row.Price <= 0) return error;
if (row.Date < DateTime.Today) return error;
```

Có thể dùng Chain of Responsibility:

```csharp
public interface IImportValidator
{
    Task<ValidationResult> ValidateAsync(ImportContext context);
}
```

Các validator:

```text
CheckRequiredFieldsValidator
CheckDuplicateValidator
CheckPriceValidator
CheckDateRangeValidator
CheckPermissionValidator
```

Lợi ích:

- Dễ mở rộng
- Dễ test
- Dễ đọc
- Mỗi rule tách riêng

---

## Case 2 - Nhiều loại tính giá

Dùng Strategy Pattern:

```text
NormalPriceStrategy
PromotionPriceStrategy
PriceShockStrategy
StorePriceStrategy
```

Thay vì nhồi hết vào một hàm lớn:

```csharp
if (type == "normal") ...
else if (type == "promotion") ...
else if (type == "price-shock") ...
```

Khi nghe "nhiều cách xử lý cùng một bài toán", hãy nhớ tới Strategy.

---

## Case 3 - Tích hợp hệ thống bên ngoài

Dùng Adapter Pattern:

```text
TelegramAdapter
EmailAdapter
ZaloAdapter
SlackAdapter
```

Service chính chỉ gọi interface nội bộ:

```csharp
public interface INotificationClient
{
    Task SendAsync(string receiver, string message);
}
```

Chi tiết API ngoài nằm trong adapter. Code nghiệp vụ không cần biết Telegram/Zalo/Slack gọi endpoint nào.

---

## Case 4 - Checkout gọi nhiều hệ thống con

Một flow checkout có thể cần:

```text
Validate cart
Reserve inventory
Create order
Process payment
Create shipment
Send email
```

Nếu controller gọi trực tiếp hết các service này, code sẽ rất rối.

Có thể dùng Facade:

```csharp
public class CheckoutFacade
{
    public Task CheckoutAsync(CheckoutRequest request)
    {
        // Orchestrate checkout steps here
        return Task.CompletedTask;
    }
}
```

Khi nghe "che một hệ thống con phức tạp bằng API đơn giản", hãy nhớ tới Facade.

---

## Case 5 - Tạo object theo loại payment

Dùng Factory Pattern:

```csharp
public interface IPaymentProcessorFactory
{
    IPaymentProcessor Create(string paymentMethod);
}
```

Các implementation:

```text
MomoPaymentProcessor
VnPayPaymentProcessor
CreditCardPaymentProcessor
```

Khi nghe "tạo object nào phụ thuộc vào input", hãy nhớ tới Factory.

---

## Case 6 - Tạo request/report phức tạp

Dùng Builder Pattern khi object có nhiều optional fields:

```csharp
var reportRequest = ReportRequestBuilder
    .Create()
    .ForMonth(5)
    .ForDepartment("Sales")
    .IncludeCharts()
    .ExportAsPdf()
    .Build();
```

Khi nghe "constructor quá dài" hoặc "object cần build từng bước", hãy nhớ tới Builder.

---

## Case 7 - Sau khi tạo đơn cần làm nhiều việc

Dùng Observer hoặc Domain Event:

```text
OrderCreatedEvent
  -> SendOrderEmailHandler
  -> UpdateInventoryHandler
  -> NotifyAdminHandler
```

Khi nghe "một sự kiện xảy ra, nhiều bên cần phản ứng", hãy nhớ tới Observer.

Nếu các handler chạy async sau response và cần đảm bảo không mất event, cân nhắc thêm Outbox Pattern.

---

## 7. Mức độ phổ biến thực tế

## Cấp 1 - Cực kỳ nên biết

```text
1. Dependency Injection
2. Service Layer
3. Repository Pattern
4. Unit of Work
5. Factory Pattern
6. Strategy Pattern
7. Builder Pattern
8. Adapter Pattern
9. Facade Pattern
10. Observer Pattern
```

---

## Cấp 2 - Rất hữu ích khi code lớn

```text
11. Chain of Responsibility
12. Command Pattern
13. Decorator Pattern
14. Template Method
15. State Pattern
16. Specification Pattern
17. CQRS
18. Cache Aside
19. Outbox Pattern
20. Background Job Pattern
```

---

## Cấp 3 - Học sau

```text
21. Event Sourcing
22. Saga Pattern
23. Mediator Pattern
24. Proxy Pattern
25. Bridge Pattern
26. Composite Pattern
27. Flyweight Pattern
28. Prototype Pattern
29. Visitor Pattern
30. Memento Pattern
```

---

## 8. Thứ tự học phù hợp cho .NET Backend / Fullstack

## Giai đoạn 1 - Nền tảng cực kỳ quan trọng

```text
Dependency Injection
Service Layer
Repository
Unit of Work
Factory
Strategy
Builder
Adapter
Facade
Observer
```

---

## Giai đoạn 2 - Xử lý nghiệp vụ phức tạp

```text
Chain of Responsibility
Specification Pattern
Command Pattern
Decorator Pattern
State Pattern
Template Method
```

Ứng dụng:

- Validate đơn hàng
- Rule khuyến mãi
- Rule giá
- Rule đổi trả
- Import Excel
- Gửi notification nhiều kênh

---

## Giai đoạn 3 - System Design / Production

```text
CQRS
Outbox Pattern
Retry Pattern
Circuit Breaker
Cache Aside
Background Job
Saga Pattern
Event Sourcing
```

Ứng dụng:

- Queue
- Redis
- RabbitMQ
- Microservices
- High performance system
- Distributed transaction

---

## 9. Kết luận

Design Pattern là cách tổ chức code theo các mẫu đã được kiểm chứng.

Với stack:

- C#
- .NET Core
- ReactJS
- PostgreSQL

Nên ưu tiên học:

```text
1. Dependency Injection
2. Service Layer
3. Repository
4. Unit of Work
5. Factory
6. Strategy
7. Builder
8. Adapter
9. Facade
10. Observer
11. Chain of Responsibility
12. Specification
13. Command
14. CQRS
15. Outbox
16. Cache Aside
```

Quan trọng nhất:

```text
Đừng học pattern theo lý thuyết trước.
Hãy học theo bài toán thực tế.
```

Một số câu dễ nhớ:

```text
Nhiều rule validate -> Chain of Responsibility
Nhiều cách tính giá -> Strategy
Tạo object theo loại -> Factory
Tạo object phức tạp -> Builder
Che hệ thống phức tạp -> Facade
Gọi API bên ngoài không khớp interface -> Adapter
Một event, nhiều handler -> Observer
Thêm hành vi mà không sửa class gốc -> Decorator
Hành vi thay đổi theo trạng thái -> State
Đóng gói action để queue/retry/undo -> Command
Xử lý async sau response -> Background Job / Outbox
```
