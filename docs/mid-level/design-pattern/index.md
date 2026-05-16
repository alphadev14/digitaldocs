# Design Pattern Roadmap

Design Pattern không phải là công thức để nhét vào mọi nơi. Nó là ngôn ngữ chung để mô tả các cách tổ chức code đã được kiểm chứng khi gặp vấn đề lặp lại trong thiết kế phần mềm.

## Cách học trong repo này

| Bài | Trọng tâm | Khi nào nên đọc |
| --- | --- | --- |
| [Overview](01-design-pattern-overview.md) | Phân loại pattern và dấu hiệu nhận biết trong code thật | Đọc đầu tiên |
| [Singleton](02-singleton-pattern.md) | Một instance dùng chung, DI lifetime, thread safety | Khi làm config, cache, logger |
| [Factory](03-factory-pattern.md) | Chọn implementation/object theo input hoặc runtime condition | Khi có nhiều loại payment, sender, exporter |
| [Builder](04-builder-pattern.md) | Tạo object phức tạp theo từng bước | Khi constructor quá dài hoặc nhiều optional field |
| [Adapter](05-adapter-pattern.md) | Bọc API/thư viện ngoài để khớp interface nội bộ | Khi tích hợp payment, email, storage, API bên thứ ba |
| [Observer](06-observer-pattern.md) | Một event xảy ra, nhiều handler phản ứng | Khi tạo đơn hàng xong cần gửi email, trừ kho, notify |
| [Strategy](07-strategy-pattern.md) | Nhiều cách xử lý cho cùng một hành động | Khi có pricing, payment, export hoặc validation thay đổi theo ngữ cảnh |
| [Facade](08-facade-pattern.md) | Gom nhiều service con sau một API đơn giản hơn | Khi một use case phải điều phối nhiều bước như booking, checkout, onboarding |
| [CQRS](11-cqrs-pattern.md) | Tách luồng đọc và ghi để mỗi bên tối ưu theo mục tiêu riêng | Khi read model và write model bắt đầu khác nhau rõ rệt |

## Nguyên tắc áp dụng

- Bắt đầu từ mùi code: `if/else` lặp lại, constructor dài, phụ thuộc framework bị rò vào domain, side effect dính chặt vào use case.
- Chọn pattern nhỏ nhất đủ giải quyết vấn đề.
- Ưu tiên code dễ đọc và dễ test hơn việc “đúng sách”.
- Trong ASP.NET Core, luôn cân nhắc Dependency Injection, service lifetime và testability.

## Câu hỏi trước khi dùng pattern

1. Pattern này có làm code dễ hiểu hơn không?
2. Có giảm coupling hoặc duplication thật không?
3. Có làm test dễ hơn không?
4. Có đang tạo thêm class chỉ vì muốn code trông “xịn” hơn không?
5. Nếu thêm yêu cầu mới, pattern này giúp mở rộng hay làm phức tạp thêm?
