# Database Performance

## Mục tiêu

Theo dõi và cải thiện hiệu năng database dựa trên metric, slow query và growth pattern.

## Cần nắm

- Slow query log.
- Connection pool.
- Query latency P95/P99.
- Table growth.
- Vacuum/analyze trong PostgreSQL.
- Partitioning.
- Read/write pattern.

## Checklist

- Có monitoring slow query.
- API không mở quá nhiều connection.
- Query list có pagination.
- Table lớn có chiến lược archive/partition nếu cần.
- Có metric database CPU, memory, disk, connection.

## Bài thực hành

- Tìm top 5 query chậm nhất.
- Kiểm tra connection pool khi load test.
- Thiết kế partition cho bảng log/event lớn.

