# Async/Await trong `C#`

## 1. Tổng quan

- **Async/Await** là cơ chế hỗ trợ lập trình **bất đồng bộ** trong C# (ra đời từ .NET 4.5)
- Giúp code bất đồng bộ dễ đọc như code đồng bộ, thay vì phải viết callback hoặc `ContinueWith`.

**Từ khóa chính**

- `async`: khai báo một phương thức bất đồng bộ -> phương thức này trả về `Task`, `Task<T>` hoặc `ValueTask`
- `await`: "tạm dừng" thực thi tại đó, chờ `Task` hoàn tất, rồi chạy tiếp. Trong lúc chờ đợi, thread không bị block.

---

## 2. Nguyên lý hoạt động

Giả sử:

```csharp
public async Task<string> GetDataAsync()
{
  var data = await DownloadAsync();
  return data + " processed";
}
```

Diễn giải:

1. `await DownloadAsync()` -> khi `DownLoadAsync()` chưa xong, **trả quyền điều khiển về caller**.
2. `Khi DownloadAsync()` xong -> quay lại chạy `return data + " processed"`.
3. `GetDataAsync() `trả về một `Task<string>` ngay lập tức, caller có thể await hoặc tiếp tục làm việc khác.

👉 Đây gọi là **deferred continuation**: không block thread, chỉ “hẹn” chạy tiếp khi có kết quả.

---

## 3. `Task`, `Task<T>` và `ValueTask`

- `Task`: đại diện cho một công việc async không có giá trị trả về.
- `Task<T>`: có giá trị trả về.
- `ValueTask` (C# 7+): tối ưu hiệu năng cho các method thường xuyên trả về kết quả đồng bộ (không cần Task allocation).

Ví dụ

```csharp
public async Task DoSomeThingAsync() {...}
public async Task<List<int>> GetNumbersAsync() {...}
public async ValueTask<int> GetFastResultAsync() {...}
```

---

## 4. I/O Bound vs CPU Bound

#### 4.1 I/O Bound (phổ biến trong API)

- Chờ resource bên ngoài (HTTP, DB, File, Redis).
- **Không tốn CPU** khi chờ -> `async/await` rất phù hợp.

```csharp
public async Task<User> GetUserAsync(int id)
{
    return await _db.Users.FindAsync(id); // không block thread
}
```

#### 4.2 CPU Bound

- Tính toán nặng, **tiêu tốn CPU**.
- `Async/await` không giúp được -> cần Task.Run để chạy trên thread khác.

```csharp
public async Task<long> SumAsync()
{
    return await Task.Run(() =>
    {
        long sum = 0;
        for (int i = 0; i < 1_000_000_000; i++) sum += i;
        return sum;
    });
}
```

---

## 5. Tóm tắt

- Async/Await giúp viết code bất đồng bộ dễ đọc.
- I/O bound -> dùng async/await.
- CPU bound -> dùng Task.Run hoặc background service.
- ...
