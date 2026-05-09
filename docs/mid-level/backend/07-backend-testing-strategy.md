# Backend Testing Strategy

## Mục tiêu

Ở Mid-Level, testing giúp bạn tự tin refactor và phát hiện regression trước khi deploy. Không cần test mọi thứ, nhưng các business flow quan trọng cần được bảo vệ.

---

## 1. Test Pyramid

```txt
        E2E Tests
     Integration Tests
        Unit Tests
```

| Loại test | Test cái gì | Tốc độ |
| --- | --- | --- |
| Unit test | Business rule, service logic, utility | Nhanh |
| Integration test | API + DB + middleware + DI | Trung bình |
| E2E test | Full flow gần giống user thật | Chậm |

Mid-Level cần biết chọn test đúng tầng, không đẩy mọi thứ thành E2E.

---

## 2. Unit Test

Unit test phù hợp cho:

- Pricing rule.
- Discount rule.
- Validation logic.
- Domain service.
- Pure function.

Ví dụ:

```csharp
[Fact]
public void CalculateTotal_ShouldApplyDiscount_WhenCustomerIsVip()
{
    var calculator = new OrderPriceCalculator();

    var total = calculator.Calculate(
        subtotal: 1_000_000,
        customerType: CustomerType.Vip);

    Assert.Equal(900_000, total);
}
```

---

## 3. Integration Test với WebApplicationFactory

Integration test kiểm tra API thật hơn:

```csharp
public class ProductApiTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;

    public ProductApiTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task GetProducts_ShouldReturnOk()
    {
        var response = await _client.GetAsync("/api/products");

        response.EnsureSuccessStatusCode();
    }
}
```

Nên test:

- Middleware pipeline.
- Authentication/authorization.
- Validation error.
- Database query.
- Response contract.

---

## 4. Test Database

Lựa chọn:

| Cách | Ưu điểm | Nhược điểm |
| --- | --- | --- |
| EF InMemory | Nhanh | Không giống relational DB thật |
| SQLite in-memory | Gần SQL hơn | Vẫn khác PostgreSQL/SQL Server |
| Testcontainers | Gần production nhất | Chậm hơn, setup phức tạp hơn |

Với query/transaction quan trọng, nên dùng database thật qua container.

---

## 5. Mock External API

Không nên để test phụ thuộc API thật.

Cách xử lý:

- Mock `HttpMessageHandler`.
- Dùng WireMock.Net.
- Tạo fake implementation cho interface.

Ví dụ:

```csharp
public interface IPaymentClient
{
    Task<PaymentResult> ChargeAsync(PaymentRequest request, CancellationToken cancellationToken);
}
```

Trong test, dùng fake client trả kết quả mong muốn.

---

## 6. Contract Test

Contract test đảm bảo response không phá client.

Nên kiểm tra:

- Field quan trọng còn tồn tại.
- Data type không đổi.
- Error format nhất quán.
- Pagination format đúng.

Ví dụ:

```json
{
  "items": [],
  "page": 1,
  "pageSize": 20,
  "totalItems": 0
}
```

---

## 7. Test Data Strategy

Test tốt cần data rõ ràng:

- Arrange data trong test.
- Không phụ thuộc data có sẵn trong DB dev.
- Cleanup sau test hoặc dùng transaction rollback.
- Dùng builder/factory để tạo object test.

Ví dụ builder:

```csharp
var order = OrderBuilder.New()
    .WithCustomer(customerId)
    .WithItem(productId, quantity: 2)
    .Build();
```

---

## Checklist

- Business rule quan trọng có unit test.
- API quan trọng có integration test.
- Auth/permission có test success và forbidden.
- Validation error có test.
- External API được mock/fake.
- Test không phụ thuộc dữ liệu thủ công.
- CI chạy test tự động.

---

## Bài thực hành

- Viết unit test cho discount rule.
- Viết integration test cho API create order.
- Test validation khi request thiếu item.
- Test user thường không được xóa product.
- Mock payment client cho flow checkout.

