# Entiy Framework Core (EF Core)

## 1. Giới thiệu

- **Entity Framework Core (EF Core)**: Là một ORM (Object Relational Mapper) cho .NET.
- Cho phép làm việc với database bằng C# object thay vì viết SQL trực tiếp.
- Hỗ trợ nhiều BDMS: SQL Server, PostgreSQL, MySQL, SQLite, CosmosDB,...

---

## 2. Cài đặt EF Core

```bash
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL    # PostgreSQL
dotnet add package Microsoft.EntityFrameworkCore.SqlServer  # SQL Server
```

---

## 3. Tạo DbContext

```csharp
using Microsoft.EntityFrameworkCore;

public class AppDbContext : DbContext
{
    public DbSet<User> Users { get; set; }
    public DbSet<Product> Products { get; set; }

    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
}

public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

Đăng kí trong `Program.cs`

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("DefaultConnection")));
```

---

## 4.Migration & Database Update

- Tạo migration:

```bash
dotnet ef migrations add InitialCreate
```

- Update database:

```bash
dotnet ef database update
```

- Xóa migration cuối

```bash
dotnet ef migrations remove
```

## 5. CRUD cơ bản với EF Core

**Thêm dữ liệu**

```csharp
var user = new User {Name = "Alpha Dev"};
db.Users.Add(user);
await db.SaveChangesAsync();
```

**Lấy dữ liệu**

```csharp
var allUsers = await db.Users.ToListAsync();
var user = await db.Users.FindAsync(1);
```

**Sửa dữ liệu**

```csharp
var user = await db.Users.FindAsync(1);
user.Name = "Phan Van Tam";
await db.SaveChangesAsync();
```

**Xóa dữ liệu**

```csharp
var user = await db.Users.FindAsync(1);
db.Users.Remove(user);
await db.SaveChangeAsync();
```

## 6. Quan hệ giữa các bảng

**One-to-Many**

```csharp
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public List<Product> Products { get; set; } = new();
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int UserId { get; set; }
    public User User { get; set; }
}
```

=> Một user tạo nhiều sản phẩm, mỗi sản phẩm thì được tạo bởi một user.

**Many-to-Many**

```csharp
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    public List<Course> Courses { get; set; }
}

public class Course
{
    public int Id { get; set; }
    public string Title { get; set; }
    public List<Student> Students { get; set; }
}
```

=> Một học sinh thì có nhiều khóa học, 1 khóa học lại có nhiều học sinh.

## 8. Best Practices

- Luôn dùng `async/await` (`ToListAsync`, `FindAsync`,...).
- Không dùng `SaveChanges` nhiều lần trong 1 request -> gộp lại.
- Sử dụng DTO thay vì trả trực tiếp entity.
- Khi queery data nhiều -> dùng **AsNoTracking()**
  để tối ưu.
- Tách `DbContext` riêng cho mỗi request (Scoped)

## 9. Tài liệu tham khảo

- [EF Core Docs](https://learn.microsoft.com/ef/core)
- [Migratioins](https://learn.microsoft.com/ef/core/managing-schemas/migrations)
- [RelationShips](https://learn.microsoft.com/ef/core/modeling/relationships)
