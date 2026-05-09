# Background Jobs & Queues

## Mục tiêu

Không phải tác vụ nào cũng nên xử lý ngay trong request. Ở Mid-Level, bạn cần biết tách tác vụ lâu hoặc dễ fail sang background job/queue để API phản hồi nhanh, retry được và không làm người dùng chờ vô ích.

---

## 1. Khi nào cần background job?

Nên dùng background job khi:

- Gửi email/SMS/push notification.
- Import/export file.
- Đồng bộ dữ liệu với hệ thống khác.
- Xử lý ảnh/video.
- Tính report nặng.
- Gọi external API lâu hoặc có retry.
- Xử lý webhook cần đảm bảo không mất message.

Không nên xử lý trong request:

```txt
User upload file
  -> API parse toàn bộ file
  -> Insert hàng chục nghìn records
  -> Gửi email
  -> Response sau 2 phút
```

Nên:

```txt
User upload file
  -> API lưu file + tạo job
  -> Enqueue message
  -> Return 202 Accepted + jobId
  -> Worker xử lý
  -> User check job status
```

---

## 2. BackgroundService trong .NET

Ví dụ worker chạy liên tục:

```csharp
public sealed class OrderSyncWorker : BackgroundService
{
    private readonly IServiceProvider _serviceProvider;
    private readonly ILogger<OrderSyncWorker> _logger;

    public OrderSyncWorker(
        IServiceProvider serviceProvider,
        ILogger<OrderSyncWorker> logger)
    {
        _serviceProvider = serviceProvider;
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            using var scope = _serviceProvider.CreateScope();
            var syncService = scope.ServiceProvider.GetRequiredService<IOrderSyncService>();

            await syncService.SyncPendingOrdersAsync(stoppingToken);
            await Task.Delay(TimeSpan.FromSeconds(10), stoppingToken);
        }
    }
}
```

Lưu ý quan trọng:

- `BackgroundService` là singleton.
- Nếu cần `DbContext` hoặc scoped service, phải tạo scope.
- Luôn truyền `stoppingToken`.

---

## 3. Queue-based Processing

Queue giúp tách producer và consumer.

```txt
API
  -> Publish message
  -> Queue
  -> Worker consume
  -> Process
```

Các công cụ phổ biến:

- RabbitMQ.
- Kafka.
- Azure Service Bus.
- AWS SQS.
- Redis queue.
- Hangfire.

Chọn đơn giản:

| Nhu cầu                 | Gợi ý                 |
| ----------------------- | --------------------- |
| Job đơn giản trong .NET | Hangfire              |
| Message queue phổ thông | RabbitMQ              |
| Event streaming lớn     | Kafka                 |
| Cloud managed queue     | SQS/Azure Service Bus |

---

## 4. Job Status

Với tác vụ người dùng cần theo dõi, nên lưu job status.

```txt
Pending
Processing
Completed
Failed
Canceled
```

Ví dụ bảng:

```txt
ImportJobs
- Id
- FileName
- Status
- TotalRows
- ProcessedRows
- ErrorMessage
- CreatedAt
- StartedAt
- CompletedAt
```

API:

```http
POST /api/import-products
GET /api/import-jobs/{jobId}
```

---

## 5. Retry và Dead Letter

Không phải lỗi nào cũng retry.

Nên retry:

- Timeout external API.
- Network temporary error.
- Database transient error.

Không nên retry mù quáng:

- Validation error.
- Data format sai.
- Permission denied.
- Business rule fail.

Sau retry limit, message nên vào dead-letter queue hoặc trạng thái failed để điều tra.

---

## 6. Idempotent Job

Job có thể chạy lại do retry. Vì vậy job nên idempotent.

Ví dụ import product:

- Dùng unique key như SKU.
- Upsert thay vì insert mù.
- Lưu processed row hoặc processed event id.
- Không gửi email nhiều lần nếu retry.

Sai:

```txt
Retry payment job -> charge tiền lần 2
```

Đúng:

```txt
Retry payment job -> kiểm tra transactionId đã xử lý chưa
```

---

## 7. Outbox Pattern

Vấn đề:

```txt
Save order to DB thành công
Publish message thất bại
=> order tồn tại nhưng event không được gửi
```

Outbox pattern:

```txt
Transaction:
  - Save order
  - Save outbox event

Background worker:
  - Read unsent outbox event
  - Publish message
  - Mark as sent
```

Pattern này giúp đảm bảo event không bị mất khi DB save thành công nhưng publish message fail.

---

## Checklist

- Tác vụ lâu được tách khỏi request.
- API trả `202 Accepted` nếu xử lý async.
- Job có status để theo dõi.
- Worker dùng `IServiceScope` khi cần scoped service.
- Có retry limit.
- Có dead-letter hoặc failed state.
- Job idempotent.
- Có log theo `jobId`.
- Có outbox pattern cho event quan trọng.

---

## Bài thực hành

- Xây flow import product bằng background job.
- API upload file trả `jobId`.
- Worker xử lý file theo batch.
- Lưu status và progress.
- Thêm retry cho lỗi transient.
- Đảm bảo retry không insert trùng SKU.
