# Observer Pattern

## 1. Observer Pattern là gì?

Observer Pattern là một Behavioral Design Pattern dùng để thông báo cho nhiều object khác khi một sự kiện xảy ra.

Nói đơn giản:

```text
Một event xảy ra -> nhiều handler được thông báo -> mỗi handler tự xử lý phần việc của mình
```

Pattern này thường xuất hiện dưới dạng event, publish/subscribe, domain event, message bus hoặc event emitter.

## 2. Vấn đề Observer giải quyết

Giả sử sau khi tạo đơn hàng, hệ thống cần:

- Gửi email xác nhận.
- Trừ tồn kho.
- Gửi thông báo cho admin.
- Tạo log/audit trail.

Nếu `OrderService` gọi trực tiếp tất cả service phụ, class này sẽ phình to và coupling rất cao.

```text
OrderService
  -> EmailService
  -> InventoryService
  -> AdminNotificationService
  -> AuditService
```

Observer giúp tách hành động chính khỏi các side effect theo sự kiện.

## 3. Ví dụ Observer đơn giản trong C#

```csharp
public sealed record OrderCreatedEvent(Guid OrderId, string CustomerEmail);

public interface IEventHandler<in TEvent>
{
    Task HandleAsync(TEvent @event);
}

public sealed class SendOrderEmailHandler : IEventHandler<OrderCreatedEvent>
{
    public Task HandleAsync(OrderCreatedEvent @event)
    {
        Console.WriteLine($"Send confirmation email to {@event.CustomerEmail}");
        return Task.CompletedTask;
    }
}

public sealed class UpdateInventoryHandler : IEventHandler<OrderCreatedEvent>
{
    public Task HandleAsync(OrderCreatedEvent @event)
    {
        Console.WriteLine($"Update inventory for order {@event.OrderId}");
        return Task.CompletedTask;
    }
}
```

Publisher tối giản:

```csharp
public sealed class EventPublisher
{
    private readonly IEnumerable<IEventHandler<OrderCreatedEvent>> _handlers;

    public EventPublisher(IEnumerable<IEventHandler<OrderCreatedEvent>> handlers)
    {
        _handlers = handlers;
    }

    public async Task PublishAsync(OrderCreatedEvent @event)
    {
        foreach (var handler in _handlers)
        {
            await handler.HandleAsync(@event);
        }
    }
}
```

Use case:

```csharp
public sealed class OrderService
{
    private readonly EventPublisher _eventPublisher;

    public OrderService(EventPublisher eventPublisher)
    {
        _eventPublisher = eventPublisher;
    }

    public async Task CreateOrderAsync(string customerEmail)
    {
        var orderId = Guid.NewGuid();

        await _eventPublisher.PublishAsync(new OrderCreatedEvent(orderId, customerEmail));
    }
}
```

## 4. Khi nào nên dùng Observer?

| Trường hợp | Có nên dùng? | Ghi chú |
| --- | --- | --- |
| Một hành động kéo theo nhiều side effect | Có | Tạo đơn, gửi email, trừ kho, audit |
| Muốn giảm coupling giữa publisher và handler | Có | Publisher không cần biết handler cụ thể |
| Cần mở rộng handler mới mà ít sửa flow chính | Có | Thêm subscriber mới |
| Flow nghiệp vụ cần transaction chặt chẽ theo thứ tự | Cẩn thận | Có thể cần orchestration rõ hơn |
| Chỉ có một hành động phụ rất đơn giản | Không cần | Gọi trực tiếp có thể dễ đọc hơn |

## 5. Observer trong backend production

Trong production, cần phân biệt hai kiểu:

| Kiểu | Đặc điểm | Ví dụ |
| --- | --- | --- |
| In-process event | Handler chạy trong cùng process/request | Domain event nội bộ, cập nhật cache nhỏ |
| Message/event bus | Event đi qua queue/broker | RabbitMQ, Kafka, Azure Service Bus |

Nếu side effect quan trọng và không được mất, không nên chỉ publish in-memory. Cần cân nhắc Outbox Pattern để lưu event cùng transaction database rồi publish sau.

## 6. Observer khác gì Mediator?

| Tiêu chí | Observer | Mediator |
| --- | --- | --- |
| Mục tiêu | Một event thông báo cho nhiều subscriber | Điều phối giao tiếp giữa nhiều object |
| Quan hệ | Publisher không biết subscriber cụ thể | Các object giao tiếp qua mediator |
| Ví dụ | `OrderCreatedEvent` có nhiều handler | `IMediator.Send(command)` trong CQRS |

## 7. Lưu ý khi dùng

- Handler nên nhỏ, rõ trách nhiệm và dễ retry.
- Cẩn thận với thứ tự chạy handler nếu nghiệp vụ phụ thuộc thứ tự.
- Log event id/correlation id để debug side effect.
- Với handler gọi service ngoài, cần timeout và retry có kiểm soát.
- Nếu publish async qua queue, cần xử lý idempotency để tránh chạy trùng.

## 8. Tóm tắt

Observer Pattern phù hợp khi một sự kiện cần kích hoạt nhiều phản ứng độc lập.

Một câu dễ nhớ:

```text
Observer tốt khi một event có nhiều người quan tâm.
Observer xấu khi flow cần thứ tự chặt chẽ nhưng lại bị tách thành các handler khó kiểm soát.
```
