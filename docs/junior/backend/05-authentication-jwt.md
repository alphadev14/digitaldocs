# Authentication với JWT

## 1. JWT là gì?

- **JWT (Json Web Token)** là chuẩn mở dùng để truyền thông tin giữa client và server dưới dạng **token**.

- Cấu trúc của JWT gồm 3 phần (Header.Payload.Signature) được nối bằng dấu `.`:

```csharp
  xxxxx.yyyyy.zzzzz
```

- **Header**: thuật toán ký (HS256, RS256).
- **Payload**: chứa claim (userId, role, exp).
- **Signature**: đảm bảo token không bị sửa.

Khi người dùng đăng nhập thành công, server cấp JWT cho client. Sau đó client gửi JWT này trong `Authorization: Bearer <token> `ở mỗi request.

---

## 2. Tạo AuthController (Login, Register)

Ví dụ:

#### `AuthController.cs`

```csharp
[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly IConfiguration _config;
    private readonly IUserService _userService;

    public AuthController(IConfiguration config, IUserService userService)
    {
        _config = config;
        _userService = userService;
    }

    [HttpPost("register")]
    public IActionResult Register(RegisterDto model)
    {
        var user = _userService.Register(model);
        return Ok(new { message = "Đăng ký thành công", user });
    }

    [HttpPost("login")]
    public IActionResult Login(LoginDto model)
    {
        var user = _userService.ValidateUser(model);
        if (user == null) return Unauthorized();

        var token = GenerateJwtToken(user);
        return Ok(new { token });
    }

    private string GenerateJwtToken(User user)
    {
        var claims = new[]
        {
            new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
            new Claim(ClaimTypes.Email, user.Email),
            new Claim(ClaimTypes.Role, user.Role)
        };

        var key = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(_config["Jwt:Key"])
        );
        var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var token = new JwtSecurityToken(
            issuer: _config["Jwt:Issuer"],
            audience: _config["Jwt:Audience"],
            claims: claims,
            expires: DateTime.Now.AddHours(1),
            signingCredentials: creds
        );

        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

#### Cấu hình trong `appsettings.json`

```csharp
"Jwt": {
  "Key": "supersecretkey123456",
  "Issuer": "yourapp",
  "Audience": "yourappusers"
}
```

## 3. Middleware xác thực JWT

Trong `Program.cs` thêm JWT authentication:

```csharp
builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
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
            Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"])
        )
    };
});

builder.Services.AddAuthorization();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();
```

👉 Khi đó, bạn có thể dùng [`Authorize`] trong controller:

```csharp
[Authorize]
[HttpGet("profile")]
public IActionResult Profile()
{
    var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
    return Ok(new { message = "Bạn đang đăng nhập", userId });
}
```

## 4. Lưu token ở Frontend (FE)

- Sau khi login thành công, server trả về:

```json
{ "token": "eyJhbGciOi..." }
```

- FE nên lưu token ở:

      - **LocalStorage**: dễ dùng, nhưng bị XSS tấn công.
      - **HttpOnly Cookie:**: an toàn hơn, nhưng khó thao tác JS. Trong thực tế thì các công ty lớn lưu ở đây.

- Ví dụ ReactJS:

```javascript
// Khi login thành công
localStorage.setItem("token", response.data.token);

// Khi gọi API
const token = localStorage.getItem("token");
axios.get("/api/user/profile", {
  headers: { Authorization: `Bearer ${token}` },
});
```

## 5. Refresh Token

- Vì **JWT** thường có thời gian sống ngắn (1h, 2h) -> cần **refresh token** để tránh login lại nhiều lần.
- Thường sẽ lưu ở BE **refresh token** và ở FE **access token** và **refresh token**
- Khi **access token** hết hạn -> FE gọi API refresh để xin token mới
