# Migrations & Versioning

## Mục tiêu

Quản lý thay đổi database schema an toàn khi làm việc theo team và deploy production.

## Cần nắm

- Migration file.
- Backward-compatible change.
- Expand and contract pattern.
- Seed data.
- Rollback strategy.
- Data migration.
- Zero-downtime migration.

## Checklist

- Migration được review như code.
- Không đổi/xóa column đang được app version cũ dùng ngay lập tức.
- Data migration lớn được chia batch.
- Có backup hoặc rollback plan.
- Migration production được chạy có kiểm soát.

## Bài thực hành

- Thêm column nullable trước, deploy app ghi dữ liệu, sau đó mới enforce not null.
- Rename column theo expand-contract.
- Viết migration seed lookup data.

