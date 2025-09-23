# ASP.NET Core Web API

## 1. Giới thiệu

ASP.NET Core Web API là framework của Microsoft dùng để xây dựng các
dịch vụ web theo kiến trúc RESTful, chạy cross-platform (Windows, Linux,
macOS).

---

## 2. Cấu trúc dự án

Một dự án Web API cơ bản gồm:

- **Controllers**: Chứa các endpoint xử lý
  request/response.
- **Models (BO/DTO)**: Đại diện cho dữ liệu truyền
  vào/ra.
- **Services (BLL)**: Xử lý nghiệp vụ.
- **Data (DAO/Repository)**: Kết nối, thao tác với database.
- **Program.cs/Startup.cs**: Cấu hình middleware, DI, routing, authentication.

---

## 3. Các bước tạo Web API

1.  Tạo dự án:

    ```bash
    dotnet new webapi -n MyApi
    ```

2.  Cấu hình routing trong `Program.cs`:

    ```csharp
    var builder = WebApplication.CreateBuilder(args);
    var app = builder.Build();
    app.MapControllers();
    app.Run();
    ```

3.  Tạo Controller mẫu:

    ```csharp
    [ApiController]
    [Route("api/[controller]")]
    public class UsersController : ControllerBase
    {
        [HttpGet]
        public IActionResult GetAll()
        {
            return Ok(new [] { "User1", "User2" });
        }
    }
    ```

---

## 4. Các thành phần quan trọng

### a. Routing & Endpoints

- `[HttpGet]`, `[HttpPost]`, `[HttpPut]`, `[HttpDelete]`
- Có thể truyền tham số qua **query string**, **route**, hoặc
  **body**.

### b. Dependency Injection (DI)

ASP.NET Core hỗ trợ DI mặc định:

```csharp
builder.Services.AddScoped<IUserService, UserService>();
```

### c. Middleware

Pipeline xử lý request/response: - `UseAuthentication()` -
`UseAuthorization()` - `UseCors()` - `UseSwagger()`

👉 Middleware chạy theo pipeline: Request → Middleware → Controller → Response.

### d. Swagger

Dùng để tạo UI test API:

```csharp
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

app.UseSwagger();
app.UseSwaggerUI();
```

---

## 5. Xác thực & Phân quyền (Authentication & Authorization)

- JWT Bearer Token
- Attribute `[Authorize]`, `[AllowAnonymous]`

Ví dụ cấu hình JWT:

```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]!))
        };
    });
```

---

## 6. Best Practices cho Junior Dev

- Tách code theo 3 lớp: **Controller → BLL → DAO**
- Dùng **async/await** khi thao tác DB/API.
- Trả về response chuẩn: `200 OK`, `400 BadRequest`,
  `401 Unauthorized`, `404 NotFound`...
- Log lỗi bằng `ILogger<T>`.
- Viết unit test cho service.

---

## 7. Tài liệu tham khảo

- [Microsoft Docs - ASP.NET Core Web
  API](https://learn.microsoft.com/aspnet/core/web-api)
- [ASP.NET Core
  Fundamentals](https://learn.microsoft.com/aspnet/core/fundamentals/)
