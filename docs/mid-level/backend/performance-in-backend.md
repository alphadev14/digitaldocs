# Performance trong .NET Backend

---

## 0. Mục tiêu

Tài liệu này giúp bạn:

- Hiểu bản chất performance trong .NET backend.
- Phân biệt đúng Thread, Task, ThreadPool và async/await.
- Biết cách xử lý concurrency để hệ thống scale ổn định.
- Nhận diện các vấn đề thường gặp về memory, GC, HttpClient, database và monitoring.
- Trả lời phỏng vấn theo hướng senior, không chỉ nói "code chạy nhanh" mà nói được về system stability.

---

## 1. Execution Model trong .NET

Flow xử lý request thực tế:

```txt
Client Request
   ↓
Kestrel Server
   ↓
ThreadPool lấy thread
   ↓
Controller
   ↓
Service
   ↓
await DB/API
   ↓
Thread được release về ThreadPool
   ↓
DB/API xử lý xong
   ↓
Continuation được resume
   ↓
Response
```

Insight quan trọng:

- Thread không bị giữ trong lúc `await` một thao tác I/O như database, HTTP API, file, queue.
- Đây là lý do async/await giúp backend scale tốt hơn với workload I/O-bound.
- Async không làm từng request chạy nhanh hơn một cách kỳ diệu, nhưng giúp server phục vụ được nhiều request đồng thời hơn.

Ví dụ:

```csharp
public async Task<ProductDto?> GetProductAsync(int id)
{
    return await _db.Products
        .AsNoTracking()
        .Where(x => x.Id == id)
        .Select(x => new ProductDto
        {
            Id = x.Id,
            Name = x.Name,
            Price = x.Price
        })
        .FirstOrDefaultAsync();
}
```

Trong lúc database xử lý query, thread xử lý request có thể được trả về ThreadPool để phục vụ request khác.

---

## 2. Thread, Task và ThreadPool

### Thread

Thread là luồng thực thi ở mức hệ điều hành.

```csharp
var thread = new Thread(() =>
{
    Console.WriteLine("Run in thread");
});

thread.Start();
```

Nhược điểm:

- Tốn tài nguyên, thường có stack riêng.
- Context switching có cost.
- Tạo quá nhiều thread làm hệ thống khó scale.

### Task

Task là một abstraction đại diện cho một đơn vị công việc bất đồng bộ.

```csharp
public async Task<List<ProductDto>> GetProductsAsync()
{
    return await _repo.GetProductsAsync();
}
```

Cần nhớ:

- `Task` không đồng nghĩa với một thread mới.
- Một `Task` có thể chỉ đại diện cho một operation đang chờ I/O.
- Với I/O-bound, async/await thường hiệu quả hơn tạo thread thủ công.

### ThreadPool

ThreadPool là pool các thread được runtime quản lý và tái sử dụng.

Lợi ích:

- Giảm overhead tạo/hủy thread.
- Giúp runtime điều phối lượng thread phù hợp.
- Là nền tảng quan trọng cho ASP.NET Core request processing.

Senior mindset:

> Không tối ưu bằng cách tạo thêm thread tùy tiện. Hãy tối ưu cách sử dụng ThreadPool, tránh block thread và giới hạn concurrency hợp lý.

---

## 3. Async/Await Deep Dive

### Vấn đề: block async bằng `.Result` hoặc `.Wait()`

Không nên:

```csharp
var result = GetDataAsync().Result;
```

Hoặc:

```csharp
GetDataAsync().Wait();
```

Vì sao nguy hiểm:

- Block thread hiện tại.
- Có thể gây deadlock trong một số context.
- Làm ThreadPool bị cạn thread khi traffic cao.
- Khi nhiều request cùng block, server có thể tăng latency rất mạnh.

Nên dùng:

```csharp
var result = await GetDataAsync();
```

### Vấn đề: chạy tuần tự dù các tác vụ độc lập

Không tối ưu:

```csharp
foreach (var item in items)
{
    await CallApiAsync(item);
}
```

