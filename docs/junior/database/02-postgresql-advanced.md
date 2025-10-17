# 02-postgresql-advanced

## Mục lục

1. Index nâng cao (types, functional, partial, multicolumn, maintenance)
2. Partitioning nâng cao (thiết kế, range/list/hash, attach/detach, chuyển dữ liệu)
3. Transaction nâng cao & Locking (advisory lock, deadlock, best practices)
4. CTE (Common Table Expressions) & Recursive CTE
5. Window Functions (OVER, PARTITION BY, ORDER BY, ranking, aggregate window)
6. EXPLAIN / EXPLAIN ANALYZE & Query tuning checklist
7. Maintenance: VACUUM, ANALYZE, REINDEX, pg_repack
8. Tham khảo

---

# 1. Index nâng cao

## 1.1 Các loại index chính

- **B-Tree**: mặc định, tốt cho `=`, `<`, `>`, `BETWEEN`, `ORDER BY`.
- **GIN (Generalized Inverted Index)**: tối ưu cho `jsonb`, `array`, và full-text search. Hỗ trợ `@>`, `?`, `?|`, `?&`.
- **GiST (Generalized Search Tree)**: dùng cho dữ liệu không gian (PostGIS) và tìm kiếm phức tạp.
- **BRIN (Block Range Index)**: chỉ vài KB; hữu ích với bảng rất lớn có dữ liệu tuần tự (time-series).
- **Hash**: chỉ cho `=`; ngày nay ít dùng (B-Tree thường tốt hơn).

## 1.2 Functional index (index trên expression)

Khi query dùng biểu thức, tạo index trên expression để dùng index:

```sql
-- chỉ số trên lower(email) để tìm case-insensitive
CREATE INDEX idx_users_email_lower ON users (lower(email));
-- query sẽ dùng:
SELECT * FROM users WHERE lower(email) = 'abc@example.com';
```

## 1.3 Partial index (index một phần có điều kiện)

Tạo index chỉ trên subset của rows để tiết kiệm không gian và tăng tốc cho queries cụ thể:

```sql
CREATE INDEX idx_users_active ON users (last_login) WHERE is_active = true;
```

## 1.4 Multicolumn index (index nhiều cột)

Chú ý thứ tự cột trong index quyết định query nào có thể dùng index:

```sql
CREATE INDEX idx_orders_customer_date ON orders (customer_id, order_date);
-- dùng tốt cho WHERE customer_id = ? AND order_date >= ?
```

## 1.5 Index cho JSONB

- GIN index cho JSONB:

```sql
CREATE INDEX idx_products_attrs_gin ON products USING GIN (attributes jsonb_path_ops);
-- hoặc default gin
CREATE INDEX idx_products_attrs_gin2 ON products USING GIN (attributes);
```

- Dùng operator `@>` để kiểm tra containment:

```sql
SELECT * FROM products WHERE attributes @> '{"cpu":"i7"}';
```

## 1.6 Unique index & constraints

- `UNIQUE` có thể được áp dụng lên expression/partial index.

```sql
CREATE UNIQUE INDEX ux_users_email_lower ON users (lower(email));
```

## 1.7 Tạo index không đồng bộ (CONCURRENTLY)

Tránh lock trên table lớn:

```sql
CREATE INDEX CONCURRENTLY idx_orders_customer ON orders(customer_id);
-- Xóa index cũng có CONCURRENTLY
DROP INDEX CONCURRENTLY idx_orders_customer;
```

Lưu ý: không được chạy trong transaction block.

## 1.8 Khi nào index không được dùng?

- Query chọn rất nhiều rows (sequential scan rẻ hơn).
- Funciton không sargable (ví dụ `WHERE LOWER(col) LIKE '%abc'`) — leading wildcard.
- Mismatch giữa data distribution và statistics.

## 1.9 Kiểm tra index được sử dụng

Sử dụng `EXPLAIN (ANALYZE, BUFFERS)` để xem planner chọn index hay sequential scan.

---

# 2. Partitioning nâng cao

## 2.1 Khi nào nên partition?

- Bảng cực lớn (hàng triệu → hàng trăm triệu rows).
- Dữ liệu có key phân vùng hợp lý (time-based, region, tenant).
- Muốn nhanh chóng xóa data bằng cách DROP PARTITION thay vì DELETE.

## 2.2 Các chiến lược partition

- **Range**: time-series (order_date).
- **List**: categorical values (region, country).
- **Hash**: chia đều load khi không có natural range.

## 2.3 Tạo bảng partitioned (Range example)

