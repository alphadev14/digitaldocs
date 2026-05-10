# Design Pattern Documentation Blueprint

File này dùng làm kiến trúc chung để viết các bài Design Pattern tiếp theo, theo format của bài `02-singleton-pattern.md`.

Mục tiêu:

- Mỗi pattern có cùng một cấu trúc dễ đọc.
- Ví dụ ưu tiên C# / ASP.NET Core vì tài liệu đang hướng tới backend .NET / fullstack.
- Luôn có phần khi nào nên dùng, khi nào không nên dùng và lưu ý thực tế.
- Tránh viết pattern như lý thuyết khô; mỗi bài cần gắn với tình huống code thật.

---

## 1. Format chuẩn cho một bài pattern

Mỗi file pattern nên đi theo cấu trúc sau:

```text
# [Pattern Name] Pattern

## 1. [Pattern Name] Pattern là gì?
## 2. [Pattern Name] giải quyết vấn đề gì?
## 3. Đặc điểm chính
## 4. Ví dụ cơ bản trong C#
## 5. Luồng hoạt động / sơ đồ minh họa
## 6. Ví dụ trong ASP.NET Core / ứng dụng thực tế
## 7. Khi nào nên dùng [Pattern Name]?
## 8. Khi nào không nên dùng [Pattern Name]?
## 9. Biến thể / cách triển khai phổ biến
## 10. So sánh với pattern hoặc cách làm gần giống
## 11. Ưu điểm
## 12. Nhược điểm
## 13. Lưu ý khi sử dụng
## 14. Ví dụ thực tế hoàn chỉnh
## 15. Ví dụ sai thường gặp
## 16. Tóm tắt
```

Không phải pattern nào cũng bắt buộc đủ 16 mục, nhưng nên giữ thứ tự tư duy giống nhau để người đọc quen nhịp.

---

## 2. Chi tiết từng phần

## 2.1 Định nghĩa

Nên viết ngắn, rõ, dễ nhớ.

Format gợi ý:

```text
[Pattern Name] là một [Creational/Structural/Behavioral] Design Pattern dùng để ...
```

Sau đó thêm một câu "Nói đơn giản":

```text
Nói đơn giản:

Input / vấn đề -> Pattern xử lý -> Output / kết quả
```

Ví dụ với Builder:

```text
Object phức tạp -> tạo từng bước -> nhận object hoàn chỉnh
```

---

## 2.2 Vấn đề pattern giải quyết

Phần này nên trả lời:

- Nếu không dùng pattern thì code xấu ở đâu?
- Code bị trùng lặp, khó test, khó mở rộng hay khó đọc?
- Pattern giúp gom trách nhiệm hoặc tách phụ thuộc như thế nào?

Nên có một ví dụ đời thực hoặc ví dụ trong backend.

---

## 2.3 Đặc điểm chính

Dùng bullet ngắn.

Ví dụ:

```text
Một Builder thường có các đặc điểm:

- Tách quá trình tạo object khỏi object chính.
- Cho phép tạo object từng bước.
- Giúp code khởi tạo dễ đọc hơn khi object có nhiều tham số.
```

---

## 2.4 Ví dụ C# cơ bản

Mỗi bài nên có ít nhất một ví dụ C# tối giản.

Quy ước:

- Code ngắn, tập trung vào pattern.
- Tên class dễ hiểu.
- Không dùng domain quá phức tạp ở ví dụ đầu tiên.
- Sau code phải có phần giải thích.

Format:

```csharp
// Code example
```

```text
Giải thích:

- Dòng / class này có vai trò gì.
- Pattern nằm ở đâu.
- Caller sử dụng pattern như thế nào.
```

---

## 2.5 Sơ đồ minh họa

Nếu pattern có luồng xử lý hoặc quan hệ class rõ ràng, nên thêm hình.

Quy ước lưu hình:

```text
docs/assets/[pattern-name]-[short-description].svg
docs/assets/[pattern-name]-[short-description].png
docs/assets/[pattern-name]-[short-description].webp
```

Quy ước nhúng hình trong markdown:

```html
<p align="center">
  <img src="../../../assets/[file-name].svg" alt="[Mô tả hình]" width="700">
</p>
```

Lưu ý:

- Với file trong `docs/mid-level/design-pattern`, đường dẫn tới assets là `../../../assets/...`.
- Nên thêm đoạn giải thích ngay sau hình.
- Không để hình đứng một mình.

---

## 2.6 Ví dụ ASP.NET Core / thực tế

Nếu pattern hay gặp trong .NET backend, nên có thêm ví dụ thực tế:

- DI registration
- Service layer
- Repository
- Middleware
- Background job
- Cache
- Logging
- Validation
- Payment / shipping / notification flow

Ví dụ format:

