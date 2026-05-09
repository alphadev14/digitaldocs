# Caching Strategy

## Mục tiêu

Cache giúp giảm tải database và external API, nhưng nếu dùng sai sẽ gây stale data, bug khó debug hoặc memory tăng không kiểm soát. Ở Mid-Level, bạn cần biết **cache cái gì, cache ở đâu, TTL bao lâu và invalidate thế nào**.

---

## 1. Khi nào nên cache?

Nên cache khi:

- Dữ liệu đọc nhiều hơn ghi.
- Query/API call tốn thời gian.
- Dữ liệu có thể chấp nhận stale trong một khoảng thời gian.
- Response giống nhau cho nhiều user hoặc nhiều request.

Không nên cache khi:

- Dữ liệu thay đổi liên tục và yêu cầu realtime.
- Dữ liệu nhạy cảm theo user nhưng cache key không đủ phân biệt.
- Chưa đo được bottleneck.
- Không có chiến lược invalidation/TTL.

---

## 2. In-Memory Cache vs Distributed Cache

| Loại | Dùng khi | Lưu ý |
| --- | --- | --- |
| In-memory cache | App chạy 1 instance, data nhỏ, không cần share | Mỗi instance có cache riêng |
| Distributed cache | Nhiều instance, cần share cache | Thường dùng Redis |

In-memory:

```csharp
builder.Services.AddMemoryCache();
```

Redis:

```csharp
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration.GetConnectionString("Redis");
});
```

---

## 3. Cache-Aside Pattern

Flow phổ biến:

```txt
Request
  -> Check cache
  -> Cache hit: return data
  -> Cache miss: query DB/API
  -> Save to cache with TTL
  -> Return data
```

Ví dụ:

```csharp
public async Task<ProductDto?> GetProductAsync(Guid id, CancellationToken cancellationToken)
{
    var cacheKey = $"product:detail:{id}";

    var cached = await _cache.GetStringAsync(cacheKey, cancellationToken);
    if (cached is not null)
    {
        return JsonSerializer.Deserialize<ProductDto>(cached);
    }

    var product = await _db.Products
        .AsNoTracking()
        .Where(x => x.Id == id)
        .Select(x => new ProductDto
        {
            Id = x.Id,
            Name = x.Name,
            Price = x.Price
        })
        .FirstOrDefaultAsync(cancellationToken);

    if (product is null)
    {
        return null;
    }

    await _cache.SetStringAsync(
        cacheKey,
        JsonSerializer.Serialize(product),
        new DistributedCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5)
        },
        cancellationToken);

    return product;
}
```

---

## 4. Cache Key Design

Cache key nên:

- Có prefix theo domain.
- Chứa đủ tham số ảnh hưởng đến data.
- Tránh quá dài.
- Có format thống nhất.

Ví dụ:

```txt
product:detail:{productId}
product:list:category:{categoryId}:page:{page}:size:{size}
user:{userId}:permissions
store:{storeId}:pricing:active
```

Sai lầm:

```txt
products
```

Key này quá chung. Nếu request khác filter/category/page, data sẽ bị lẫn.

---

## 5. TTL và Invalidation

TTL là tuyến phòng thủ bắt buộc.

```csharp
new DistributedCacheEntryOptions
{
    AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
}
```

Invalidation phổ biến:

- Update product -> xóa `product:detail:{id}`.
- Update category -> xóa list cache liên quan nếu có cách track.
- Permission thay đổi -> xóa `user:{userId}:permissions`.

Với list cache có nhiều filter, invalidation khó hơn. Có thể:

- TTL ngắn.
- Cache detail, hạn chế cache list phức tạp.
- Dùng version key.
- Dùng tag-based cache nếu hệ thống hỗ trợ.

---

## 6. Cache Stampede

Cache stampede xảy ra khi một key hết hạn, nhiều request cùng lúc miss cache và cùng query DB.

Cách giảm:

- TTL có jitter.
- Lock per key.
- Background refresh.
- Stale-while-revalidate.

Ví dụ jitter:

```csharp
var ttl = TimeSpan.FromMinutes(5)
    .Add(TimeSpan.FromSeconds(Random.Shared.Next(0, 60)));
```

---

## 7. Cache dữ liệu user-specific

Nếu cache dữ liệu theo user, key phải chứa user id hoặc tenant id:

```txt
user:{userId}:dashboard
tenant:{tenantId}:settings
```

Không được cache response có dữ liệu nhạy cảm bằng key chung.

---

## Checklist

- Cache có TTL.
- Cache key rõ format và chứa đủ tham số.
- Có invalidation strategy khi data thay đổi.
- Không cache dữ liệu nhạy cảm bằng key chung.
- Có log/metric cache hit/miss cho cache quan trọng.
- Tránh cache list phức tạp nếu invalidation quá khó.
- Có giải pháp giảm cache stampede cho key nóng.

---

## Bài thực hành

- Cache product detail bằng Redis trong 5 phút.
- Khi update product, xóa cache detail.
- Thêm metric/log cache hit/miss.
- Tạo list cache có key theo filter/page/pageSize.
- Thử thêm jitter cho TTL.

