# 🧠 Mid-Level Backend Roadmap

Ở Junior, bạn đã học cách tạo Web API, dùng Controller, Service, Repository, EF Core CRUD, JWT, middleware, DI và async/await. Sang Mid-Level, mục tiêu không còn là "biết dùng" nữa, mà là **thiết kế backend có khả năng maintain, scale, debug và vận hành trong production**.

---

## Tư duy chuyển từ Junior sang Mid-Level

| Junior tập trung | Mid-Level cần nâng cấp |
| --- | --- |
| Tạo endpoint CRUD | Thiết kế API contract ổn định, versioning, error format |
| Biết DI và middleware | Hiểu pipeline, lifetime, scoped service, cross-cutting concerns |
| EF Core CRUD | Query optimization, transaction, concurrency, migration strategy |
| LINQ cơ bản | Hiểu `IQueryable`, deferred execution, SQL generated, projection |
| JWT login cơ bản | Refresh token, authorization policy, claims, token storage, security risks |
| Async/await | ThreadPool, concurrency limit, cancellation token, avoiding sync-over-async |
| Log lỗi đơn giản | Structured logging, tracing, metrics, correlation id |
| Service/repository | Clean architecture boundary, domain logic, integration patterns |

---

## Những topic nên học

| Topic | Vì sao quan trọng |
| --- | --- |
| [API Architecture](01-api-architecture.md) | API là contract với frontend/mobile/service khác, cần ổn định và dễ mở rộng |
| [EF Core Query Optimization](02-ef-core-query-optimization.md) | Phần lớn bottleneck backend đến từ database/query |
| [Caching Strategy](03-caching-strategy.md) | Giảm tải DB/API, tăng tốc read-heavy workload |
| [Background Jobs & Queues](04-background-jobs-and-queues.md) | Tách tác vụ lâu khỏi request, xử lý retry và job status |
| [Resilience & Observability](05-resilience-and-observability.md) | Giúp hệ thống chịu lỗi và debug được trong production |
| [Security & Authorization](06-security-and-authorization.md) | Bảo vệ API, xử lý token, quyền và dữ liệu nhạy cảm |
| [Backend Testing Strategy](07-backend-testing-strategy.md) | Tự tin refactor và giảm regression |
| [Performance trong .NET Backend](performance-in-backend.md) | Hiểu async, ThreadPool, GC, memory, HttpClient và monitoring |

---

## Lộ trình học gợi ý

1. **API contract trước**: chuẩn hóa request/response/error/pagination/filtering.
2. **Data access tiếp theo**: tối ưu EF Core, transaction, concurrency và migration.
3. **Security**: refresh token, claims, authorization policy, secret management.
4. **Reliability**: timeout, retry, circuit breaker, idempotency.
5. **Scalability**: cache, background jobs, queue, concurrency limit.
6. **Observability**: structured log, metrics, tracing, correlation id.
7. **Testing**: unit test business logic, integration test API/database, contract test.

---

## Checklist Mid-Level Backend

- API có response format nhất quán.
- Controller mỏng, business logic nằm ở application/domain service.
- Query list luôn có pagination.
- Query read-only dùng `AsNoTracking()` và projection.
- Có transaction rõ ràng cho use case cần atomic.
- External API call có timeout, retry hợp lý và logging.
- Không dùng `.Result` hoặc `.Wait()` trong request flow.
- Có cancellation token cho request/DB/API call quan trọng.
- Có cache TTL và invalidation strategy.
- Background job idempotent và có retry/dead-letter handling.
- Auth có refresh token và authorization policy.
- Có structured logging, metric, health check.
- Có unit/integration test cho flow quan trọng.