```csharp
builder.Services.AddScoped<IOrderService, OrderService>();
```

Sau đó giải thích vòng đời hoặc cách inject.

---

## 2.7 Khi nào nên dùng?

Phần này nên có bullet và bảng.

Format:

```text
Nên dùng [Pattern Name] khi:

- ...
- ...
- ...
```

Bảng gợi ý:

```markdown
| Trường hợp | Có nên dùng? | Ghi chú |
| ---------- | ------------ | ------- |
| ... | Có | ... |
| ... | Không | ... |
```

---

## 2.8 Khi nào không nên dùng?

Phần này rất quan trọng để tránh lạm dụng pattern.

Nên viết rõ:

- Pattern làm code phức tạp hơn khi nào?
- Khi nào chỉ cần code đơn giản?
- Khi nào pattern bị dùng sai mục đích?

Mỗi bài nên có một ví dụ sai thường gặp.

---

## 2.9 So sánh với cách làm gần giống

Nếu pattern dễ bị nhầm, nên có bảng so sánh.

Ví dụ:

- Singleton vs Static Class
- Factory vs Builder
- Factory Method vs Abstract Factory
- Strategy vs State
- Adapter vs Facade
- Decorator vs Proxy
- Observer vs Mediator

Format:

```markdown
| Tiêu chí | Pattern A | Pattern B |
| -------- | --------- | --------- |
| Mục đích | ... | ... |
| Khi dùng | ... | ... |
```

---

## 2.10 Ưu điểm, nhược điểm, lưu ý

Ba phần này nên ngắn nhưng thực tế.

Checklist nên có:

```text
1. Pattern này có làm code dễ hiểu hơn không?
2. Có đang over-engineering không?
3. Có ảnh hưởng test không?
4. Có phù hợp với Dependency Injection không?
5. Có vấn đề thread-safe, state hoặc lifecycle không?
```

---

## 2.11 Tóm tắt

Phần cuối nên có:

- Một đoạn tóm tắt ngắn.
- 3-5 ý nên nhớ.
- Một câu dễ nhớ.

Format:

```text
Một câu dễ nhớ:

[Pattern Name] tốt khi ...
[Pattern Name] xấu khi ...
```

---

## 3. Thứ tự các pattern nên viết tiếp

Thứ tự đề xuất cho tài liệu mid-level:

| Thứ tự | File | Pattern | Nhóm | Mức ưu tiên |
| ------ | ---- | ------- | ---- | ----------- |
| 01 | `01-design-pattern-overview.md` | Overview | Tổng quan | Đã có |
| 02 | `02-singleton-pattern.md` | Singleton | Creational | Đã có |
| 03 | `03-builder-pattern.md` | Builder | Creational | Rất nên viết |
| 04 | `04-factory-pattern.md` | Factory Method | Creational | Rất nên viết |
| 05 | `05-adapter-pattern.md` | Adapter | Structural | Rất nên viết |
| 06 | `06-facade-pattern.md` | Facade | Structural | Rất nên viết |
| 07 | `07-strategy-pattern.md` | Strategy | Behavioral | Rất nên viết |
| 08 | `08-observer-pattern.md` | Observer | Behavioral | Rất nên viết |
| 09 | `09-decorator-pattern.md` | Decorator | Structural | Nên viết |
| 10 | `10-chain-of-responsibility-pattern.md` | Chain of Responsibility | Behavioral | Nên viết |
| 11 | `11-command-pattern.md` | Command | Behavioral | Nên viết |
| 12 | `12-template-method-pattern.md` | Template Method | Behavioral | Nên viết |
| 13 | `13-state-pattern.md` | State | Behavioral | Nên viết |
| 14 | `14-proxy-pattern.md` | Proxy | Structural | Nên biết |
| 15 | `15-abstract-factory-pattern.md` | Abstract Factory | Creational | Học sau Factory |
| 16 | `16-prototype-pattern.md` | Prototype | Creational | Học sau |
| 17 | `17-bridge-pattern.md` | Bridge | Structural | Học sau |
| 18 | `18-composite-pattern.md` | Composite | Structural | Học sau |
| 19 | `19-flyweight-pattern.md` | Flyweight | Structural | Học sau |
| 20 | `20-mediator-pattern.md` | Mediator | Behavioral | Học sau |
| 21 | `21-visitor-pattern.md` | Visitor | Behavioral | Khó, học cuối |
| 22 | `22-memento-pattern.md` | Memento | Behavioral | Ít gặp |

---

## 4. Blueprint riêng cho các pattern sắp viết

## 4.1 Builder Pattern

Nên tập trung vào:

- Object có nhiều field hoặc optional parameters.
- Constructor quá dài.
- Fluent API.
- So sánh Builder với object initializer và Factory.
- Ví dụ: tạo `Order`, `EmailMessage`, `ReportRequest`.