Đoạn trên gọi API tuần tự. Nếu mỗi call mất 200ms và có 100 item, tổng thời gian có thể rất lớn.

Cải thiện với `Task.WhenAll`:

```csharp
var tasks = items.Select(CallApiAsync);
await Task.WhenAll(tasks);
```

Lưu ý:

- `Task.WhenAll` phù hợp khi các tác vụ độc lập.
- Không nên dùng không giới hạn nếu tác vụ gọi DB/API bên ngoài.
- Cần kết hợp concurrency limit khi số lượng item lớn.

---

## 4. Concurrency Control

### Vấn đề

Ví dụ:

```txt
1000 item cần xử lý
   ↓
Task.WhenAll tạo 1000 operation đồng thời
   ↓
1000 DB connection hoặc API call
   ↓
DB/API overload
```

`Task.WhenAll` giúp chạy song song, nhưng không tự bảo vệ downstream service.

### Giải pháp: SemaphoreSlim

```csharp
var semaphore = new SemaphoreSlim(20);

await Task.WhenAll(items.Select(async item =>
{
    await semaphore.WaitAsync();

    try
    {
        await ProcessAsync(item);
    }
    finally
    {
        semaphore.Release();
    }
}));
```

Ý nghĩa:

- Tối đa 20 tác vụ chạy đồng thời.
- Các tác vụ còn lại chờ đến lượt.
- Giảm nguy cơ overload database, HTTP API, Redis, message broker.

### Real case

Ví dụ hệ thống xử lý giá cho 3000 cửa hàng:

- Nếu không limit concurrency, hệ thống có thể bắn quá nhiều query hoặc API call cùng lúc.
- Database connection pool bị cạn.
- Latency tăng, timeout tăng, retry càng làm hệ thống nặng hơn.

Cách xử lý tốt hơn:

- Chia batch.
- Giới hạn concurrency.
- Có timeout.
- Có retry với backoff.
- Có metric để quan sát throughput, latency và error rate.

---

## 5. Memory Leak trong Production

Memory leak trong .NET thường không phải vì GC không chạy, mà vì object vẫn còn được reference nên GC không thể dọn.

### Case 1: Static collection

```csharp
private static readonly List<object> Cache = new();
```

Nếu collection này cứ tăng và không bao giờ xóa, RAM sẽ tăng dần theo thời gian.

Cách xử lý:

- Giới hạn size.
- Có eviction policy.
- Dùng `IMemoryCache` với expiration.
- Theo dõi memory usage trong production.

### Case 2: Event không unsubscribe

```csharp
publisher.OnChanged += handler;
```

Nếu object subscribe event nhưng không unsubscribe, publisher vẫn giữ reference đến subscriber.

Cách xử lý:

```csharp
publisher.OnChanged -= handler;
```

Hoặc dùng pattern phù hợp với lifetime của object.

### Case 3: Cache không có TTL

Ví dụ với Redis:

```txt
SET product:1 "{...}"
```

Nếu key không có TTL, dữ liệu có thể tồn tại mãi và làm Redis tăng memory.

Nên dùng:

```txt
SET product:1 "{...}" EX 3600
```

### Case 4: Sai lifetime của DbContext

Không nên giữ `DbContext` trong Singleton service.

Vấn đề:

- `DbContext` không thiết kế để sống quá lâu.
- Change tracker giữ nhiều entity.
- Có thể gây memory tăng và lỗi concurrency.

Nên dùng:

- `DbContext` với scoped lifetime trong web request.
- `IDbContextFactory<TContext>` cho background job hoặc worker cần tạo context thủ công.

---

## 6. Garbage Collector (GC)

GC dọn object không còn được reference.

### Generations

| Generation | Ý nghĩa |
| --- | --- |
| Gen 0 | Object sống ngắn, thường được collect nhanh |
| Gen 1 | Object sống lâu hơn Gen 0 |
| Gen 2 | Object sống lâu, collection thường đắt hơn |
| LOH | Large Object Heap, thường chứa object lớn |

### Vấn đề thường gặp

