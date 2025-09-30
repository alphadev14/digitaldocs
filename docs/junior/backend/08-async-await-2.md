# Async/Await phần 2

Trong bài trước, mình đã tìm hiểu về Async/Await trong C#. Bài này mình sẽ đi chi tiết hơn về các thuật ngữ đã nhắn đến trong bài trước.

## 1. Ý nghĩa của **Task** trong **C#**

- `Task` = đại diện cho môt **công việc bất đồng bộ** đang hoặc sẽ chạy.
- Có thể coi `Task` như một **lời hứa (promise)** rằng "công việc này sẽ được hoàn thành trong tương lai".

👉 Ý nghĩa: thay vì phải chờ xong ngay, Task cho phép bạn để đó và làm việc khác.

---

## 2. Caller là gì?

- **caller** = nơi gọi đến method async.
- Khi bạn `await` một async method, nếu nó chưa xong -> quyền điều khiển được trả về lại cho caller để tiếp tục làm việc khác, không bị chặn.

Ví dụ:

```csharp
public async Task<string> DownloadAsync() {
  await Task.Delay(2000); // giả lập tải dữ liệu
  return "Data";
}

public async Task RunAsync();
{
  Console.WriteLine("Start");

  var result = await DownloadAsync(); // caller = RunAsync()
  Console.WriteLine(result);

  Console.WriteLine("End");
}
```

**Ở đây**

- `RunAsync()` gọi `DownloadAsync()` -> RunAsync chính là **caller**.
- Khi `DownloadAsync()` đang `await Task.Delay(2000)` -> nó trả về quyền điểu khiển cho `RunAsync()`, cho phép RunAsync không bị block thread.

👉 Ý nghĩa: hệ thống không bị “đứng chờ”, có thể phục vụ request khác.

---

Minh họa:

<p align="center">
  <img src="/digitaldocs/assets/async-await-image-1.png" alt="Cấu hình biến môi trường" width="600">
</p>

<p align="center">
  <img src="/digitaldocs/assets/async-await-image-2.png" alt="Cấu hình biến môi trường" width="600">
</p>

---

## 3. Thread là gì?

- **Thread** = luồng thực thi nhỏ nhất trong chương trình.
- CPU có thể chạy nhiều thread song song (đa luồng).
- Một ứng dụng .NET Core có thể có nhiều thread: **Main thread** (luồng chính), **ThreadPool threads** (luồng dùng cho `async/await`, xử lý song song).

- Khi bạn chạy `async/await`: Nếu **I/O bound** -> thread nhường lại, không bị giữ. Nếu **CPU bound** -> thread phải chạy tính toán, tốn CPU

## 4. Khi nào thì tốn CPU?

#### Tốn CPU khi công việc phải tính toán nặng.

- Vòng lặp lớn (for hàng triệu lần).
- Xử lý ảnh, video, AI, thuật toán mã hóa...
- Phân tích dữ liệu, machine learning...

Ví dụ:

```csharp
for (int i = 0; i < 1000000000; i++)
  sum += i; // tốn CPU vì phải tính toán
```

### Không tốn CPU khi chỉ chờ I/O.

- Gọi API bên ngoài.
- Đọc/ghi file, database.
- Gửi request qua network.

Ví dụ:

```csharp
await httpClient.GetStringAsync("https://api.example.com");
```

👉 Trong lúc chờ server phản hồi, CPU rảnh có thể xử lý việc khác.

---

## 5. Tóm tắt

- Task = lời hứa về một công việc sẽ hoàn thành.
- Caller = nơi gọi method async. Async method chưa xong thì trả quyền điều khiển về caller.
- Thread = luồng chạy trong CPU. Async/await giúp không block thread.
- CPU tốn tài nguyên khi phải tính toán nặng (CPU bound), còn khi chỉ chờ I/O (API/DB/file...) thì không tốn CPU.
