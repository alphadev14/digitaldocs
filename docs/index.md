# P&T Digital Docs

Xin chào, mình là **Tâm (TamPV)**. Đây là không gian ghi chú kỹ thuật dành cho quá trình học, xây dựng và vận hành sản phẩm web với **.NET**, **React**, **PostgreSQL**, **Docker**, **CI/CD** và **System Design**.

Mục tiêu của bộ tài liệu này là biến kiến thức rời rạc thành một hệ thống có thể đọc theo lộ trình, tra cứu nhanh khi làm dự án, và đủ thực tế để áp dụng vào môi trường production.

## Hướng đi chính

| Nhóm nội dung | Bạn sẽ học được gì | Nên bắt đầu từ |
| --- | --- | --- |
| Backend .NET | Thiết kế Web API, middleware, DI, EF Core, JWT, Clean Architecture, async/await và performance | [ASP.NET Core Web API](junior/backend/01-aspnet-core-webapi.md) |
| Frontend React | Component, hooks, routing, state management, data fetching, cache và tối ưu render | [React Basic](junior/frontend/01-react-basic.md) |
| Database | PostgreSQL, transaction, isolation level, index, partitioning, JSONB và query plan | [PostgreSQL Basic](junior/database/01-postgresql-basic.md) |
| Testing & Deployment | Unit test, integration test, deploy frontend/backend/database lên cloud | [Unit Test .NET](junior/Testing%20&%20Deployment/01-unit-test-dotnet.md) |
| DevOps | Docker, Nginx, GitLab CI/CD, logging, health check và deployment strategy | [Docker Basic](mid-level/devops/docker/01-docker-basic.md) |
| System Design | Tư duy thiết kế hệ thống, component, trade-off, scalability và reliability | [System Design Introduction](mid-level/system-design/01-system-design-introduction.md) |
| Design Pattern | Singleton, Factory, Builder, Adapter, Observer và cách áp dụng trong .NET backend | [Design Pattern Overview](mid-level/design-pattern/01-design-pattern-overview.md) |

## Lộ trình đọc gợi ý

### 1. Junior - Nắm nền tảng để làm được dự án

Giai đoạn này tập trung vào việc hiểu framework, viết được API, xây được UI, thao tác database và deploy một ứng dụng cơ bản.

1. [ASP.NET Core Web API](junior/backend/01-aspnet-core-webapi.md)
2. [Entity Framework Core](junior/backend/03-entity-framework-core.md)
3. [Authentication với JWT](junior/backend/05-authentication-jwt.md)
4. [React Basic](junior/frontend/01-react-basic.md)
5. [PostgreSQL Basic](junior/database/01-postgresql-basic.md)
6. [Deploy Backend](junior/Testing%20&%20Deployment/03-deploy-backend.md)
7. [Deploy Frontend](junior/Testing%20&%20Deployment/04-deploy-frontend.md)

### 2. Mid-Level - Làm hệ thống dễ maintain và vận hành

Giai đoạn này chuyển từ “biết dùng” sang “biết thiết kế”. Bạn sẽ gặp các chủ đề như API contract, query optimization, caching, background jobs, observability, security, testing strategy và deployment strategy.

Các điểm nên ưu tiên:

- Chuẩn hóa API response, error format, pagination và versioning.
- Hiểu EF Core sinh SQL như thế nào, khi nào cần projection, `AsNoTracking()` và index phù hợp.
- Biết phân loại state trong frontend: local state, global state, server state và URL state.
- Có timeout, retry, health check, logging có correlation id và metric cơ bản.
- Biết rollback, blue-green hoặc rolling deployment ở mức khái niệm và thực hành tối thiểu.

### 3. Senior - Đang mở rộng

Phần Senior hiện là khu vực để phát triển tiếp các chủ đề như architecture decision, distributed system, technical leadership, incident review và cost/performance trade-off.

## Chuẩn chất lượng của tài liệu

Mỗi bài nên trả lời được bốn câu hỏi:

- **Vấn đề là gì?** Bài toán thực tế nào khiến kỹ thuật này cần thiết?
- **Cách làm đúng là gì?** Có ví dụ hoặc checklist để áp dụng.
- **Sai lầm thường gặp là gì?** Nêu rõ rủi ro khi dùng sai.
- **Khi nào không nên dùng?** Tránh biến kiến thức thành over-engineering.

## Stack trọng tâm

| Layer | Công nghệ |
| --- | --- |
| Backend | C#, ASP.NET Core Web API, Entity Framework Core, LINQ, JWT |
| Frontend | React, TypeScript, React Router, Context API, React Query/SWR ở mức định hướng |
| Database | PostgreSQL, transaction, index, partitioning, JSONB, execution plan |
| DevOps | Docker, Docker Compose, Nginx, GitLab CI/CD, Vercel, Render, Linux basics |
| Quality | Unit test, integration test, structured logging, health check, monitoring |

## Liên kết

| Kênh | Link |
| --- | --- |
| Website | [https://tampv.pro.vn](https://tampv.pro.vn/) |
| GitHub | [https://github.com/alphadev14](https://github.com/alphadev14) |
| LinkedIn | [https://www.linkedin.com/in/tampv](https://www.linkedin.com/in/tampv) |
| Source code | [digitaldocs](https://github.com/alphadev14/digitaldocs) |

© 2026 P&T Digital Docs.