- Gen 2 collection nhiều có thể tạo pause đáng kể.
- Object lớn hơn khoảng 85KB thường vào LOH.
- LOH fragmentation có thể làm memory usage tăng.
- Allocate quá nhiều object trong hot path làm GC chạy thường xuyên.

### Tips

- Tránh tạo object không cần thiết trong loop lớn.
- Dùng pagination hoặc streaming thay vì load toàn bộ dữ liệu.
- Tránh giữ reference lâu hơn cần thiết.
- Với query đọc dữ liệu, dùng `AsNoTracking()`.
- Với string nối nhiều lần, dùng `StringBuilder`.

---

## 7. Optimization Techniques

### CPU-bound vs I/O-bound

| Loại workload | Ví dụ | Hướng xử lý |
| --- | --- | --- |
| CPU-bound | Tính toán nặng, encode, compress, xử lý ảnh | Parallelism, background processing, scale CPU |
| I/O-bound | DB query, HTTP API, file, queue | async/await, connection pooling, timeout |

Không nên dùng cùng một cách tối ưu cho mọi vấn đề. Trước tiên phải biết bottleneck nằm ở đâu.

### Pagination

Không nên:

```csharp
var products = await _db.Products.ToListAsync();
```

Nên:

```csharp
var products = await _db.Products
    .AsNoTracking()
    .OrderBy(x => x.Id)
    .Skip((page - 1) * size)
    .Take(size)
    .ToListAsync();
```

Lợi ích:

- Giảm memory.
- Giảm thời gian query.
- Giảm payload trả về client.

### EF Core

Với query chỉ đọc:

```csharp
var products = await _db.Products
    .AsNoTracking()
    .ToListAsync();
```

Lợi ích:

- Không cần change tracking.
- Giảm memory.
- Giảm CPU.

Tránh N+1 query:

```csharp
var orders = await _db.Orders
    .Include(x => x.Items)
    .AsNoTracking()
    .ToListAsync();
```

Hoặc tốt hơn là projection:

```csharp
var orders = await _db.Orders
    .AsNoTracking()
    .Select(x => new OrderDto
    {
        Id = x.Id,
        Total = x.Total,
        ItemCount = x.Items.Count
    })
    .ToListAsync();
```

### String

Không nên:

```csharp
var result = "";

foreach (var item in items)
{
    result += item.Name;
}
```

Nên:

```csharp
var builder = new StringBuilder();

foreach (var item in items)
{
    builder.Append(item.Name);
}

var result = builder.ToString();
```

### Dispose resource

Các resource như stream, file, database connection, HTTP response cần được dispose đúng cách.

```csharp
await using var stream = File.OpenRead(path);
```

Hoặc:

```csharp
using var response = await httpClient.GetAsync(url);
```

---

## 8. HttpClient

### Sai: tạo HttpClient liên tục

Không nên:

```csharp
using var client = new HttpClient();
var response = await client.GetAsync(url);
```

Vấn đề:

- Có thể gây socket exhaustion.
- DNS refresh không được xử lý tối ưu.
- Khó quản lý timeout, retry, logging.

### Đúng: dùng IHttpClientFactory

```csharp
builder.Services.AddHttpClient();
```

Inject:

```csharp
public class ProductApiClient
{
    private readonly HttpClient _httpClient;

    public ProductApiClient(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }

    public async Task<ProductDto?> GetProductAsync(int id)
    {
        return await _httpClient.GetFromJsonAsync<ProductDto>($"/products/{id}");
    }
}
```

Register typed client:

```csharp
builder.Services.AddHttpClient<ProductApiClient>(client =>
{
    client.BaseAddress = new Uri("https://api.example.com");
    client.Timeout = TimeSpan.FromSeconds(5);
});
```

---

## 9. Monitoring

Không thể tối ưu performance chỉ bằng cảm giác. Cần đo.

### Metrics cần theo dõi

- CPU usage.
- RAM usage.
- GC time và số lần Gen 2 collection.
- ThreadPool usage.
- Request latency, đặc biệt P95 và P99.
- Error rate.
- Timeout rate.
- Database query duration.
- Database connection pool.
- HTTP client latency.
- Queue length và processing time.

