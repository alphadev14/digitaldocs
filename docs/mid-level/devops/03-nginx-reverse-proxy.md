# Nginx Reverse Proxy

## Mục tiêu

Dùng Nginx để route request, phục vụ static file và làm reverse proxy cho backend.

## Cần nắm

- Server block.
- Reverse proxy.
- Static file hosting.
- SSL/TLS.
- Gzip/Brotli.
- Cache header.
- WebSocket proxy.

## Checklist

- API được proxy đúng path.
- Static frontend có cache header phù hợp.
- HTTPS được bật ở production.
- Timeout proxy đủ hợp lý.
- Log access/error được lưu.

## Bài thực hành

- Serve React build bằng Nginx.
- Proxy `/api` về ASP.NET Core API.
- Thêm SSL bằng Let's Encrypt nếu deploy VPS.

