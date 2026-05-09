# EF Core Query Optimization

## Mục tiêu

Ở Junior, bạn đã biết CRUD bằng EF Core. Ở Mid-Level, bạn cần hiểu EF Core sinh SQL như thế nào, khi nào query chạy, vì sao chậm và cách tối ưu để giảm CPU, RAM, network và database load.

---

## 1. `IEnumerable` vs `IQueryable`

`IQueryable` chưa chạy ngay. Nó xây dựng expression tree để EF Core dịch sang SQL khi enumerate.

```csharp
IQueryable<Product> query = _db.Products.Where(x => x.IsActive);

query = query.Where(x => x.Price > 100);

var products = await query.ToListAsync();
```

Query chỉ chạy tại:

- `ToListAsync()`
- `FirstOrDefaultAsync()`
- `CountAsync()`
- `AnyAsync()`
- `SingleAsync()`

Sai lầm phổ biến:

```csharp
var products = await _db.Products.ToListAsync();

var expensiveProducts = products
    .Where(x => x.Price > 100)
    .ToList();
```

Đoạn trên load toàn bộ product về memory rồi mới filter.

Nên:

```csharp
var expensiveProducts = await _db.Products
    .Where(x => x.Price > 100)
    .ToListAsync();
```

---

## 2. Projection trước, entity sau

Không nên load entity nếu chỉ cần vài field:

```csharp
var products = await _db.Products
    .AsNoTracking()
    .ToListAsync();
```

Nên projection:

```csharp
var products = await _db.Products
    .AsNoTracking()
    .Where(x => x.IsActive)
    .Select(x => new ProductListItemDto
    {
        Id = x.Id,
        Name = x.Name,
        Price = x.Price,
        CategoryName = x.Category.Name
    })
    .ToListAsync();
```

Lợi ích:

- SQL chỉ select field cần thiết.
- Ít memory hơn.
- Ít network payload giữa DB và app hơn.
- Tránh tracking không cần thiết.

---

## 3. `AsNoTracking()`

Dùng cho query chỉ đọc:

```csharp
var orders = await _db.Orders
    .AsNoTracking()
    .Where(x => x.CustomerId == customerId)
    .ToListAsync();
```

Không nên dùng `AsNoTracking()` nếu bạn cần update entity đó trong cùng DbContext.

Nếu cần identity resolution nhưng không cần tracking:

```csharp
var orders = await _db.Orders
    .AsNoTrackingWithIdentityResolution()
    .Include(x => x.Items)
    .ToListAsync();
```

---

## 4. Tránh N+1 Query

Vấn đề:

```csharp
var orders = await _db.Orders.ToListAsync();

foreach (var order in orders)
{
    Console.WriteLine(order.Customer.Name);
}
```

Nếu lazy loading bật, mỗi order có thể query thêm customer. 100 orders thành 101 queries.

Cách xử lý:

```csharp
var orders = await _db.Orders
    .AsNoTracking()
    .Include(x => x.Customer)
    .ToListAsync();
```

Hoặc tốt hơn, projection:

```csharp
var orders = await _db.Orders
    .AsNoTracking()
    .Select(x => new OrderListItemDto
    {
        Id = x.Id,
        Code = x.Code,
        CustomerName = x.Customer.Name,
        TotalAmount = x.TotalAmount
    })
    .ToListAsync();
```

---

## 5. Pagination đúng cách

Offset pagination:

```csharp
var products = await _db.Products
    .AsNoTracking()
    .OrderByDescending(x => x.CreatedAt)
    .Skip((page - 1) * pageSize)
    .Take(pageSize)
    .Select(x => new ProductListItemDto
    {
        Id = x.Id,
        Name = x.Name
    })
    .ToListAsync();
```

Với bảng rất lớn, offset sâu có thể chậm. Cân nhắc keyset pagination:

```csharp
var products = await _db.Products
    .AsNoTracking()
    .Where(x => x.Id > lastSeenId)
    .OrderBy(x => x.Id)
    .Take(pageSize)
    .ToListAsync();
```

---

## 6. Transaction và SaveChanges

Không nên gọi `SaveChangesAsync()` nhiều lần trong một use case nếu cần atomic:

```csharp
_db.Orders.Add(order);
await _db.SaveChangesAsync();

_db.OrderLogs.Add(log);
await _db.SaveChangesAsync();
```

Nên gom:

```csharp
_db.Orders.Add(order);
_db.OrderLogs.Add(log);

await _db.SaveChangesAsync(cancellationToken);
```

Nếu nhiều bước cần transaction rõ ràng:

```csharp
await using var transaction = await _db.Database.BeginTransactionAsync(cancellationToken);

try
{
    _db.Orders.Add(order);
    await _db.SaveChangesAsync(cancellationToken);

    _db.OrderLogs.Add(log);
    await _db.SaveChangesAsync(cancellationToken);

    await transaction.CommitAsync(cancellationToken);
}
catch
{
    await transaction.RollbackAsync(cancellationToken);
    throw;
}
```

---

## 7. Optimistic Concurrency

Dùng để tránh lost update khi 2 user cùng sửa một record.

```csharp
public class Product
{
    public Guid Id { get; set; }
    public string Name { get; set; } = default!;

    [Timestamp]
    public byte[] RowVersion { get; set; } = default!;
}
```

Khi conflict, EF Core ném `DbUpdateConcurrencyException`. Backend nên trả lỗi rõ ràng để client reload data.

---

## 8. Debug SQL

Bật log query:

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
{
    options.UseNpgsql(connectionString);
    options.EnableSensitiveDataLogging(builder.Environment.IsDevelopment());
});
```

Xem SQL của query:

```csharp
var query = _db.Products
    .Where(x => x.IsActive)
    .Select(x => new { x.Id, x.Name });

var sql = query.ToQueryString();
```

---

## Checklist

- Không `ToListAsync()` quá sớm.
- Query read-only dùng `AsNoTracking()`.
- Dùng projection cho list/detail response.
- Tránh N+1 query.
- List endpoint có pagination.
- Có index cho filter/sort/join phổ biến.
- Use case cần atomic có transaction.
- Có concurrency strategy cho dữ liệu dễ bị update cùng lúc.
- Query quan trọng được kiểm tra SQL generated.

---

## Bài thực hành

- Viết API search order có filter theo status, date range, customer.
- Tối ưu response bằng projection.
- Thêm pagination và sort whitelist.
- Log SQL generated.
- Tạo case concurrent update inventory và xử lý conflict.

