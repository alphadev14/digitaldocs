# 🚀 P&T Digital Docs

Xin chào, mình là **Tâm (TamPV)**, một Fullstack Developer tập trung vào hệ sinh thái **.NET**, **React** và **PostgreSQL**. Đây là không gian mình dùng để ghi lại kiến thức, kinh nghiệm triển khai thực tế và các ghi chú kỹ thuật trong quá trình xây dựng sản phẩm web.

> Mục tiêu của Digital Docs là biến kiến thức rời rạc thành một hệ thống tài liệu dễ đọc, dễ tra cứu và có thể áp dụng ngay vào dự án thật.

<p>
  <span style="background:#512BD4;color:white;padding:6px 10px;border-radius:999px;font-size:13px;">.NET</span>
  <span style="background:#61DAFB;color:#102A43;padding:6px 10px;border-radius:999px;font-size:13px;">React</span>
  <span style="background:#3178C6;color:white;padding:6px 10px;border-radius:999px;font-size:13px;">TypeScript</span>
  <span style="background:#336791;color:white;padding:6px 10px;border-radius:999px;font-size:13px;">PostgreSQL</span>
  <span style="background:#2496ED;color:white;padding:6px 10px;border-radius:999px;font-size:13px;">Docker</span>
  <span style="background:#FC6D26;color:white;padding:6px 10px;border-radius:999px;font-size:13px;">GitLab CI/CD</span>
</p>

---

## ✨ Tổng quan

| Icon | Hạng mục | Nội dung |
| --- | --- | --- |
| 👨‍💻 | Vai trò | Fullstack Developer |
| 🧠 | Backend | .NET, ASP.NET Core Web API, Entity Framework Core, LINQ, Clean Architecture |
| ⚛️ | Frontend | React, Next.js, TypeScript, React Router, Context API |
| 🗄️ | Database | PostgreSQL, transaction, indexing, partitioning, JSONB, query optimization |
| 🐳 | DevOps | Docker, Nginx, GitLab CI/CD, Vercel, Render, Linux basics |
| 📈 | Định hướng | Backend performance, system design, cloud deployment, production readiness |
| 📍 | Địa điểm | Ho Chi Minh City, Vietnam |

---

## 🧰 Công nghệ đang sử dụng

### 🧠 Backend

Mình làm việc chủ yếu với **ASP.NET Core Web API**, xây dựng API theo hướng rõ trách nhiệm, dễ test và dễ mở rộng.

- 🟣 ASP.NET Core, Middleware, Dependency Injection.
- 🟢 Entity Framework Core, LINQ, migration, query optimization.
- 🔐 Authentication bằng JWT.
- 🏗️ Clean Architecture, SOLID, separation of concerns.
- ⚡ Async/await, ThreadPool, concurrency control và backend performance.

**Tài liệu nổi bật:**

- [ASP.NET Core WebAPI](junior/backend/01-aspnet-core-webapi.md)
- [Middleware & Dependency Injection](junior/backend/02-middleware-dependency-injection.md)
- [Entity Framework Core](junior/backend/03-entity-framework-core.md)
- [Clean Architecture & SOLID](junior/backend/06-clean-architecture-solid.md)
- [Performance trong .NET Backend](mid-level/backend/performance-in-backend.md)

### ⚛️ Frontend

Ở phía frontend, mình ưu tiên trải nghiệm rõ ràng, component dễ tái sử dụng và state management vừa đủ với nhu cầu của sản phẩm.

- ⚛️ React fundamentals, hooks và component composition.
- 🧭 React Router cho navigation.
- 🧩 Context API cho state dùng chung.
- 🚀 Tối ưu render và performance frontend.
- ▲ Next.js cho các dự án cần routing, SSR hoặc cấu trúc frontend hiện đại hơn.

**Tài liệu nổi bật:**

- [React Basic](junior/frontend/01-react-basic.md)
- [React Hooks](junior/frontend/02-react-hooks.md)
- [Context API](junior/frontend/03-context-api.md)
- [React Router](junior/frontend/05-react-router.md)
- [Performance Optimization](junior/frontend/06-performance-optimization.md)

### 🗄️ Database

Với database, mình tập trung vào cách thiết kế và truy vấn sao cho hệ thống chạy ổn định khi dữ liệu tăng dần.

