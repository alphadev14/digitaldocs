# Builder Pattern

## 1. Builder Pattern là gì?

Builder Pattern là một Creational Design Pattern dùng để tạo object phức tạp theo từng bước, thay vì nhồi tất cả tham số vào một constructor dài.

Nói đơn giản:

```text
Object nhiều cấu hình -> builder gom từng bước -> trả về object hoàn chỉnh, dễ đọc
```

Builder hữu ích khi object có nhiều optional field, nhiều rule validate khi khởi tạo, hoặc cần fluent API để code gọi đọc giống mô tả nghiệp vụ.

## 2. Vấn đề Builder giải quyết

Không dùng Builder, code khởi tạo object dễ rơi vào tình trạng constructor quá dài:

```csharp
var request = new ReportRequest(
    "sales",
    new DateOnly(2026, 1, 1),
    new DateOnly(2026, 1, 31),
    true,
    true,
    "pdf",
    "vi-VN"
);
```

Vấn đề:

- Người đọc khó nhớ tham số thứ tư, thứ năm có ý nghĩa gì.
- Dễ truyền nhầm thứ tự nếu nhiều tham số cùng kiểu.
- Optional field làm constructor phình to.
- Logic validate object bị rải rác ở caller.

Builder giúp caller mô tả object theo từng bước rõ nghĩa hơn.

## 3. Ví dụ Builder trong C#

```csharp
public sealed class ReportRequest
{
    public string Department { get; init; } = "";
    public DateOnly FromDate { get; init; }
    public DateOnly ToDate { get; init; }
    public bool IncludeCharts { get; init; }
    public bool IncludeSummary { get; init; }
    public string Format { get; init; } = "pdf";
}

public sealed class ReportRequestBuilder
{
    private readonly ReportRequest _request = new();

    public ReportRequestBuilder ForDepartment(string department)
    {
        _request.Department = department;
        return this;
    }

    public ReportRequestBuilder From(DateOnly fromDate, DateOnly toDate)
    {
        if (toDate < fromDate)
        {
            throw new ArgumentException("ToDate must be greater than or equal to FromDate");
        }

        _request.FromDate = fromDate;
        _request.ToDate = toDate;
        return this;
    }

    public ReportRequestBuilder WithCharts()
    {
        _request.IncludeCharts = true;
        return this;
    }

    public ReportRequestBuilder WithSummary()
    {
        _request.IncludeSummary = true;
        return this;
    }

    public ReportRequestBuilder AsFormat(string format)
    {
        _request.Format = format;
        return this;
    }

    public ReportRequest Build()
    {
        if (string.IsNullOrWhiteSpace(_request.Department))
        {
            throw new InvalidOperationException("Department is required");
        }

        return _request;
    }
}
```

Cách dùng:

```csharp
var request = new ReportRequestBuilder()
    .ForDepartment("Sales")
    .From(new DateOnly(2026, 1, 1), new DateOnly(2026, 1, 31))
    .WithCharts()
    .WithSummary()
    .AsFormat("pdf")
    .Build();
```

Điểm hay là code gọi thể hiện ý định rõ hơn constructor dài.

## 4. Khi nào nên dùng Builder?

| Trường hợp | Có nên dùng? | Ghi chú |
| --- | --- | --- |
| Object có nhiều optional field | Có | Builder giúp đọc rõ hơn |
| Constructor có nhiều tham số cùng kiểu | Có | Giảm lỗi truyền nhầm thứ tự |
| Object cần validate khi tạo | Có | Có thể gom validate vào `Build()` |
| Object đơn giản có 2-3 field | Không cần | Object initializer đủ rõ |
| Entity EF Core thông thường | Cẩn thận | EF Core có yêu cầu riêng về constructor/setter |

## 5. Builder khác gì Factory?

| Tiêu chí | Builder | Factory |
| --- | --- | --- |
| Mục tiêu | Tạo một object phức tạp từng bước | Chọn object/implementation phù hợp |
| Dấu hiệu | Constructor dài, optional field nhiều | `switch`/`if` chọn loại object |
| Cách dùng | `WithX().WithY().Build()` | `Create(type)` |
| Ví dụ | Tạo report request, email message | Chọn payment processor |

## 6. Lưu ý khi dùng

- Không dùng Builder chỉ để bọc lại object quá đơn giản.
- Nếu object cần immutable thật sự, cân nhắc builder giữ state riêng rồi tạo object mới ở `Build()`.
- Không để builder chứa business flow lớn; builder chỉ nên phục vụ khởi tạo object.
- Với ASP.NET Core options, có thể dùng options pattern thay vì tự viết builder.

## 7. Tóm tắt

Builder Pattern phù hợp khi quá trình tạo object có nhiều bước, nhiều field tùy chọn hoặc cần validate rõ ràng.

Một câu dễ nhớ:

```text
Builder tốt khi object khó tạo bằng constructor ngắn gọn.
Builder xấu khi nó chỉ làm một object đơn giản trở nên rườm rà.
```