Các mục nên có:

```text
1. Builder là gì?
2. Vấn đề constructor quá dài
3. Builder hoạt động như thế nào?
4. Ví dụ Product + Builder
5. Fluent Builder
6. Builder trong thực tế .NET
7. Khi nên dùng / không nên dùng
8. Builder vs Factory
9. Lưu ý tránh over-engineering
10. Tóm tắt
```

---

## 4.2 Factory Pattern

Nên tập trung vào:

- Tách logic tạo object ra khỏi nơi sử dụng.
- Chọn implementation theo loại.
- Giảm `if else` rải rác.
- Ví dụ: payment method, notification sender, export file.

Các mục nên có:

```text
1. Factory là gì?
2. Vấn đề tạo object rải rác
3. Simple Factory
4. Factory Method
5. Ví dụ C# payment
6. Factory trong ASP.NET Core DI
7. Khi nên dùng / không nên dùng
8. Factory vs Builder
9. Ví dụ sai thường gặp
10. Tóm tắt
```

---

## 4.3 Adapter Pattern

Nên tập trung vào:

- Chuyển interface không tương thích thành interface mình cần.
- Tích hợp thư viện bên thứ ba.
- Che giấu API ngoài.
- Ví dụ: payment gateway, email provider, storage provider.

Các mục nên có:

```text
1. Adapter là gì?
2. Vấn đề interface không khớp
3. Object Adapter
4. Ví dụ tích hợp external payment
5. Adapter trong clean architecture
6. Khi nên dùng / không nên dùng
7. Adapter vs Facade
8. Lưu ý khi wrap thư viện ngoài
9. Tóm tắt
```

---

## 4.4 Facade Pattern

Nên tập trung vào:

- Che giấu hệ thống con phức tạp.
- Tạo API đơn giản cho caller.
- Ví dụ: checkout flow gọi inventory, payment, shipping, email.
- So sánh với Adapter.

Các mục nên có:

```text
1. Facade là gì?
2. Vấn đề caller phải biết quá nhiều service
3. Facade gom orchestration như thế nào?
4. Ví dụ CheckoutFacade
5. Facade trong service layer
6. Khi nên dùng / không nên dùng
7. Facade vs Adapter
8. Ví dụ sai: God Facade
9. Tóm tắt
```

---

## 4.5 Strategy Pattern

Nên tập trung vào:

- Thay đổi thuật toán/hành vi tại runtime.
- Loại bỏ nhiều `if else` hoặc `switch`.
- Ví dụ: pricing, discount, shipping fee, sorting, payment.

Các mục nên có:

```text
1. Strategy là gì?
2. Vấn đề nhiều if else theo loại
3. Interface strategy
4. Concrete strategies
5. Context sử dụng strategy
6. Strategy với DI
7. Khi nên dùng / không nên dùng
8. Strategy vs State
9. Ví dụ sai thường gặp
10. Tóm tắt
```

---

## 4.6 Observer Pattern

Nên tập trung vào:

- Publish/Subscribe.
- Một sự kiện xảy ra, nhiều handler phản ứng.
- Giảm coupling giữa publisher và subscriber.
- Ví dụ: order created -> send email, update stock, notify admin.

Các mục nên có:

```text
1. Observer là gì?
2. Vấn đề một action kéo theo nhiều side effects
3. Publisher / Subscriber
4. Ví dụ C# event
5. Ví dụ domain event trong backend
6. Observer trong frontend / React
7. Khi nên dùng / không nên dùng
8. Observer vs Mediator
9. Lưu ý async/event bus
10. Tóm tắt
```

---

## 5. Quy ước viết code example

Ưu tiên:

- Code C# hiện đại, dễ đọc.
- Dùng interface khi pattern cần tách abstraction.
- Không đưa quá nhiều framework vào ví dụ đầu tiên.
- Ví dụ framework đặt ở phần sau.
- Tên class thể hiện rõ domain.

Không nên:

- Viết ví dụ quá trừu tượng như `ClassA`, `ClassB`.
- Nhồi nhiều pattern trong một ví dụ đầu tiên.
- Dùng pattern chỉ để chứng minh pattern.
- Bỏ qua phần giải thích sau code.

---

## 6. Quy ước giọng văn

Giữ giọng giống bài Singleton:

- Dùng tiếng Việt dễ hiểu.
- Có câu "Nói đơn giản".
- Có bảng khi cần so sánh.
- Có checklist trước khi dùng.
- Có ví dụ sai để người đọc tránh lạm dụng.
- Kết luận bằng câu dễ nhớ.

Mẫu kết bài:

```text
Một câu dễ nhớ:

[Pattern Name] tốt khi ...
[Pattern Name] xấu khi ...
```
