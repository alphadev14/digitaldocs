# LINQ (Language Integrated Query)

## 1. LINQ là gì?

- **LINQ** (Language Integrated Query) là cú pháp truy vấn dữ liệu được tích hợp trực tiếp trong C#.

### Giúp truy vấn trên nhiều nguồn dữ liệu khác nhau

- **LINQ to Objects** → collection, array, list
- **LINQ to Entities** → Entity Framework (database)
- **LINQ to XML**, **LINQ to SQL**, v.v.

### Ưu điểm

- Code ngắn gọn, dễ đọc
- **Type-safe** (có kiểm tra kiểu dữ liệu lúc compile)
- Hỗ trợ **deferred execution** (chỉ chạy khi thực sự duyệt dữ liệu)

## 2. Cú pháp LINQ

Có 2 kiểu chính:

- Query Syntax (Giống SQL)

```csharp
 var result = from n in numbers
              where n % 2 == 0
              orderby n descending
              select n;
```

- Method Syntax (Dùng lambda)

```csharp
var result = numbers
      .Where(n => n%2 == 0)
      .OrderByDescending(m => m)
      .ToList();
```

👉 Thực tế dev thường dùng Method Syntax nhiều hơn.

## 3. Các toán tử LINQ quan trọng

#### a. Filtering (lọc dữ liệu)

```csharp
var evenNumber = numbers.Where(n => n%2 == 0);
```

#### b. Projection (chọn dữ liệu mới)

```csharp
var squares = number.Select(n => n*n);
```

#### c. Sorting (sắp xếp)

```csharp
var sorted = numbers.OrderBy(n => n); // Sắp xếp tăng dần
var sortedDesc= numbers.OrderByDescending(n => n); // Sắp xếp giảm dần
```

#### d. Grouping (nhóm dữ liệu)

```csharp
var grouped = students
.GroupBy(s => s.Class)
.Select(y => new StudentCountBO {
  Class = y.Key,
  Count = y.Count()
}).ToList();
```

#### e. Aggregation (tính toán)

```csharp
int count = numbers.Count();
int sum = numbers.Sum();
int max = number.Max();
double avg = numbers.Average();
```

#### g. Distinct & Set operations

```csharp
var uniqueNumbers = number.Unique();
var common = list1.Intersect(list2); // Lấy ra các phần tử chung của 2 list (Giao của 2 tập hợp).
var all = list1.Union(list2); // Ghép cả 2 tập hợp lại và loại bỏ các phần tử trùng (Hợp của 2 tập hợp).
var except = list1.Except(list2); // Lấy những phần tử có trong list1 mà không có trong list2 (Hiệu của 2 tập hợp).
```

## 4. Deferred Execution

LINQ không thực thi truy vấn ngay khi bạn định nghĩa nó. Thay vào đó, truy vấn chỉ được thực thi khi bạn **duyệt/đòi kết quả** (enumerate) - Ví dụ: `foreach`, `.ToList()`, `.ToArray()`, `Count()`, `.First()`...

Điều này áp dụng hầu hết các toán tử LINQ (như `Where`, `Select`, `OrderBy`...), và cả với **IQueryable** (EF Core) - ở đó **expression tree** được xây dựng và **dịch sang SQL** chỉ khi **enumerate**

Ví dụ

```csharp
var numbers = new List<int> {1,2,3};

// Định nghĩa query chưa chạy
var q = numbers.Where(n => {
  Console.WriteLine($"Filtering {n}");
  return n % 2 == 1;
})

Console.WriteLine("Before enumerate");

// Bắt đầu enumerate => query chạy, predicate in ra khi thực thi
foreach(var number in numbers) {
  Console.WriteLine($"Result: {x}");
}

```

Output:

```csharp
Before enumeration
Filtering 1
Result: 1
Filtering 2
Filtering 3
Result: 3
```

=> predicate chỉ được gọi khi enumerate.

## 5. Tóm tắt ngắn ngọn

- **Deferrend** = `Where`, `Select`, `OrderBy`, `Skip`, `Take`, `Join`...
- **Immediate** = `ToList()`, `ToArray()`, `Count()`, `First()`, `Any()`, `Fisrt()`, `FirstOrDefault()`...
