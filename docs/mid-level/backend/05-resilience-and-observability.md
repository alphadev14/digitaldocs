# Resilience & Observability

## Mục tiêu

Resilience giúp hệ thống chịu lỗi tốt hơn. Observability giúp bạn biết hệ thống đang lỗi ở đâu. Ở production, backend không chỉ cần chạy đúng, mà còn cần **dễ quan sát, dễ debug và không sập dây chuyền khi dependency lỗi**.

---

## 1. Timeout

External API call không có timeout là rủi ro lớn.

```csharp
builder.Services.AddHttpClient<PaymentClient>(client =>
{
    client.BaseAddress = new Uri("https://payment.example.com");
    client.Timeout = TimeSpan.FromSeconds(5);
});
```

Timeout nên dựa trên:

- SLA của hệ thống.
- Latency thực tế của dependency.
- User experience.
- Retry strategy.

Không nên để request treo quá lâu rồi giữ connection/thread/resource.

---

## 2. Retry

Retry phù hợp cho lỗi transient:

- Network timeout.
- 502/503/504.
- Temporary database issue.

Không retry cho:

- 400 Bad Request.
- 401/403.
- Business validation error.
- Data conflict không thể tự sửa.

Retry nên có backoff:

```txt
Retry 1: sau 200ms
Retry 2: sau 500ms
Retry 3: sau 1000ms
```

Lưu ý: Retry làm tăng tải. Nếu dependency đang quá tải, retry quá nhiều sẽ làm nó chết nhanh hơn.

---

## 3. Circuit Breaker

Circuit breaker giúp ngừng gọi dependency đang lỗi liên tục.

Flow:

```txt
Closed: gọi bình thường
  -> nhiều lỗi
Open: chặn call một thời gian
  -> sau cooldown
Half-open: thử vài request
  -> thành công thì Closed, fail thì Open lại
```

Dùng khi:

- External API hay timeout.
- Payment/shipping/search service có lúc không ổn định.
- Muốn tránh request của mình góp phần làm dependency quá tải.

---

## 4. Fallback

Fallback là phương án trả kết quả thay thế khi dependency lỗi.

Ví dụ:

- Không lấy được recommendation -> trả empty list.
- Không lấy được tỷ giá realtime -> dùng tỷ giá cached.
- Không gửi được email ngay -> enqueue retry.

Không phải flow nào cũng có fallback. Payment lỗi thì thường không nên giả vờ thành công.

---

## 5. Structured Logging

Log tốt không chỉ là message string.

Không nên:

```csharp
_logger.LogInformation("Create order success");
```

Nên:

```csharp
_logger.LogInformation(
    "Create order success. OrderId={OrderId}, UserId={UserId}, TotalAmount={TotalAmount}",
    order.Id,
    userId,
    order.TotalAmount);
```

Log nên có:

- RequestId/TraceId.
- UserId nếu có.
- Entity id quan trọng: OrderId, ProductId.
- External endpoint.
- Duration.
- Error code.

---

## 6. Correlation Id

Correlation id giúp trace một request qua nhiều service.

Middleware đơn giản:

```csharp
public sealed class CorrelationIdMiddleware
{
    private const string HeaderName = "X-Correlation-Id";
    private readonly RequestDelegate _next;

    public CorrelationIdMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var correlationId = context.Request.Headers.TryGetValue(HeaderName, out var value)
            ? value.ToString()
            : Guid.NewGuid().ToString("N");

        context.Response.Headers[HeaderName] = correlationId;
        context.Items[HeaderName] = correlationId;

        await _next(context);
    }
}
```

---

## 7. Metrics

Metric quan trọng:

- Request count.
- Request duration.
- Error rate.
- DB query duration.
- External API latency.
- Queue length.
- Background job success/fail.
- Cache hit/miss.
- CPU, memory, GC, ThreadPool.

Nên nhìn P95/P99 latency, không chỉ average.

---

## 8. Health Check

Health check giúp load balancer/monitoring biết service còn khỏe không.

```csharp
builder.Services.AddHealthChecks()
    .AddNpgSql(connectionString)
    .AddRedis(redisConnectionString);

app.MapHealthChecks("/health");
```

Tách:

- Liveness: app process còn sống không.
- Readiness: app đã sẵn sàng nhận traffic chưa.

---

## 9. Distributed Tracing

Tracing giúp biết request đi qua service nào, mất bao lâu ở đâu.

Flow:

```txt
Frontend
  -> API Gateway
  -> Order API
  -> Payment API
  -> Database
```

OpenTelemetry thường dùng để export trace/metric/log về backend như Jaeger, Grafana Tempo, Application Insights.

---

## Checklist

- External API có timeout.
- Retry có giới hạn và chỉ dùng cho transient error.
- Có circuit breaker cho dependency dễ lỗi.
- Có fallback nếu nghiệp vụ cho phép.
- Log structured, có context.
- Có correlation id.
- Có metric latency/error rate/throughput.
- Có health check.
- Có tracing cho flow nhiều service.

---

## Bài thực hành

- Thêm correlation id middleware.
- Log request duration.
- Thêm timeout cho typed HttpClient.
- Giả lập API ngoài timeout và thêm retry.
- Thêm health check cho database.
- Tạo dashboard request latency và error rate.

