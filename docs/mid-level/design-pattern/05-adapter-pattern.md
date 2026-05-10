# Adapter Pattern

## 1. Adapter Pattern là gì?

Adapter Pattern là một Structural Design Pattern dùng để chuyển interface không tương thích thành interface mà code hiện tại mong muốn.

Nói đơn giản:

```text
API bên ngoài nói một kiểu -> Adapter dịch lại -> code nội bộ dùng interface quen thuộc
```

Pattern này rất hay gặp khi tích hợp payment gateway, email provider, storage service, SMS/Zalo/Telegram hoặc SDK bên thứ ba.

## 2. Vấn đề Adapter giải quyết

Giả sử hệ thống cần gửi thông báo qua nhiều kênh. Code nội bộ muốn dùng interface chung:

```csharp
public interface INotificationSender
{
    Task SendAsync(string receiver, string message);
}
```

Nhưng SDK bên ngoài lại có API khác:

```csharp
public sealed class TelegramClient
{
    public Task PushMessageAsync(long chatId, string text)
    {
        return Task.CompletedTask;
    }
}
```

Nếu service chính gọi trực tiếp `TelegramClient`, domain/application layer sẽ bị phụ thuộc vào chi tiết của bên thứ ba.

## 3. Ví dụ Adapter trong C#

```csharp
public sealed class TelegramNotificationAdapter : INotificationSender
{
    private readonly TelegramClient _telegramClient;

    public TelegramNotificationAdapter(TelegramClient telegramClient)
    {
        _telegramClient = telegramClient;
    }

    public Task SendAsync(string receiver, string message)
    {
        var chatId = long.Parse(receiver);

        return _telegramClient.PushMessageAsync(chatId, message);
    }
}
```

Service chính chỉ biết `INotificationSender`:

```csharp
public sealed class NotificationService
{
    private readonly INotificationSender _sender;

    public NotificationService(INotificationSender sender)
    {
        _sender = sender;
    }

    public Task NotifyAsync(string receiver, string message)
    {
        return _sender.SendAsync(receiver, message);
    }
}
```

Lợi ích:

- Code nghiệp vụ không biết Telegram API gọi method nào.
- Dễ thay Telegram bằng Email, SMS hoặc provider khác.
- Dễ mock `INotificationSender` khi test.

## 4. Adapter trong Clean Architecture

Trong Clean Architecture, Adapter thường nằm ở Infrastructure layer.

```text
Application layer -> interface nội bộ -> Infrastructure adapter -> external API/SDK
```

Ví dụ:

- `IPaymentGateway` được application layer định nghĩa.
- `VnPayGatewayAdapter` implement interface đó và gọi API VNPAY.
- `MomoGatewayAdapter` implement interface đó và gọi API MoMo.

Nhờ vậy use case không phụ thuộc trực tiếp vào SDK hoặc HTTP contract của bên ngoài.

## 5. Khi nào nên dùng Adapter?

| Trường hợp | Có nên dùng? | Ghi chú |
| --- | --- | --- |
| Tích hợp SDK/API bên thứ ba | Có | Che giấu chi tiết bên ngoài |
| Interface cũ không khớp interface mới | Có | Adapter giúp tương thích dần |
| Muốn test use case mà không gọi service thật | Có | Mock interface nội bộ |
| API đã khớp hoàn toàn với code mình | Không cần | Adapter có thể thừa |
| Muốn gom nhiều bước nghiệp vụ phức tạp | Không hẳn | Facade thường hợp hơn |

## 6. Adapter khác gì Facade?

| Tiêu chí | Adapter | Facade |
| --- | --- | --- |
| Mục tiêu | Chuyển interface không tương thích thành interface mình cần | Che hệ thống con phức tạp bằng API đơn giản |
| Thường dùng khi | Tích hợp thư viện/API ngoài | Một flow cần gọi nhiều service con |
| Ví dụ | Bọc Telegram SDK thành `INotificationSender` | `CheckoutFacade` gọi cart, payment, shipping, email |

## 7. Lưu ý khi dùng

- Adapter nên mỏng, tập trung vào chuyển đổi input/output và gọi API ngoài.
- Không nhồi business logic chính vào Adapter.
- Nên normalize lỗi từ provider ngoài thành exception/result mà application layer hiểu được.
- Với HTTP API, adapter nên có timeout, retry có kiểm soát và logging phù hợp.

## 8. Tóm tắt

Adapter Pattern giúp code nội bộ không bị rò rỉ chi tiết của hệ thống bên ngoài.

Một câu dễ nhớ:

```text
Adapter tốt khi hai interface không khớp nhưng vẫn cần làm việc cùng nhau.
Adapter xấu khi nó trở thành nơi chứa cả nghiệp vụ chính lẫn logic tích hợp.
```
