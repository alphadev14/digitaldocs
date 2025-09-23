# Middleware & Dependency Injection (ASP.NET Core)

## 1. Giới thiệu nhanh

- **Middleware** là các thành phần xử lý pipeline của HTTP request/response. Mỗi request đi qua chuỗi middleware theo thứ tự đã đăng ký.
- **Dependency Injection (DI)**: ASP.NET Core có DI container built-in, dùng để đăng ký và resolve dependencies cho controller, middleware, services, v.v.

---

## 2. Middleware là gì?

- Một middleware là một component nhận `HttpContext` và `RequestDelegate next` (hoặc có phương thức `InvokeAsync(HttpContext)`), có thể xử lý request trước khi gọi `next()` và/hoặc xử lý response sau khi `next()` trả về.
- Middleware thường được sử dụng để: logging, authentication, error handling, response compression, CORS, v.v.

Ví dụ minimal (inline middleware):

```csharp
app.Use(async (context, next) =>
{
    Console.WriteLine("Before next");
    await next.Invoke();
    Console.WriteLine("After next");
});
```

---

## 3. Tạo Custom Middleware (chi tiết)

**Cách 1 — Class dạng chuẩn:**

```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;

    public RequestLoggingMiddleware(RequestDelegate next, ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        _logger.LogInformation("Handling request: {Method} {Path}", context.Request.Method, context.Request.Path);
        await _next(context);
        _logger.LogInformation("Finished handling request.");
    }
}
```

Đăng ký trong `Program.cs`:

```csharp
app.UseMiddleware<RequestLoggingMiddleware>();
```

**Extension method (gọn):**

```csharp
public static class RequestLoggingMiddlewareExtensions
{
    public static IApplicationBuilder UseRequestLogging(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<RequestLoggingMiddleware>();
    }
}
```

Sau đó `app.UseRequestLogging();`

**Cách 2 — Inline lambda (nhanh cho cases đơn giản):**

```csharp
app.Use(async (context, next) =>
{
    // trước khi next
    await next();
    // sau khi next
});
```

**Inject service vào Middleware:** constructor nhận service sẽ tự resolve nhờ DI container (ví dụ `ILogger<T>` hoặc custom service).

---

## 4. Thứ tự Middleware (Order)

- **Order quan trọng**: Middleware chạy theo thứ tự `app.UseX()` được gọi. Khi gọi `next()`, control quay về middleware trước đó (LIFO for after-next).
- Nếu middleware không gọi `await _next(context)`, pipeline sẽ dừng ở đó (ví dụ: trả về 401 sớm).
- Ví dụ thứ tự hay thấy:
  1. Exception handling (catch exceptions sớm nhất)
  2. HTTPS Redirection / HSTS
  3. Static Files
  4. Routing
  5. Authentication
  6. Authorization
  7. Endpoint / MVC / Controllers
- Ví dụ: nếu đặt `UseAuthorization()` trước `UseAuthentication()` thì sẽ không hoạt động đúng.

---

## 5. Middleware hữu ích & thứ tự khuyến nghị

- `UseExceptionHandler` (global error handler)
- `UseHttpsRedirection`
- `UseStaticFiles`
- `UseRouting`
- `UseCors`
- `UseAuthentication`
- `UseAuthorization`
- `UseResponseCompression`
- `UseResponseCaching`
- `UseSwagger` / `UseSwaggerUI` (thường trước routing/dev-only)
- Tóm tắt ví dụ `Program.cs`:

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.UseExceptionHandler("/error"); // 1
app.UseHttpsRedirection();        // 2
app.UseStaticFiles();             // 3
app.UseRouting();                 // 4
app.UseCors("DefaultPolicy");     // 5
app.UseAuthentication();         // 6
app.UseAuthorization();          // 7
app.MapControllers();
app.Run();
```

---

## 6. Đọc/ghi Request Body trong Middleware

- `HttpRequest.Body` là stream chỉ đọc một lần. Nếu middleware đọc body trước Controller, phải `EnableBuffering()`:

```csharp
context.Request.EnableBuffering();
using var reader = new StreamReader(context.Request.Body, leaveOpen: true);
var body = await reader.ReadToEndAsync();
context.Request.Body.Position = 0; // reset để controller có thể đọc
```

- Ghi response body có thể dùng `Response.Body` wrapper để capture:

```csharp
var originalBody = context.Response.Body;
using var memStream = new MemoryStream();
context.Response.Body = memStream;

