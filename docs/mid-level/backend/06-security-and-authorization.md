# Security & Authorization

## Mục tiêu

Ở Junior, bạn đã biết JWT login cơ bản. Ở Mid-Level, bạn cần hiểu token lifecycle, refresh token, claims, policy-based authorization, bảo vệ API khỏi các lỗi phổ biến và quản lý secret an toàn.

---

## 1. Authentication vs Authorization

| Khái niệm | Câu hỏi | Ví dụ |
| --- | --- | --- |
| Authentication | Bạn là ai? | Login bằng email/password, nhận JWT |
| Authorization | Bạn được làm gì? | Chỉ Admin được xóa user |

JWT chỉ chứng minh identity/claim. Backend vẫn cần authorization rule.

---

## 2. Access Token và Refresh Token

Access token nên sống ngắn:

```txt
Access token: 5-30 phút
Refresh token: vài ngày hoặc vài tuần
```

Refresh token nên:

- Lưu database dạng hash.
- Có expiry.
- Có revoke status.
- Rotate mỗi lần refresh.
- Gắn với user/device/session.

Flow:

```txt
Login
  -> issue access token + refresh token
Access token hết hạn
  -> client gọi /auth/refresh
  -> server validate refresh token
  -> rotate refresh token
  -> trả access token mới
Logout
  -> revoke refresh token
```

---

## 3. Token Storage

| Cách lưu | Ưu điểm | Rủi ro |
| --- | --- | --- |
| localStorage | Dễ implement | Dễ bị XSS lấy token |
| HttpOnly cookie | JS không đọc được token | Cần xử lý CSRF |

Nếu dùng cookie:

- `HttpOnly`
- `Secure`
- `SameSite`
- CSRF protection nếu cần

---

## 4. Claims và Role

Claim là thông tin về user:

```csharp
new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
new Claim(ClaimTypes.Email, user.Email),
new Claim(ClaimTypes.Role, user.Role),
new Claim("tenant_id", user.TenantId.ToString())
```

Không nên nhét quá nhiều data vào JWT:

- Token lớn làm request nặng.
- Data trong token có thể stale.
- Token không nên chứa dữ liệu nhạy cảm.

---

## 5. Policy-based Authorization

Thay vì chỉ dùng role:

```csharp
[Authorize(Roles = "Admin")]
```

Có thể dùng policy:

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("CanManageProducts", policy =>
    {
        policy.RequireClaim("permission", "products.manage");
    });
});
```

Sử dụng:

```csharp
[Authorize(Policy = "CanManageProducts")]
[HttpPost]
public async Task<IActionResult> CreateProduct(CreateProductRequest request)
{
    // ...
}
```

Policy phù hợp khi permission phức tạp hơn role.

---

## 6. Resource-based Authorization

Ví dụ user chỉ được sửa order của chính mình:

```csharp
if (order.UserId != currentUserId && !User.IsInRole("Admin"))
{
    return Forbid();
}
```

Với hệ thống lớn, tách rule vào authorization service:

```csharp
var canUpdate = await _orderAuthorizationService
    .CanUpdateAsync(order, currentUser, cancellationToken);
```

---

## 7. Password Security

Không bao giờ lưu password plain text.

Nên dùng:

- ASP.NET Core Identity password hasher.
- BCrypt/Argon2/PBKDF2.
- Salt.
- Lockout khi login sai nhiều lần.

Checklist:

- Password hash, không encrypt 2 chiều.
- Có rate limit login.
- Có lockout hoặc captcha nếu brute-force.
- Không log password/token.

---

## 8. API Security Checklist

- Bắt buộc HTTPS ở production.
- Validate input.
- Không expose stack trace cho client.
- Không log token/password/secret.
- Secret nằm trong environment/secret manager, không commit repo.
- CORS chỉ allow origin cần thiết.
- Có rate limiting cho endpoint nhạy cảm.
- Có audit log cho hành động quan trọng.
- Refresh token có revoke/rotate.

---

## Bài thực hành

- Thiết kế bảng `RefreshTokens`.
- Implement `/auth/login`, `/auth/refresh`, `/auth/logout`.
- Thêm policy `CanManageProducts`.
- Thêm rate limit cho login.
- Thêm audit log khi admin xóa user.