```sql
CREATE TABLE orders (
  id BIGSERIAL PRIMARY KEY,
  customer_id BIGINT,
  order_date DATE NOT NULL,
  amount NUMERIC(12,2)
) PARTITION BY RANGE (order_date);

CREATE TABLE orders_2023 PARTITION OF orders
  FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');

CREATE TABLE orders_2024 PARTITION OF orders
  FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');
```

- Lưu ý: bảng `orders` (parent) không chứa dữ liệu thực tế (để tránh duplicate rows) — data nằm trong partitions.

## 2.4 Hash partition example

```sql
CREATE TABLE logs (
  id BIGSERIAL PRIMARY KEY,
  event_time TIMESTAMPTZ,
  payload JSONB
) PARTITION BY HASH (id);

CREATE TABLE logs_p0 PARTITION OF logs FOR VALUES WITH (MODULUS 4, REMAINDER 0);
CREATE TABLE logs_p1 PARTITION OF logs FOR VALUES WITH (MODULUS 4, REMAINDER 1);
CREATE TABLE logs_p2 PARTITION OF logs FOR VALUES WITH (MODULUS 4, REMAINDER 2);
CREATE TABLE logs_p3 PARTITION OF logs FOR VALUES WITH (MODULUS 4, REMAINDER 3);
```

## 2.5 Chuyển một bảng hiện có sang partitioned

Quy trình an toàn (một trong các cách):

1. Tạo bảng mới partitioned (orders_new) với cấu trúc giống table cũ.
2. Tạo partitions (range/list/hash) tương ứng.
3. Copy dữ liệu theo từng partition để tránh long-running transaction:

```sql
INSERT INTO orders_new PARTITION FOR VALUES FROM ('2023-01-01') TO ('2024-01-01')
SELECT * FROM orders WHERE order_date >= '2023-01-01' AND order_date < '2024-01-01';
```

4. Khi đã copy xong mọi partition, kiểm tra checksum / counts.
5. Đổi tên tables (rename) hoặc drop table cũ và rename new → production.
6. Tạo index trên từng partition (không có global index cho tất cả partitions).

**Lưu ý:** từ PostgreSQL 11+ có `ATTACH PARTITION` để gắn bảng con đã có dữ liệu, nhưng cần đảm bảo phân vùng key phù hợp cho dữ liệu hiện có.

## 2.6 Partition pruning & performance

- Planner sẽ **prune** các partitions không cần thiết dựa trên WHERE clause → giảm scan.
- Đảm bảo WHERE có predicate trên partition key để planner prune.
- Ví dụ: `WHERE order_date >= '2024-01-01'` → chỉ scan partitions chứa 2024.

## 2.7 Index & Partition

- Index được tạo trên partition parent có thể propagate sang child (PG >= 12). Tuy nhiên, mỗi partition có index vật lý riêng.
- Reindex/maintenance cần chạy trên partitions hoặc dùng script để loop qua partitions.

---

# 3. Transaction nâng cao & Locking

## 3.1 Lock types

- **Row-level locks**: `SELECT ... FOR UPDATE`, `FOR SHARE`.
- **Table-level locks**: `LOCK TABLE ... IN EXCLUSIVE MODE`.
- **Advisory locks**: app-level locking via `pg_advisory_lock()` / `pg_try_advisory_lock()`.

## 3.2 Deadlock và detection

- Deadlock xảy ra khi transaction A giữ lock X cần Y, transaction B giữ Y cần X.
- PostgreSQL có deadlock detector và sẽ abort 1 transaction để break.
- Best practice: acquire locks in consistent order, keep transactions short.

## 3.3 Advisory lock example

Use application-level lock for coarse operations:

```sql
-- block until get lock
SELECT pg_advisory_lock(12345);

-- try lock (non-blocking)
SELECT pg_try_advisory_lock(12345);

-- release
SELECT pg_advisory_unlock(12345);
```

## 3.4 Transaction snapshot & Serializable

- **Serializable** isolation uses Serializable Snapshot Isolation (SSI) in PG to prevent anomalies.
- In high-contention work, Serializable may cause serialization failures that require retry logic in application.

## 3.5 Long running transaction issues

- Keeps old row versions (MVCC) alive → increased disk usage and bloat.
- Vacuum can't reclaim tuples until no transaction needs old snapshot.
- Avoid long-running idle transactions (e.g., open psql and forget).

---

# 4. CTE (Common Table Expressions)

## 4.1 Simple CTE (WITH)

- CTE giúp tách query phức tạp thành các phần logic dễ đọc:

