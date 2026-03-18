# 🧱 Kiến thức cơ bản về docker

## 🎯 Mục tiêu

- Hiểu Docker là gì
- Hiểu tại sao cần Docker
- Nắm các khái niệm: Image, Container, Volume, Network, Registry
- Hiểu bản chất: **Container = Process (không phải VM)**

---

# 🚀 1. Docker là gì?

Docker là nền tảng giúp bạn **đóng gói ứng dụng + môi trường chạy** vào một đơn vị gọi là **container**.

👉 Điều này đảm bảo:

> "Chạy ở máy dev → chạy y chang trên server"

---

## 💥 Vấn đề trước Docker

- Máy dev: Node 18, .NET 8
- Server: Node 16, thiếu thư viện

👉 Kết quả:

- Lỗi môi trường
- Khó deploy

---

## ✅ Docker giải quyết

Docker đóng gói:

- Source code
- Runtime
- Dependencies

👉 Ứng dụng chạy **consistent ở mọi môi trường**

---

# ⚔️ 2. Docker vs Virtual Machine

| Tiêu chí    | Docker               | VM                 |
| ----------- | -------------------- | ------------------ |
| OS          | Dùng chung kernel    | Mỗi VM có OS riêng |
| Size        | Nhẹ (MB)             | Nặng (GB)          |
| Start       | Nhanh (seconds)      | Chậm (minutes)     |
| Performance | Gần native           | Chậm hơn           |
| Use case    | Microservices, CI/CD | Chạy OS khác       |

---

## 🔥 Kiến trúc

### VM:

```
App
OS (Ubuntu)
Hypervisor
Hardware
```

### Docker:

```
App
Docker Engine
Host OS
Hardware
```

👉 Docker **không tạo OS mới**

---

# 🧠 3. Các khái niệm cốt lõi

---

## 📦 3.1 Image

- Là template (read-only)
- Dùng để tạo container

```bash
docker pull ubuntu
```

---

## 📦 3.2 Container

- Là instance của image

```bash
docker run ubuntu
```

👉 Flow:

```
Image → Container → Running App
```

---

## 🏪 3.3 Registry (Docker Hub)

- Nơi lưu trữ image
- Giống npm / nuget

```bash
docker pull nginx
```

---

## 💾 3.4 Volume

- Lưu dữ liệu ngoài container
- Giữ data khi container bị xóa

### ❌ Sai

```bash
docker run postgres
```

### ✅ Đúng

```bash
docker run -v mydata:/var/lib/postgresql/data postgres
```

---

## 🌐 3.5 Network

- Cho phép container giao tiếp với nhau

Ví dụ:

- Backend → Database

```
postgres:5432
```

---

# ⚡ 4. Thực hành

---

## ✅ Kiểm tra Docker

```bash
docker --version
```

---

## ✅ Hello World

```bash
docker run hello-world
```

👉 Docker sẽ:

1. Pull image nếu chưa có
2. Tạo container
3. Chạy và in kết quả

---

## ✅ Chạy Ubuntu container

```bash
docker run -it ubuntu bash
```

### Giải thích:

- `-it`: interactive terminal
- `ubuntu`: image
- `bash`: command

👉 Kết quả:

```
root@container:/#
```

---

# 🔥 5. Container = Process

Docker sử dụng:

- Linux namespaces
- cgroups

👉 Để tạo:

- Isolation
- Resource control

---

## So sánh

|                | Container | Process thường |
| -------------- | --------- | -------------- |
| Isolation      | Cao       | Thấp           |
| Resource limit | Có        | Ít             |
| Portable       | Có        | Không          |

---

👉 Kết luận:

> Container = Process + Isolation + Packaging

---

# 🚀 6. Mapping vào project thực tế

Ví dụ hệ thống:

- .NET Backend
- React Frontend
- PostgreSQL
- Redis

👉 Docker hóa:

| Service  | Container |
| -------- | --------- |
| Backend  | 1         |
| Frontend | 1         |
| Database | 1         |
| Redis    | 1         |

---

**Author:** Alpha DevOp
