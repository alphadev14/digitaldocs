# Strategy Pattern

## 1. Strategy Pattern là gì?

Strategy Pattern là một **Behavioral Design Pattern** dùng để tách nhiều cách xử lý khác nhau của cùng một hành động thành các class riêng, sau đó chọn strategy phù hợp khi chạy chương trình.

Nói đơn giản:

```text
Cùng một mục tiêu -> có nhiều cách xử lý khác nhau -> mỗi cách là một strategy riêng
```

Pattern này thường giúp:

- Giảm `if/else` hoặc `switch/case` dài.
- Dễ mở rộng thêm cách xử lý mới mà ít sửa code cũ.
- Tách từng thuật toán thành class nhỏ, dễ test.
- Cho phép thay đổi hành vi theo runtime condition.

## 2. Vấn đề Strategy giải quyết

Giả sử hệ thống có nhiều kiểu giảm giá:

- Giảm cố định.
- Giảm theo phần trăm.
- Flash sale.
- Thành viên VIP.

Nếu dồn hết logic vào một hàm:

```csharp
public decimal GetPrice(decimal price, string discountType)
{
    if (discountType == "FIXED")
    {
        return price - 50_000;
    }

    if (discountType == "PERCENT")
    {
        return price * 0.9m;
    }

    if (discountType == "VIP")
    {
        return price * 0.85m;
    }

    return price;
}
```

Ban đầu code vẫn chạy được. Nhưng khi số loại giảm giá tăng lên, method này sẽ:

- Dài và khó đọc.
- Khó test từng rule riêng.
- Dễ vi phạm Open/Closed Principle vì thêm rule mới thường phải sửa method cũ.
- Dễ kéo business rule của nhiều loại promotion dính chặt vào cùng một class.

Strategy Pattern giúp tách từng rule ra khỏi nhau.

## 3. Cấu trúc cơ bản

Một implementation Strategy thường có ba phần:

| Thành phần | Vai trò |
| --- | --- |
| Strategy interface | Định nghĩa hành vi chung |
| Concrete strategies | Các cách xử lý cụ thể |
| Context/service | Nhận strategy và sử dụng nó |

Trong ví dụ giảm giá:

```text
IDiscountStrategy
  -> FixedDiscountStrategy
  -> PercentDiscountStrategy
  -> VipDiscountStrategy

PriceService
  -> dùng IDiscountStrategy
```

## 4. Ví dụ Strategy đơn giản trong C#

### Bước 1: Tạo interface chung

```csharp
public interface IDiscountStrategy
{
    decimal Calculate(decimal price);
}
```

### Bước 2: Tạo từng strategy riêng

```csharp
public sealed class FixedDiscountStrategy : IDiscountStrategy
{
    public decimal Calculate(decimal price)
    {
        return price - 50_000;
    }
}
```

```csharp
public sealed class PercentDiscountStrategy : IDiscountStrategy
{
    public decimal Calculate(decimal price)
    {
        return price * 0.9m;
    }
}
```

```csharp
public sealed class VipDiscountStrategy : IDiscountStrategy
{
    public decimal Calculate(decimal price)
    {
        return price * 0.85m;
    }
}
```

### Bước 3: Service sử dụng strategy

```csharp
public sealed class PriceService
{
    private readonly IDiscountStrategy _discountStrategy;

    public PriceService(IDiscountStrategy discountStrategy)
    {
        _discountStrategy = discountStrategy;
    }

    public decimal GetFinalPrice(decimal price)
    {
        return _discountStrategy.Calculate(price);
    }
}
```

### Bước 4: Chọn strategy khi sử dụng

```csharp
IDiscountStrategy strategy = new FixedDiscountStrategy();

var service = new PriceService(strategy);

var result = service.GetFinalPrice(500_000);

Console.WriteLine(result);
```

Kết quả:

```text
450000
```

Nếu muốn đổi hành vi:

```csharp
IDiscountStrategy strategy = new PercentDiscountStrategy();

var service = new PriceService(strategy);

var result = service.GetFinalPrice(500_000);
```

`PriceService` không cần biết chi tiết logic giảm giá nào đang chạy. Nó chỉ làm việc với abstraction `IDiscountStrategy`.

## 5. Kết hợp Factory với Strategy

Trong app thực tế, strategy thường không được khởi tạo thủ công từng nơi. Một factory có thể chịu trách nhiệm chọn implementation phù hợp theo input.

```csharp
public enum DiscountType
{
    Fixed,
    Percent,
    Vip
}

public sealed class DiscountStrategyFactory
{
    public IDiscountStrategy Create(DiscountType type)
    {
        return type switch
        {
            DiscountType.Fixed => new FixedDiscountStrategy(),
            DiscountType.Percent => new PercentDiscountStrategy(),
            DiscountType.Vip => new VipDiscountStrategy(),
            _ => throw new ArgumentOutOfRangeException(nameof(type), type, null)
        };
    }
}
```

Sử dụng:

```csharp
var factory = new DiscountStrategyFactory();

var strategy = factory.Create(DiscountType.Fixed);

var service = new PriceService(strategy);

var result = service.GetFinalPrice(500_000);
```