### Tools

- Prometheus.
- Grafana.
- OpenTelemetry.
- Application Insights.
- Seq hoặc ELK cho log.
- dotnet-counters, dotnet-trace, dotnet-dump khi cần debug sâu.

Ví dụ với `dotnet-counters`:

```bash
dotnet-counters monitor --process-id <pid>
```

Các chỉ số đáng chú ý:

- `cpu-usage`.
- `working-set`.
- `gc-heap-size`.
- `threadpool-thread-count`.
- `threadpool-queue-length`.

---

## 10. Debug Performance

Checklist khi hệ thống chậm:

```txt
CPU có cao không?
RAM có tăng liên tục không?
GC pause có nhiều không?
ThreadPool queue có tăng không?
DB query có chậm không?
Connection pool có bị cạn không?
External API có timeout không?
Network latency có bất thường không?
Cache hit rate có thấp không?
```

Cách tiếp cận:

1. Xác định symptom: latency cao, timeout, CPU cao, RAM tăng hay throughput giảm.
2. Xem metric theo thời gian.
3. Tìm bottleneck chính.
4. Reproduce nếu có thể.
5. Tối ưu điểm nghẽn lớn nhất trước.
6. Đo lại sau khi sửa.

Không nên tối ưu trước khi có dữ liệu, vì rất dễ tối ưu nhầm chỗ.

---

## 11. Senior Mindset

Performance không chỉ là:

```txt
Code chạy nhanh hơn
```

Mà là:

```txt
System ổn định hơn, scale tốt hơn, dễ quan sát hơn và ít timeout hơn
```

Một backend engineer có mindset tốt sẽ quan tâm:

- Bottleneck nằm ở CPU, memory, database, network hay external service.
- Workload là CPU-bound hay I/O-bound.
- Có đang block thread không.
- Có đang tạo quá nhiều concurrency không.
- Có cơ chế timeout, retry, circuit breaker chưa.
- Có metric và log đủ để debug production không.

---

## 12. Câu trả lời phỏng vấn

Gợi ý trả lời:

> Em tối ưu performance backend bằng cách bắt đầu từ đo đạc trước, ví dụ latency, CPU, RAM, GC, ThreadPool và database query duration. Với workload I/O-bound, em dùng async/await để tránh block thread và tận dụng ThreadPool tốt hơn. Với các tác vụ độc lập, em có thể dùng Task.WhenAll, nhưng nếu số lượng lớn thì em giới hạn concurrency bằng SemaphoreSlim để không làm quá tải database hoặc external API. Với EF Core, em dùng pagination, projection và AsNoTracking cho query chỉ đọc, đồng thời tránh N+1 query. Em cũng chú ý memory leak như static collection, event không unsubscribe, cache không TTL và sai lifetime của DbContext. Trong production, em theo dõi metric bằng OpenTelemetry, Prometheus/Grafana hoặc Application Insights để xác định bottleneck rồi tối ưu đúng điểm nghẽn.

Câu ngắn hơn:

> Performance backend không chỉ là code nhanh, mà là hệ thống ổn định khi tải tăng. Em tập trung vào async đúng cách, không block ThreadPool, giới hạn concurrency, tối ưu database query, quản lý memory và luôn có monitoring để tìm bottleneck thực tế.

---

## 13. Summary

Performance trong .NET backend xoay quanh các điểm chính:

- ThreadPool và async/await giúp scale tốt với I/O-bound workload.
- Không block async bằng `.Result` hoặc `.Wait()`.
- `Task.WhenAll` cần đi kèm concurrency control khi xử lý số lượng lớn.
- Memory leak thường xảy ra vì object còn reference, không phải vì GC không hoạt động.
- GC cần được quan sát, đặc biệt Gen 2, LOH và allocation trong hot path.
- EF Core cần pagination, projection, `AsNoTracking()` và tránh N+1.
- `HttpClient` nên được quản lý bằng `IHttpClientFactory`.
- Performance phải được đo bằng metric, trace và log trước khi tối ưu.

