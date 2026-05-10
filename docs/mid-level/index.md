# Mid-Level Roadmap

Mid-Level là giai đoạn chuyển từ “làm được feature” sang “thiết kế được hệ thống dễ maintain, dễ debug, dễ mở rộng và có thể vận hành trong production”.

## Năng lực trọng tâm

| Mảng | Cần nâng cấp |
| --- | --- |
| Backend | API contract, versioning, validation, authorization, EF Core optimization, cache, background jobs |
| Frontend | Component architecture, server state, data fetching, performance, accessibility và testing |
| Database | Schema design, indexing, transaction, migration strategy, slow query và capacity planning |
| DevOps | Docker Compose, CI/CD, Nginx, logging, health check, rollback và deployment strategy |
| System Design | Component, data flow, trade-off, reliability, scalability và observability |
| Design Pattern | Nhận diện vấn đề trong code và chọn pattern vừa đủ, không over-engineering |

## Cách học hiệu quả

1. Học theo vấn đề thực tế, không học theo tên kỹ thuật trước.
2. Với mỗi topic, luôn hỏi: “nếu production lỗi thì mình debug bằng gì?”.
3. Ưu tiên checklist áp dụng được: naming, error format, pagination, timeout, retry, log, metric.
4. Sau mỗi bài, thử áp dụng vào một project nhỏ để thấy trade-off.

## Checklist Mid-Level

- API có request/response/error format nhất quán.
- Query list có pagination, filter rõ ràng và tránh load dư dữ liệu.
- Có chiến lược cache, invalidation và TTL.
- External call có timeout, retry có kiểm soát và logging.
- Job chạy nền idempotent, có retry và trạng thái rõ ràng.
- Frontend phân biệt được local state, global state, server state và URL state.
- Database migration có kế hoạch rollback hoặc backward compatibility.
- Pipeline có build, test, deploy và quản lý environment.
- Hệ thống có log, health check và metric tối thiểu.