await _next(context);

memStream.Seek(0, SeekOrigin.Begin);
var text = new StreamReader(memStream).ReadToEnd();
// ... log text
memStream.Seek(0, SeekOrigin.Begin);
await memStream.CopyToAsync(originalBody);
```

- **Cẩn trọng**: Buffering lớn với body lớn tốn memory => không dùng để log file upload lớn.

---

## 7. Exception handling middleware (global error handling)

Tạo middleware bắt mọi exception, trả response chuẩn JSON lỗi:

```csharp
public class ErrorHandlingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ErrorHandlingMiddleware> _logger;

    public ErrorHandlingMiddleware(RequestDelegate next, ILogger<ErrorHandlingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Unhandled exception");
            context.Response.ContentType = "application/json";
            context.Response.StatusCode = StatusCodes.Status500InternalServerError;
            var result = System.Text.Json.JsonSerializer.Serialize(new { error = "Internal server error" });
            await context.Response.WriteAsync(result);
        }
    }
}
```

**Lưu ý:** Đặt middleware bắt lỗi này **đầu pipeline** (trước middleware khác) để có thể catch các lỗi phát sinh phía sau.

---

## 8. Cách test Middleware

- Dùng `WebApplicationFactory<TEntryPoint>` (integration test) hoặc `TestServer`.
  Ví dụ đơn giản với `WebApplicationFactory` (xUnit):

```csharp
public class MiddlewareTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;

    public MiddlewareTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task Test_Custom_Middleware_Logs()
    {
        var response = await _client.GetAsync("/api/values");
        Assert.True(response.IsSuccessStatusCode);
        // kiểm tra header, body, hoặc logs (tuỳ cách implement)
    }
}
```

---

## 9. Dependency Injection (DI) trong ASP.NET Core — khái niệm cơ bản

- Container được cấu hình trong `Program.cs` bằng `builder.Services` (IServiceCollection).
- Sau khi đăng ký service, framework sẽ _inject_ khi cần (constructor injection).
- Ví dụ đăng ký và sử dụng:

```csharp
public interface IEmailService { Task Send(string to, string subject); }
public class SmtpEmailService : IEmailService { /* ... */ }

builder.Services.AddScoped<IEmailService, SmtpEmailService>();
// inject vào controller
public class AccountController : ControllerBase
{
    private readonly IEmailService _email;
    public AccountController(IEmailService email) { _email = email; }
}
```

---

## 10. Service Lifetimes — chi tiết & pitfalls

1. **Transient** (`AddTransient<T>`)

   - Mỗi lần resolve => tạo instance mới.
   - Dùng cho lightweight, stateless services.

2. **Scoped** (`AddScoped<T>`)

   - Một instance cho **mỗi HTTP request**.
   - Dùng cho DB contexts (EF Core DbContext), repository, unit-of-work.
   - **Quan trọng**: Không inject scoped service vào singleton (sẽ gây lifetime capture issue).

3. **Singleton** (`AddSingleton<T>`)
   - Một instance cho toàn app lifetime.
   - Dùng cho config, cache in-memory, stateless service global.
   - **Pitfall**: Không inject scoped service trực tiếp vào singleton. Nếu cần, tạo scope bên trong singleton bằng `IServiceProvider.CreateScope()`.

**Ví dụ sai (không nên):**

```csharp
// Không nên: DbContext là scoped, nhưng inject trực tiếp vào singleton
builder.Services.AddSingleton<MyServiceThatDependsOnDbContext>();
```

**Cách đúng nếu singleton cần dùng scoped work occasionally:**

```csharp
public class MySingleton
{
    private readonly IServiceProvider _provider;
    public MySingleton(IServiceProvider provider) { _provider = provider; }

