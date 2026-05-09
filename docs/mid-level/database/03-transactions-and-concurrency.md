# Transactions & Concurrency

## Mục tiêu

Hiểu transaction, isolation level và các lỗi concurrency thường gặp trong hệ thống thật.

## Cần nắm

- ACID.
- Read committed, repeatable read, serializable.
- Dirty read, non-repeatable read, phantom read.
- Row lock.
- Deadlock.
- Optimistic concurrency.
- Pessimistic locking.

## Checklist

- Transaction càng ngắn càng tốt.
- Không gọi external API bên trong transaction nếu không cần.
- Update critical data có cơ chế chống lost update.
- Có retry hợp lý cho deadlock/transient error.
- Hiểu isolation level đang dùng trong database.

## Bài thực hành

- Mô phỏng hai request cùng update inventory.
- Thêm optimistic concurrency bằng version column.
- Tạo deadlock đơn giản và phân tích nguyên nhân.