```sql
WITH active_users AS (
  SELECT id FROM users WHERE is_active = true
)
SELECT u.id, count(o.*) AS orders
FROM active_users au
JOIN users u ON u.id = au.id
LEFT JOIN orders o ON o.user_id = u.id
GROUP BY u.id;
```

## 4.2 Materialization vs Inline (Postgres behavior)

- CTE trong PG trước v12 là **materialized** (thực hiện và cache kết quả).
- Từ PG12+, CTE có thể inline giống subquery nếu optimizer thấy tốt (tránh unnecessary materialization) — nhưng có thể dùng `MATERIALIZED` / `NOT MATERIALIZED` để force behavior:

```sql
WITH t AS MATERIALIZED (SELECT ...)
WITH t AS NOT MATERIALIZED (SELECT ...)
```

## 4.3 Recursive CTE (đệ quy)

Dùng để traverse hierarchical data (parent-child):

```sql
WITH RECURSIVE subordinates AS (
  SELECT id, manager_id, name FROM employees WHERE id = 1
  UNION ALL
  SELECT e.id, e.manager_id, e.name
  FROM employees e
  JOIN subordinates s ON e.manager_id = s.id
)
SELECT * FROM subordinates;
```

---

# 5. Window Functions

## 5.1 Tổng quan

Window functions chạy trên "window" của rows tương ứng và không gộp các rows lại (khác GROUP BY).  
Cú pháp chung: `function(...) OVER (PARTITION BY ... ORDER BY ... ROWS BETWEEN ...)`

## 5.2 Common window functions

- `ROW_NUMBER()` - index từng row trong partition.
- `RANK()` / `DENSE_RANK()` - ranking with gaps or dense.
- `LAG(col, offset)` / `LEAD(col, offset)` - truy xuất giá trị trước/sau.
- Aggregate window: `SUM(... ) OVER (...)`, `AVG(...) OVER (...)`.

## 5.3 Ví dụ

Top customer by month:

```sql
SELECT
  customer_id,
  order_month,
  total,
  RANK() OVER (PARTITION BY order_month ORDER BY total DESC) AS rank_in_month
FROM monthly_totals
ORDER BY order_month, rank_in_month;
```

Running total:

```sql
SELECT
  id, order_date, amount,
  SUM(amount) OVER (ORDER BY order_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM orders
ORDER BY order_date;
```

## 5.4 Performance note

Window functions may require sorting; consider appropriate indexes on partition/order columns.

---

# 6. EXPLAIN / EXPLAIN ANALYZE & Query tuning checklist

## 6.1 EXPLAIN usage

```sql
EXPLAIN SELECT * FROM orders WHERE order_date >= '2024-01-01';
EXPLAIN ANALYZE SELECT * FROM orders WHERE order_date >= '2024-01-01';
```

- `EXPLAIN` shows planned steps; `ANALYZE` runs query and shows actual time and rows. Use `BUFFERS` for buffer usage.

## 6.2 Common things to check

- Is planner using index or seq scan?
- Cost estimates vs actual rows — statistics stale? Run `ANALYZE`.
- Is there an expensive sort? Add index supporting ORDER BY.
- Are joins done via nested loops vs hash join? For large joins, hash join may be better but requires memory.
- Check `pg_stat_statements` to find hot queries.

## 6.3 Example: optimize slow query

- Add missing index (covering index).
- Rewrite subqueries into joins or CTEs when appropriate.
- Avoid SELECT \*; only request necessary columns.
- Use LIMIT early when possible.

---

# 7. Maintenance: VACUUM, ANALYZE, REINDEX, pg_repack

## 7.1 VACUUM

- `VACUUM` reclaims space from dead tuples (MVCC).
- `VACUUM FULL` reclaims space but requires exclusive lock; slow.
- Autovacuum runs automatically; tune autovacuum settings for large tables.

## 7.2 ANALYZE

- Collect statistics for planner; run manually after large data changes:

```sql
ANALYZE orders;
```

## 7.3 REINDEX

- Rebuild corrupted or bloated indexes:

```sql
REINDEX TABLE orders;
```

## 7.4 pg_repack

- Extension to rebuild table/index with minimal locks (good for production maintenance).

---

# 8. Tham khảo

- PostgreSQL official docs: https://www.postgresql.org/docs/
- Index types: https://www.postgresql.org/docs/current/indexes-types.html
- Partitioning: https://www.postgresql.org/docs/current/ddl-partitioning.html
- EXPLAIN: https://www.postgresql.org/docs/current/using-explain.html

---

_Ghi chú:_ Bạn có thể copy file này vào `docs/junior/database/02-postgresql-advanced.md` trong repo MkDocs của bạn.
