# Deploy Backend Lên Render

## 1. Chuẩn bị dự án

- Khởi tạo server **.Net Core API**
- Config các Middleware, DI, DB...

---

## 2. Viết Dockerfile

- Dockerfile

```dockerfile
# Stage 1: build
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Copy solution và csproj
COPY todo-list.sln ./
COPY server/*.csproj server/

# Restore dependencies (force restore trong Linux)
RUN dotnet restore todo-list.sln --disable-parallel --force

# Copy toàn bộ source code
COPY . .

# Publish project
WORKDIR /src/server
RUN dotnet publish -c Release -o /app/publish --no-restore

# Stage 2: runtime
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime
WORKDIR /app

EXPOSE 8080
ENV ASPNETCORE_URLS=http://+:8080

COPY --from=build /app/publish .

ENTRYPOINT ["dotnet", "server.dll"]

```

- Thêm file `.dockerignore` ở root

```gitignore
bin/
obj/
.vscode/
*.user
*.db

```

- Build test local

```bash
docker build -t todo-api .
docker run --rm -p 8686:8080 todo-api
```

Mở http://localhost:8686/swagger
→ nếu chạy OK thì lên Render chắc chắn ổn.

---

## 3. Config lại trong file Program.cs

```csharp
using System.Text;
using server.BLL.Todo;
using server.DAO.Todo;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using server.Services;
using server.BLL.Auth;
using Microsoft.EntityFrameworkCore;
using server.Data;
using server.DAO.Auth;
using server.BLL.User;
using server.DAO.User;

var builder = WebApplication.CreateBuilder(args);

// ================== Services ==================
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(options =>
{
    // Thêm authorize vào Swagger
    options.AddSecurityDefinition("Bearer", new Microsoft.OpenApi.Models.OpenApiSecurityScheme
    {
        In = Microsoft.OpenApi.Models.ParameterLocation.Header,
        Description = "Nhập JWT token vào đây: Bearer {token}",
        Name = "Authorization",
        Type = Microsoft.OpenApi.Models.SecuritySchemeType.ApiKey,
        Scheme = "Bearer"
    });

    options.AddSecurityRequirement(new Microsoft.OpenApi.Models.OpenApiSecurityRequirement
    {
        {
            new Microsoft.OpenApi.Models.OpenApiSecurityScheme
            {
                Reference = new Microsoft.OpenApi.Models.OpenApiReference
                {
                    Type = Microsoft.OpenApi.Models.ReferenceType.SecurityScheme,
                    Id = "Bearer"
                }
            },
            new string[] {}
        }
    });
});

// ================== DB ==================
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseNpgsql(connectionString));

// ========== CORS ==========
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend", policy =>
    {
        policy.AllowAnyOrigin()
              .AllowAnyHeader()
              .AllowAnyMethod();
    });
});

// ========== JWT Authentication ==========
var jwtKey = builder.Configuration["Jwt:Key"] ?? "zx+w2tbmVmSApTH6N+I95qBeTF44g0QAQJPm7D/omlE=";
var jwtIssuer = builder.Configuration["Jwt:Issuer"] ?? "todo-api";

builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = false,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = jwtIssuer,
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(jwtKey))
        };
    });

// ========== Dependency Injection ==========
builder.Services.AddHttpContextAccessor();
builder.Services.AddScoped<ITokenService, TokenService>();
builder.Services.AddScoped<IUserContextService, UserContextService>();
builder.Services.AddScoped<TodoDAO>();
builder.Services.AddScoped<TodoBLL>();
builder.Services.AddScoped<AuthBLL>();
builder.Services.AddScoped<AuthDAO>();
builder.Services.AddScoped<UserBLL>();
builder.Services.AddScoped<UserDAO>();

// ================== Build App ==================
var app = builder.Build();

// Swagger UI (cho cả production luôn, tiện test trên Render)
app.UseSwagger();
app.UseSwaggerUI();

app.UseHttpsRedirection();
app.UseCors("AllowFrontend");
app.UseAuthentication(); // Quan trọng: phải đặt trước UseAuthorization
app.UseAuthorization();

app.MapControllers();

// Đọc cổng từ biến môi trường (Render cung cấp PORT)
var port = Environment.GetEnvironmentVariable("PORT") ?? "8686";
app.Run($"http://0.0.0.0:{port}");

```

---

## 4. Deploy trên Render

- Vào Render Dashboard.
- Chọn **New** -> **Web Service**.
- Kết nối với Github, chọn repo dự án.
- Render sẽ nhận diện Dockerfile => build tự động.
- Chọn Region gần Việt Nam (Singapore).
- Set port = 5000 hoặc 8686 (Render sẽ tự map). Trong `Program.cs`, đảm bảo có:

```csharp
// Đọc cổng từ biến môi trường (Render cung cấp PORT)
var port = Environment.GetEnvironmentVariable("PORT") ?? "8686";
app.Run($"http://0.0.0.0:{port}");

```

---

## 5. Environment Variables

Cấu hình các biến môi trường trên **Render**

- B1: Vào **Dashboard**
- B2: Vào dự án api hiện tại
- B3: Vào **Environment** để thêm các biến môi trường

<p align="center">
  <img src="/digitaldocs/assets/config-env-be.png" alt="Cấu hình biến môi trường" width="600">
</p>

## 6. Check Log

<p align="center">
  <img src="/digitaldocs/assets/check-log-deploy-be.png" alt="Cấu hình biến môi trường" width="600">
</p>
