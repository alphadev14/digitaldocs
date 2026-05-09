# API Architecture

## Mục tiêu

Ở Junior, bạn biết tạo Controller và endpoint CRUD. Ở Mid-Level, bạn cần thiết kế API như một **contract lâu dài** giữa backend và client. API tốt phải rõ ràng, ổn định, dễ debug, dễ versioning và không làm lộ chi tiết database.

---

## 1. API Contract

Một API contract nên trả lời được:

- Endpoint này dùng để làm gì?
- Request cần field nào?
- Response trả về shape nào?
- Lỗi validation/business/system được format ra sao?
- Pagination/filter/sort quy ước thế nào?
- API có breaking change không?

Ví dụ response list nên nhất quán:

```json
{
  "items": [],
  "page": 1,
  "pageSize": 20,
  "totalItems": 120,
  "totalPages": 6
}
```

Ví dụ error response:

```json
{
  "traceId": "00-abc...",
  "code": "PRODUCT_NOT_FOUND",
  "message": "Product not found",
  "details": {
    "productId": "123"
  }
}
```

---

## 2. Controller nên mỏng

Không nên để business logic trong Controller:

```csharp
[HttpPost]
public async Task<IActionResult> Create(CreateOrderRequest request)
{
    if (request.Items.Count == 0)
    {
        return BadRequest();
    }

    // Tính giá, check tồn kho, lưu DB, gửi email...
}
```

Nên để Controller điều phối HTTP, còn business logic nằm ở service/use case:

```csharp
[HttpPost]
public async Task<ActionResult<OrderResponse>> Create(
    CreateOrderRequest request,
    CancellationToken cancellationToken)
{
    var result = await _orderService.CreateAsync(request, cancellationToken);
    return CreatedAtAction(nameof(GetById), new { id = result.Id }, result);
}
```

---

## 3. Request/Response DTO

Không nên trả thẳng entity EF Core:

```csharp
return Ok(order);
```

Vấn đề:

- Lộ field nội bộ.
- Dễ tạo circular reference.
- Client phụ thuộc vào schema database.
- Khó thay đổi model nội bộ.

Nên dùng DTO:

```csharp
public sealed class OrderResponse
{
    public Guid Id { get; init; }
    public string Code { get; init; } = default!;
    public string Status { get; init; } = default!;
    public decimal TotalAmount { get; init; }
    public DateTime CreatedAt { get; init; }
}
```

---

## 4. Validation Strategy

Validation nên tách thành 2 loại:

| Loại | Ví dụ | Xử lý |
| --- | --- | --- |
| Input validation | Email sai format, thiếu field, pageSize quá lớn | Trả `400 Bad Request` |
| Business validation | Hủy order đã shipped, mua vượt tồn kho | Trả lỗi domain/business phù hợp |

Ví dụ input validation:

```csharp
public sealed class CreateProductRequest
{
    [Required]
    [MaxLength(200)]
    public string Name { get; init; } = default!;

    [Range(0.01, 999999999)]
    public decimal Price { get; init; }
}
```

Với dự án lớn, có thể dùng FluentValidation để validation tách khỏi DTO.

---

## 5. Pagination, Filtering, Sorting

Không nên:

```http
GET /api/products
```

Rồi trả toàn bộ dữ liệu.

Nên:

```http
GET /api/products?page=1&pageSize=20&keyword=phone&sort=-createdAt
```

Quy tắc nên có:

- `page` bắt đầu từ 1.
- `pageSize` có max, ví dụ 100.
- Sort field phải whitelist, không nhận field tùy ý rồi đưa thẳng vào query.
- Filter phải rõ kiểu dữ liệu.

---

## 6. API Versioning

Khi API có client khác đang dùng, breaking change cần được quản lý.

Ví dụ:

```http
GET /api/v1/products
GET /api/v2/products
```

Khi nào cần version:

- Đổi response shape.
- Đổi ý nghĩa field.
- Xóa field client đang dùng.
- Đổi behavior quan trọng.

Không cần version nếu chỉ thêm field mới mà client cũ không bị ảnh hưởng.

---

## 7. Idempotency

Idempotency giúp tránh double submit trong các operation quan trọng như tạo order, thanh toán, chuyển tiền.

Ví dụ client gửi:

```http
POST /api/orders
Idempotency-Key: 3f70d8a7-6f38-4e12-8f62-b8b2a1f0d0f1
```

Backend lưu key này. Nếu cùng key được gửi lại, backend trả kết quả cũ thay vì tạo order mới.

Nên áp dụng cho:

- Payment.
- Create order.
- Import file.
- External webhook.
- Retry từ frontend/mobile/API gateway.

---

## 8. CancellationToken

Nếu client disconnect hoặc request bị cancel, backend nên truyền cancellation token xuống DB/API call:

```csharp
[HttpGet("{id:guid}")]
public async Task<ActionResult<ProductResponse>> GetById(
    Guid id,
    CancellationToken cancellationToken)
{
    var product = await _productService.GetByIdAsync(id, cancellationToken);
    return Ok(product);
}
```

Service:

```csharp
public Task<ProductResponse?> GetByIdAsync(Guid id, CancellationToken cancellationToken)
{
    return _db.Products
        .AsNoTracking()
        .Where(x => x.Id == id)
        .Select(x => new ProductResponse
        {
            Id = x.Id,
            Name = x.Name,
            Price = x.Price
        })
        .FirstOrDefaultAsync(cancellationToken);
}
```

---

## Checklist

- Controller mỏng, không chứa business logic phức tạp.
- API response và error response có format thống nhất.
- Không trả thẳng EF Core entity.
- Query list có pagination và giới hạn `pageSize`.
- Sort/filter được validate.
- API breaking change có versioning.
- Operation quan trọng có idempotency.
- Request flow có truyền `CancellationToken`.

---

## Bài thực hành

Thiết kế module Order:

- `POST /api/v1/orders`: tạo order, có idempotency key.
- `GET /api/v1/orders/{id}`: xem chi tiết.
- `GET /api/v1/orders`: search có pagination/filter/sort.
- Error response dùng chung format.
- Controller chỉ gọi service, không chứa business logic.

