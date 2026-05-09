# Docker Compose & Environment

## Mục tiêu

Chạy local/staging stack nhất quán bằng Docker Compose và quản lý environment rõ ràng.

## Cần nắm

- `docker-compose.yml`.
- Service, network, volume.
- `.env`.
- Health check.
- Override file cho local/staging.
- Database container.

## Checklist

- Mỗi service có name rõ ràng.
- Config qua environment variable.
- Không commit secret thật.
- Volume database được đặt tên.
- Service phụ thuộc có health check.

## Bài thực hành

- Tạo compose gồm API, PostgreSQL, Redis.
- Thêm health check cho database.
- Tách `.env.example` và `.env.local`.