- 🐘 PostgreSQL basics.
- 🔁 Transaction và isolation levels.
- 📇 Indexing, partitioning và query optimization.
- 🧾 JSONB trong PostgreSQL.
- 🔎 Tư duy đọc execution plan và tránh query gây tải lớn.

**Tài liệu nổi bật:**

- [PostgreSQL Basic](junior/database/01-postgresql-basic.md)
- [Transactions](junior/database/02-transactions.md)
- [Isolation Levels](junior/database/03-isolation-levels.md)
- [Index & Partitioning](junior/database/04-index-partitioning.md)
- [Query Optimization](junior/database/06-query-optimization.md)

### 🐳 DevOps và Deployment

Mình ghi lại các workflow triển khai thường gặp để từ source code có thể đi tới môi trường chạy thật.

- 🐳 Docker image, container, volume và network.
- 🌐 Nginx cho frontend deployment.
- 🦊 GitLab CI/CD.
- ▲ Deploy frontend lên Vercel.
- ☁️ Deploy backend và database lên cloud service.
- 🐧 Linux basics cho môi trường server.

**Tài liệu nổi bật:**

- [Docker Basic](mid-level/devops/docker/01-docker-basic.md)
- [Containers](mid-level/devops/docker/03-containers.md)
- [Volume And Data](mid-level/devops/docker/04-volume-and-data.md)
- [Network](mid-level/devops/docker/05-network.md)
- [GitLab CI/CD Setup](mid-level/devops/labs/03-gitlab-ci-cd-setup.md)

---

## 🔥 Những chủ đề mình đang đào sâu

### 🖥️ System Design

Mình quan tâm đến cách thiết kế hệ thống theo hướng dễ scale, dễ quan sát và dễ vận hành.

- 🧱 Phân rã component.
- 🔄 Luồng dữ liệu giữa client, backend, database và external services.
- ⚖️ Trade-off giữa consistency, availability, latency và cost.
- 📡 Caching, queue, retry, timeout và observability.

Tham khảo:

- [System Design Introduction](mid-level/system-design/01-system-design-introduction.md)
- [System Design Life Cycle](mid-level/system-design/02-system-design-life-cycle.md)
- [Component of System Design](mid-level/system-design/03-component-of-system-design.md)

### 🛡️ Production Readiness

Một hệ thống tốt không chỉ chạy được ở local, mà còn cần chịu được lỗi, dễ debug và dễ triển khai.

- 🪵 Logging rõ ngữ cảnh.
- 📊 Monitoring latency, CPU, RAM, GC, database và external API.
- 💓 Health check, timeout, retry và fallback.
- 🧮 Tối ưu query, connection pool và memory usage.
- 🚢 CI/CD để giảm thao tác thủ công khi release.

---

## 🗺️ Cách đọc tài liệu

Nếu bạn mới bắt đầu, có thể đi theo thứ tự:

1. [Junior Backend](junior/backend/01-aspnet-core-webapi.md)
2. [Junior Frontend](junior/frontend/01-react-basic.md)
3. [Junior Database](junior/database/01-postgresql-basic.md)
4. [Testing & Deployment](junior/Testing%20&%20Deployment/01-unit-test-dotnet.md)
5. [Mid-Level Backend Performance](mid-level/backend/performance-in-backend.md)
6. [System Design](mid-level/system-design/01-system-design-introduction.md)
7. [Docker & DevOps](mid-level/devops/docker/01-docker-basic.md)

---

## 📝 Nguyên tắc ghi chú

- ✅ Viết ngắn gọn, có ví dụ và ưu tiên tình huống thực tế.
- 🧭 Tách rõ khái niệm, vấn đề, cách xử lý và lỗi thường gặp.
- 💡 Không chỉ học syntax, mà hiểu vì sao một kỹ thuật hữu ích trong production.
- 🔁 Cập nhật dần theo trải nghiệm triển khai và debug hệ thống thật.

---

## 🌐 Liên kết

| Kênh | Link |
| --- | --- |
| 🌐 Website | [https://tampv.pro.vn](https://tampv.pro.vn/) |
| 🐙 GitHub | [https://github.com/alphadev14](https://github.com/alphadev14) |
| 💼 LinkedIn | [https://www.linkedin.com/in/tampv](https://www.linkedin.com/in/tampv) |
| 📦 Source code | [digitaldocs](https://github.com/alphadev14/digitaldocs) |

---

## 📜 License

© 2025 P&T Digital. All rights reserved.
