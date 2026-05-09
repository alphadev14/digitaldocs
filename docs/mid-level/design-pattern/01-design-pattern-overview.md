# Tổng Quan Về Design Pattern

## 1. Design Pattern là gì?

Design Pattern là “mẫu giải pháp” đã được kiểm chứng để xử lý các vấn đề lặp đi lặp lại trong thiết kế phần mềm.

Nó không phải là code copy-paste, mà là cách tổ chức code để:

- Dễ mở rộng
- Dễ bảo trì
- Dễ test
- Giảm code trùng lặp
- Giảm phụ thuộc giữa các module
- Giúp team hiểu chung một ngôn ngữ thiết kế

Ví dụ:

Thay vì viết logic tạo object ở nhiều nơi, mình dùng Factory Pattern để gom logic tạo object lại một chỗ.

---

# 2. Design Pattern dùng trong thực tế như thế nào?

## Backend .NET

Ví dụ phổ biến:

```text
Controller -> Service -> Repository -> Database
```

Đây là sự kết hợp của nhiều pattern:

- Repository Pattern
- Service Layer Pattern
- Dependency Injection
- Unit of Work
- Strategy Pattern
- Factory Pattern

Ví dụ Strategy Pattern:

```csharp
public interface IPricingStrategy
{
    decimal CalculatePrice(Product product);
}

public class NormalPriceStrategy : IPricingStrategy
{
    public decimal CalculatePrice(Product product)
        => product.BasePrice;
}

public class PromotionPriceStrategy : IPricingStrategy
{
    public decimal CalculatePrice(Product product)
        => product.BasePrice * 0.9m;
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

Ví dụ:

```tsx
function useProducts() {
  const [products, setProducts] = useState([]);

  useEffect(() => {
    fetchProducts().then(setProducts);
  }, []);

  return { products };
}
```

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

- API trả response nhanh
- Các xử lý như gửi email, Telegram, sync dữ liệu sẽ chạy background

=> dùng:

- Background Job
- Queue
- Outbox Pattern

---

# 3. Các nhóm Design Pattern chính

## 3.1 Creational Pattern

Dùng để xử lý việc khởi tạo object.

| Pattern          | Mục đích                | Mức độ nên học      |
| ---------------- | ----------------------- | ------------------- |
| Factory Method   | Tạo object theo loại    | Rất nên học         |
| Abstract Factory | Tạo họ object liên quan | Trung bình          |
| Builder          | Tạo object phức tạp     | Rất nên học         |
| Singleton        | Một instance dùng chung | Biết, dùng cẩn thận |
| Prototype        | Clone object            | Ít gặp              |

---

## 3.2 Structural Pattern

Dùng để tổ chức quan hệ giữa class/module.

| Pattern   | Mục đích                           | Mức độ nên học |
| --------- | ---------------------------------- | -------------- |
| Adapter   | Chuyển đổi interface               | Rất nên học    |
| Facade    | Che giấu hệ thống phức tạp         | Rất nên học    |
| Decorator | Bọc thêm hành vi                   | Rất nên học    |
| Proxy     | Đại diện object thật               | Nên biết       |
| Composite | Cấu trúc cây                       | Nên biết       |
| Bridge    | Tách abstraction và implementation | Trung bình     |
| Flyweight | Tối ưu bộ nhớ                      | Ít gặp         |

---

## 3.3 Behavioral Pattern

Dùng để xử lý hành vi và luồng xử lý.

| Pattern                 | Mục đích                | Mức độ nên học |
| ----------------------- | ----------------------- | -------------- |
| Strategy                | Thay đổi thuật toán     | Rất nên học    |
| Observer                | Publish/Subscribe       | Rất nên học    |
| Chain of Responsibility | Validate nhiều bước     | Rất nên học    |
| Command                 | Đóng gói hành động      | Nên học        |
| Template Method         | Khung xử lý cố định     | Nên học        |
| State                   | Hành vi theo trạng thái | Nên học        |
| Mediator                | Giảm phụ thuộc          | Trung bình     |
| Iterator                | Duyệt collection        | Biết là đủ     |
| Visitor                 | Thêm hành vi phức tạp   | Khó            |
| Memento                 | Khôi phục trạng thái    | Ít gặp         |

---

# 4. Mức độ phổ biến thực tế

## Cấp 1 - Cực kỳ nên biết

```text
1. Dependency Injection
2. Repository Pattern
3. Service Layer
4. Factory Pattern
5. Strategy Pattern
6. Builder Pattern
7. Adapter Pattern
8. Facade Pattern
9. Decorator Pattern
10. Observer Pattern
```

---

## Cấp 2 - Rất hữu ích khi code lớn

```text
11. Chain of Responsibility
12. Unit of Work
13. Command Pattern
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
24. Visitor Pattern
25. Bridge Pattern
26. Flyweight Pattern
27. Prototype Pattern
```

---

# 5. Thứ tự học phù hợp cho .NET Backend / Fullstack

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
```

---

## Giai đoạn 2 - Xử lý nghiệp vụ phức tạp

```text
Chain of Responsibility
Specification Pattern
Command Pattern
State Pattern
Template Method
```

Ứng dụng:

- Validate đơn hàng
- Rule khuyến mãi
- Rule giá
- Rule đổi trả
- Import Excel

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
```

Ứng dụng:

- Queue
- Redis
- RabbitMQ
- Microservices
- High performance system

---

# 6. Ví dụ áp dụng thực tế

## Case 1 - Validate nhiều rule khi import Excel

Không nên viết:

```csharp
if (...) return error;
if (...) return error;
if (...) return error;
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

Thay vì nhồi hết vào một hàm lớn.

---

## Case 3 - Tích hợp hệ thống bên ngoài

Dùng Adapter Pattern:

```text
TelegramAdapter
EmailAdapter
ZaloAdapter
SlackAdapter
```

Service chính không cần biết chi tiết từng API.

---

# 7. Kết luận

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
10. Chain of Responsibility
11. Specification
12. Command
13. CQRS
14. Outbox
15. Cache Aside
```

Quan trọng nhất:

Đừng học pattern theo lý trước.
thuyết
Hãy học theo bài toán thực tế:

- Nhiều rule validate -> Chain of Responsibility
- Nhiều cách tính giá -> Strategy
- Tạo object phức tạp -> Builder
- Che hệ thống phức tạp -> Facade
- Gọi API bên ngoài -> Adapter
- Xử lý async sau response -> Background Job / Outbox