Khi số strategy tăng lên, cách này giúp gom logic lựa chọn vào một chỗ thay vì để `if/else` rải rác khắp codebase.

## 6. Strategy với Dependency Injection trong ASP.NET Core

Trong .NET production, strategy thường được đăng ký qua DI để:

- Quản lý dependency rõ ràng.
- Dễ mock khi test.
- Không cần `new` thủ công ở nhiều nơi.

Ví dụ đăng ký nhiều strategy:

```csharp
builder.Services.AddScoped<FixedDiscountStrategy>();
builder.Services.AddScoped<PercentDiscountStrategy>();
builder.Services.AddScoped<VipDiscountStrategy>();
builder.Services.AddScoped<DiscountStrategyResolver>();
```

Resolver:

```csharp
public sealed class DiscountStrategyResolver
{
    private readonly FixedDiscountStrategy _fixedDiscountStrategy;
    private readonly PercentDiscountStrategy _percentDiscountStrategy;
    private readonly VipDiscountStrategy _vipDiscountStrategy;

    public DiscountStrategyResolver(
        FixedDiscountStrategy fixedDiscountStrategy,
        PercentDiscountStrategy percentDiscountStrategy,
        VipDiscountStrategy vipDiscountStrategy)
    {
        _fixedDiscountStrategy = fixedDiscountStrategy;
        _percentDiscountStrategy = percentDiscountStrategy;
        _vipDiscountStrategy = vipDiscountStrategy;
    }

    public IDiscountStrategy Resolve(DiscountType type)
    {
        return type switch
        {
            DiscountType.Fixed => _fixedDiscountStrategy,
            DiscountType.Percent => _percentDiscountStrategy,
            DiscountType.Vip => _vipDiscountStrategy,
            _ => throw new ArgumentOutOfRangeException(nameof(type), type, null)
        };
    }
}
```

Khi mỗi strategy cần thêm dependency riêng như repository, config hay external service, DI sẽ giúp constructor của từng strategy vẫn rõ ràng và testable.

## 7. Khi nào nên dùng Strategy?

| Trường hợp | Có nên dùng? | Ghi chú |
| --- | --- | --- |
| Có nhiều cách xử lý cho cùng một hành động | Có | Tính giá, tính phí ship, payment, export |
| Logic `if/else` thay đổi theo loại nghiệp vụ | Có | Mỗi nhánh có thể trở thành một strategy |
| Cần thay đổi thuật toán lúc runtime | Có | Chọn theo loại khách hàng, khu vực, cấu hình |
| Mỗi rule cần test độc lập | Có | Strategy nhỏ giúp test tập trung |
| Chỉ có 2 nhánh rất đơn giản và ít thay đổi | Chưa cần | `if/else` có thể dễ đọc hơn |
| Logic chỉ khác nhau ở vài hằng số | Cân nhắc | Có thể dùng config trước khi tạo thêm class |

## 8. Strategy khác gì Factory?

| Tiêu chí | Strategy | Factory |
| --- | --- | --- |
| Mục tiêu chính | Thay đổi cách xử lý | Tạo object phù hợp |
| Câu hỏi nó trả lời | "Xử lý như thế nào?" | "Tạo object nào?" |
| Ví dụ | Cách tính giảm giá | Chọn loại strategy cần tạo |
| Có thể kết hợp không? | Có | Factory thường chọn strategy |

Một cách nhớ nhanh:

```text
Factory chọn object.
Strategy chọn hành vi.
```

## 9. Ví dụ thực tế hay gặp

Strategy rất phù hợp cho:

- `IPricingStrategy`: tính giá thường, giá VIP, giá campaign.
- `IPaymentStrategy`: thanh toán tiền mặt, ví điện tử, thẻ.
- `IExportStrategy`: export PDF, Excel, CSV.
- `INotificationStrategy`: gửi email, SMS, push notification.
- `IShippingFeeStrategy`: tính phí theo nội thành, liên tỉnh, express.
- `IValidationStrategy`: validate rule khác nhau theo từng loại đơn hàng.

## 10. Lưu ý khi dùng

- Đừng tạo strategy chỉ để thay một giá trị nhỏ nếu config là đủ.
- Tên strategy nên phản ánh rule nghiệp vụ, không chỉ phản ánh kỹ thuật.
- Nếu có quá nhiều strategy gần giống nhau, hãy xem lại abstraction hoặc dữ liệu đầu vào.
- Tránh để context/service biết quá nhiều chi tiết về concrete strategy.
- Nếu chọn strategy bằng string rải rác khắp nơi, nên dùng enum hoặc resolver/factory để gom lại.

## 11. Tóm tắt

Strategy Pattern phù hợp khi cùng một hành động có nhiều cách xử lý khác nhau và các cách đó có khả năng thay đổi hoặc mở rộng.

Một câu dễ nhớ:

```text
Strategy tốt khi hành vi thay đổi.
Strategy thừa khi chỉ có một vài nhánh nhỏ, ổn định và rất dễ đọc.
```
