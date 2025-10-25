# PostgreSQL Advanced

## 1. Stored Procedures & Functions

### 🔹 1.1 Function

- Dùng để **trả về giá trị** (RETURN).
- Có thể viết bằng SQL hoặc PL/pgSQL.
- **Ví dụ:**

```sql
CREATE OR REPLACE FUNCTION get_user_count()
RETURNS INTEGER AS $$
BEGIN
  RETURN (SELECT COUNT(*) FROM users);
END;
$$ LANGUAGE plpgsql;
```

### 🔹 1.2 Stored Procedure

- Không trả về giá trị, thường dùng để **xử lý logic nghiệp vụ hoặc cập nhật nhiều bảng**.
- Gọi bằng `CALL`, không dùng `SELECT`.
- **Ví dụ:**

```sql
CREATE OR REPLACE PROCEDURE update_order_status(order_id INT, new_status TEXT)
LANGUAGE plpgsql AS $$
BEGIN
  UPDATE orders SET status = new_status WHERE id = order_id;
END;
$$;
```

---

## 2. Triggers

- Tự động thực thi **khi có INSERT, UPDATE, DELETE**.
- Dùng để **ghi log, kiểm tra dữ liệu, đồng bộ bảng khác**.

**Ví dụ:** Log khi thêm user mới:

```sql
CREATE TABLE user_logs (
  id SERIAL PRIMARY KEY,
  user_id INT,
  action TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE OR REPLACE FUNCTION log_user_insert()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO user_logs(user_id, action)
  VALUES (NEW.id, 'User created');
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_user_insert
AFTER INSERT ON users
FOR EACH ROW
EXECUTE FUNCTION log_user_insert();
```

---

## 3. Views & Materialized Views

### 🔹 3.1 View

- View là **bảng ảo** dựa trên kết quả của truy vấn.
- Giúp **tái sử dụng truy vấn phức tạp**.

```sql
CREATE VIEW active_users AS
SELECT id, name, email
FROM users
WHERE is_active = TRUE;
```

### 🔹 3.2 Materialized View

- Lưu kết quả truy vấn **vật lý trên đĩa**, cần **refresh** khi dữ liệu thay đổi.

```sql
CREATE MATERIALIZED VIEW sales_summary AS
SELECT customer_id, SUM(amount) AS total_sales
FROM orders
GROUP BY customer_id;

REFRESH MATERIALIZED VIEW sales_summary;
```

---

## 4. Common Table Expressions (CTE)

- Giúp **chia nhỏ truy vấn phức tạp**.
- Hỗ trợ **truy vấn đệ quy (recursive)**.

**Ví dụ – CTE cơ bản:**

```sql
WITH high_value_orders AS (
  SELECT * FROM orders WHERE amount > 1000
)
SELECT * FROM high_value_orders WHERE status = 'completed';
```

**Ví dụ – CTE đệ quy (recursive):**

```sql
WITH RECURSIVE subordinates AS (
  SELECT id, manager_id, name
  FROM employees
  WHERE manager_id IS NULL

  UNION ALL

  SELECT e.id, e.manager_id, e.name
  FROM employees e
  INNER JOIN subordinates s ON e.manager_id = s.id
)
SELECT * FROM subordinates;
```

---

## 5. Performance Tuning nâng cao

### 🔹 5.1 Connection Pooling

- Dùng công cụ như **pgBouncer** để quản lý kết nối.
- Giúp giảm tải khi có nhiều truy vấn đồng thời.

### 🔹 5.2 Query Plan & Cost

- Dùng `EXPLAIN (ANALYZE, BUFFERS)` để xem chi phí thực thi.
- Tập trung tối ưu các phần có **Seq Scan** hoặc **Nested Loop**.

### 🔹 5.3 Partitioning nâng cao

- Dùng **hash partition** hoặc **range partition** để chia dữ liệu lớn.
- Giúp tăng tốc truy vấn và dễ dàng quản lý dữ liệu.

```sql
CREATE TABLE orders_2025 PARTITION OF orders
FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');
```

### 🔹 5.4 VACUUM & ANALYZE

- **VACUUM**: dọn rác dữ liệu sau khi xóa.
- **ANALYZE**: cập nhật thống kê để query planner tối ưu.

```sql
VACUUM (VERBOSE, ANALYZE);
```

### 🔹 5.5 Parallel Query

- PostgreSQL hỗ trợ chạy song song trên CPU nhiều lõi.
- Tối ưu cho truy vấn lớn.

```sql
SET max_parallel_workers_per_gather = 4;
```

---

## 6. Extensions hữu ích

| Extension            | Mục đích                               |
| -------------------- | -------------------------------------- |
| `pg_stat_statements` | Theo dõi và phân tích truy vấn         |
| `uuid-ossp`          | Sinh UUID                              |
| `pg_trgm`            | Tìm kiếm chuỗi gần đúng (fuzzy search) |
| `citext`             | Kiểu text không phân biệt hoa thường   |
| `timescaledb`        | Lưu dữ liệu dạng time-series           |

---

## 7. Tổng kết

| Chủ đề                            | Kỹ năng chính                        |
| --------------------------------- | ------------------------------------ |
| **Stored Procedures / Functions** | Viết logic nghiệp vụ trong DB        |
| **Triggers**                      | Tự động xử lý khi dữ liệu thay đổi   |
| **Views / Materialized Views**    | Tối ưu truy vấn và tái sử dụng       |
| **CTE / Recursive Query**         | Làm việc với dữ liệu phức tạp        |
| **Performance Tuning**            | Tối ưu truy vấn và quản lý hiệu năng |
| **Extensions**                    | Mở rộng khả năng PostgreSQL          |

---

✅ **Mục tiêu sau phần này:**

- Hiểu và triển khai logic nghiệp vụ trong DB.
- Biết tối ưu hiệu năng và cấu trúc hệ thống DB lớn.
- Làm việc tự tin với các truy vấn phức tạp và công cụ phân tích hiệu năng.
