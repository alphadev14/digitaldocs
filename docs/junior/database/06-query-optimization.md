# Query Optimization in PostgreSQL

Tối ưu hóa truy vấn là bước quan trọng giúp hệ thống đạt hiệu năng cao, đặc biệt khi dữ liệu lớn. PostgreSQL cung cấp nhiều công cụ và kỹ thuật để giúp chúng ra hiểu và cải thiện hiệu suất truy vấn.

Chạy nhanh hơn bằng cách:

- Giảm số lượng dữ liệu phải quét (scan).
- Chọn index và kế hoạch thực thi (execution plan) tối ưu nhất.
- Giảm thiểu JOIN, sort, hoặc I/O không cần thiết.

Công cụ mạnh nhất để phân tích hiệu năng trong PostgreSQL chính là **EXPLAIN** và **EXPLAIN ANALYZE**.

---

## 1. Phân tích truy vấn bằng EXPLAIN

### 🧩 Lệnh cơ bản

```sql
EXPLAIN SELECT * FROM users WHERE age > 30;
```

- PostgreSQL sẽ hiển thị **kế hoạch thực thi (execution plan)** — cách hệ thống dự định chạy truy vấn.
- Để xem chi tiết hơn, thêm `ANALYZE`:

```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE age > 30;
```

➡️ PostgreSQL sẽ **thực thi truy vấn thật** và hiển thị thời gian thực thi, số dòng đọc, index nào được dùng.

### 📊 Các thành phần trong EXPLAIN

| Thành phần       | Ý nghĩa                                                                        |
| ---------------- | ------------------------------------------------------------------------------ |
| Seq Scan         | Quét tuần tự toàn bộ bảng. (Chậm với bảng lớn)                                 |
| Index Scan       | Quét dữ liệu qua index (Nhanh hơn nhiều)                                       |
| Bitmap Heap Scan | Dùng khi truy vấn nhiều dòng qua index                                         |
| Nested Loop      | Lặp qua từng dòng của bảng A để JOIN bảng B                                    |
| Hash Join        | Dựng hash table từ 1 bảng rồi JOIN bảng còn lại (hiệu quả với tập dữ liệu lớn) |
| Merge Join       | JOIN hai bảng đã được sắp xếp theo cùng key                                    |

---

## 2. Tối ưu JOIN

### 💡 Các loại JOIN phổ biến

- **Nested Loop Join** → tốt cho bảng nhỏ JOIN với bảng có index.
- **Hash Join** → tốt cho tập dữ liệu lớn không sắp xếp.
- **Merge Join** → tốt cho dữ liệu đã ORDER BY hoặc có index phù hợp.

### ⚙️ Mẹo tối ưu JOIN

1. Đảm bảo **cột JOIN có index** (đặc biệt khi JOIN theo khóa ngoại).
2. Giới hạn dữ liệu trước khi JOIN (`WHERE`, `LIMIT`, `SELECT cụ thể`).
3. Tránh JOIN nhiều bảng không cần thiết.
4. Sử dụng **EXPLAIN ANALYZE** để xem kế hoạch thực tế.

---

## 3. Index Tuning

### 🎯 Khi nào nên tạo index

- Cột dùng trong `WHERE`, `JOIN`, `ORDER BY`, `GROUP BY`.
- Không nên index quá nhiều (tốn RAM và thời gian ghi dữ liệu).

### 🧱 Các loại index

| Loại index                           | Mô tả                             | Phù hợp với                     |
| ------------------------------------ | --------------------------------- | ------------------------------- |
| **B-Tree**                           | Mặc định, tốt cho so sánh =, <, > | Most common queries             |
| **Hash**                             | Chỉ cho so sánh "="               | Truy vấn key-value              |
| **GIN (Generalized Inverted Index)** | Tìm kiếm text, JSONB, array       | Full-text search, JSONB         |
| **GiST**                             | Không gian, địa lý, vector        | Dữ liệu hình học, vector search |

### ⚡ Tips

```sql
CREATE INDEX idx_user_email ON users(email);
CREATE INDEX idx_orders_userid_date ON orders(user_id, order_date);
```

- Index đa cột (multi-column index) chỉ có hiệu quả **theo thứ tự từ trái qua phải**.

---

## 4. Cấu trúc câu lệnh truy vấn

### 🚀 Best practices

1. **SELECT chỉ cột cần thiết** → giảm IO.
2. **Tránh SELECT \*** → gây chậm khi bảng lớn.
3. **Dùng WHERE trước khi JOIN** để lọc sớm dữ liệu.
4. **Sử dụng LIMIT** khi chỉ cần một phần dữ liệu.
5. **Cân nhắc subquery vs CTE (WITH)** — CTE dễ đọc hơn, nhưng có thể bị vật hóa (materialize) khiến chậm hơn.

### Ví dụ

```sql
-- Không tốt
SELECT * FROM orders o
JOIN users u ON o.user_id = u.id;

-- Tốt hơn
SELECT o.id, o.total, u.name
FROM orders o
JOIN users u ON o.user_id = u.id
WHERE o.status = 'completed';
```

---

## 5. Tối ưu ORDER BY, GROUP BY

### ORDER BY

- Tận dụng index nếu sắp xếp cùng chiều với index.
- Tránh ORDER BY nhiều cột khi không có index tương ứng.

### GROUP BY

- Nếu thường xuyên GROUP BY trên cột nào, hãy tạo index hoặc materialized view.

---

## 6. Vacuum & Analyze

- **VACUUM** → dọn rác, giải phóng không gian chết.
- **ANALYZE** → cập nhật thống kê cho optimizer.

```sql
VACUUM ANALYZE;
```

➡️ Giúp PostgreSQL lên kế hoạch truy vấn hiệu quả hơn.

---

## 7. Tổng hợp quy trình tối ưu

1. Dùng `EXPLAIN ANALYZE` để hiểu cách truy vấn chạy.
2. Kiểm tra **index** có được dùng không.
3. Giảm kích thước dữ liệu đầu vào (WHERE, LIMIT).
4. Chọn **JOIN phù hợp**.
5. Cấu trúc câu lệnh logic, dễ đọc.
6. Thường xuyên `VACUUM` và `ANALYZE`.

---

## 📘 Kết luận

Tối ưu hóa truy vấn không phải chỉ là thêm index, mà là **quy trình tổng thể** gồm hiểu dữ liệu, phân tích kế hoạch thực thi, và viết truy vấn hợp lý.  
Khi bạn hiểu sâu `EXPLAIN`, `JOIN`, và `INDEX`, bạn đã nắm được cốt lõi của hiệu năng trong PostgreSQL.
