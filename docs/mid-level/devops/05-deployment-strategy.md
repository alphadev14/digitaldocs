# Deployment Strategy

## Mục tiêu

Deploy ứng dụng an toàn hơn, có rollback và giảm downtime.

## Cần nắm

- Rolling deployment.
- Blue-green deployment.
- Canary release.
- Rollback.
- Database migration khi deploy.
- Feature flag.
- Smoke test.

## Checklist

- Mỗi release có version rõ ràng.
- Có rollback plan.
- Migration tương thích với app version cũ/mới.
- Có smoke test sau deploy.
- Feature lớn có thể bật/tắt bằng flag nếu cần.

## Bài thực hành

- Thiết kế flow deploy API có migration.
- Viết checklist smoke test sau deploy.
- Mô phỏng rollback khi release lỗi.