    public void DoWork()
    {
        using var scope = _provider.CreateScope();
        var db = scope.ServiceProvider.GetRequiredService<MyDbContext>();
        // use db
    }
}
```

---

## 11. Đăng ký service (patterns)

- **Repository pattern**:

```csharp
services.AddScoped<IUserRepository, UserRepository>();
services.AddScoped<IUserService, UserService>();
```

- **Options pattern** (cấu hình strongly-typed):

```csharp
public class MyOptions { public string Value {get;set;} }

builder.Services.Configure<MyOptions>(builder.Configuration.GetSection("MyOptions"));
// Inject via IOptions<MyOptions> hoặc IOptionsSnapshot<MyOptions>
```

- **Open generic registration**:

```csharp
services.AddScoped(typeof(IRepository<>), typeof(EfRepository<>));
```

- **Factory / Lambda registration** (khi cần custom creation):

```csharp
services.AddSingleton<IMyService>(provider => {
    var config = provider.GetRequiredService<IConfiguration>();
    return new MyService(config["X"]);
});
```

---

## 12. Typed HttpClient / IHttpClientFactory

- Dùng `IHttpClientFactory` thay vì `new HttpClient()`.
- Typed client example:

```csharp
public class GitHubClient
{
    private readonly HttpClient _http;
    public GitHubClient(HttpClient http) => _http = http;
    public Task<string> GetUser(string user) => _http.GetStringAsync($"/users/{user}");
}

services.AddHttpClient<GitHubClient>(client => {
    client.BaseAddress = new Uri("https://api.github.com/");
    client.DefaultRequestHeaders.Add("User-Agent", "MyApp");
});
```

---

## 13. Background tasks & IServiceScope (IHostedService)

- Khi viết background worker singleton (`IHostedService` / `BackgroundService`), nếu cần dùng scoped services (DbContext, repo), bắt buộc `CreateScope()`:

```csharp
public class Worker : BackgroundService
{
    private readonly IServiceProvider _provider;
    public Worker(IServiceProvider provider) { _provider = provider; }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        using var scope = _provider.CreateScope();
        var db = scope.ServiceProvider.GetRequiredService<MyDbContext>();
        // work
    }
}
```

---

## 14. Các lỗi thường gặp & Best Practices

- **Lỗi**: Inject `Scoped` vào `Singleton` => lifetime capture error.  
  **Fix**: use `CreateScope()` inside singleton when you need scoped services.
- **Lỗi**: Đọc request body trong middleware nhưng không reset position => controller không đọc được.  
  **Fix**: `context.Request.EnableBuffering()` và `context.Request.Body.Position = 0`.
- **Lỗi**: Middleware đặt sai thứ tự (ví dụ đặt `UseAuthorization` trước `UseAuthentication`).  
  **Fix**: đảm bảo thứ tự: Routing -> Authentication -> Authorization -> Endpoints.
- **Best practices**:
  - Keep middleware thin; logic phức tạp vào service.
  - Không buffer toàn bộ request body nếu không cần.
  - Sử dụng `ILogger<T>` để log trong middleware/service.
  - Sử dụng `IOptionsSnapshot<T>` cho config reload trong môi trường phát triển.

---

## 15. Tài nguyên tham khảo

- [Microsoft docs – Middleware](https://learn.microsoft.com/aspnet/core/fundamentals/middleware)
- [Microsoft docs – Dependency injection](https://learn.microsoft.com/aspnet/core/fundamentals/dependency-injection)
- [IHttpClientFactory](https://learn.microsoft.com/aspnet/core/fundamentals/http-requests)
