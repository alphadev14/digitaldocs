# Indexing & Query Plan

## Mục tiêu

Biết đọc query plan và tạo index dựa trên query thực tế, không tạo index theo cảm giác.

## Cần nắm

- B-tree index.
- Composite index.
- Selectivity.
- Covering index.
- `EXPLAIN` và `EXPLAIN ANALYZE`.
- Sequential scan vs index scan.
- Cost của việc có quá nhiều index.

## Checklist

- Query chậm được đo bằng execution plan.
- Composite index đúng thứ tự column.
- Index phục vụ filter, join, sort phổ biến.
- Không tạo index cho column ít selectivity nếu không có lý do.
- Cân nhắc write overhead khi thêm index.

## Bài thực hành

- Tạo query search order theo customer, status, created date.
- Chạy `EXPLAIN ANALYZE` trước và sau khi thêm index.
- Thử đổi thứ tự composite index và so sánh.

