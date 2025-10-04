# Index & Partitioning trong PostgreSQL

## 1. Index

- **Index** là cấu trúc dữ liệu đặc biệt (thường là `B-Tree`) giúp tăng tốc độ truy vấn trong cơ sở dữ liệu.
- Hoạt động giống như mục lục trong sách: thay vì đọc toàn bộ sách, DB có thể tra nhanh vị trí của dữ liệu.

#### 1.1. Cách hoạt động

- Khi bạn chạy query với `WHERE`, `JOIN`, `ORDER BY`, PostgreSQL sẽ kiểm tra có Index phù hợp hay không.
- Nếu có Index -> DB truy vấn nhanh hơn nhiều (do không phải scan toàn bộ bảng).

#### 1.2 Ví dụ tạo Index

```sql
-- Tạo index trên cột email
CREATE INDEX idx_user_email ON users(email);

-- Xóa index
DROP INDEX idx_user_email;

```

#### 1.3 Các loại Index trong PostgreSQL

###### a. B-Tree (mặc định):

`Dùng cho so sánh =, <, >, BETWEEN`.

###### b. Hash Index:

`tối ưu cho so sánh = (ít dùng, vì B-Tree cũng làm được)`.

###### c. GIN (Generalized Inverted Index):

`tối ưu cho dữ liệu dạng mảng, JSONB, full-text search`.

###### d. GiST (Generalized Search Tree):

`cho dữ liệu không gian (PostGIS, tìm kiếm khoảng cách)`.

###### d. BRIN (Block Range Index):

`tiết kiệm bộ nhớ, phù hợp với bảng rất lớn (dữ liệu tuần tự, ví dụ cột timestamp).`.

#### 3. Cách sử dụng

**📌 Khi nào nên dùng Index**

- Truy vấn `SELECT` thường xuyên trên một cột.

- `JOIN` giữa các bảng.

- `WHERE`, `ORDER BY`, `GROUP BY` trên cột đó.

**❌ Không nên tạo quá nhiều Index vì**

- Làm chậm INSERT, UPDATE, DELETE (do phải cập nhật index).
- Tốn dung lượng lưu trữ.

## 2. Partition Table (Phân vùng bảng)

- **Partitioning** là chia bảng lớn thành nhiều **bảng con (partition)** để dễ quản lý và tăng hiệu suất query.

- Người dùng query bảng cha, PostgreSQL sẽ tự động tìm partition phù hợp.

**📌 Các loại Partitioning trong PostgreSQL**

- **Range Partitioning**: chia theo khoảng giá trị.

```sql
CREATE TABLE sales (
  id SERIAL,
  sale_date DATE NOT NULL,
  amount NUMERIC
) PARTITION BY RANGE (sale_date);

CREATE TABLE sales_2023 PARTITION OF sales
  FOR VALUES FROM ('2023-01-01') TO ('2023-12-31');

CREATE TABLE sales_2024 PARTITION OF sales
  FOR VALUES FROM ('2024-01-01') TO ('2024-12-31');
```

- **List Partitioning**: chia theo danh sách giá trị cụ thể.

```sql
CREATE TABLE users (
  id SERIAL,
  region TEXT NOT NULL
) PARTITION BY LIST(region);

CREATE TABLE users_vn PARTITION OF users FOR VALUES IN ('VN');
CREATE TABLE users_us PARTITION OF users FOR VALUES IN ('US');
```

- **Hash Partitioning**: PostgreSQL tự hash giá trị cột để chia theo điều kiện.

```sql
CREATE TABLE orders (
  id SERIAL,
  customer_id INT
) PARTITION BY HASH (customer_id);

CREATE TABLE orders_p1 PARTITION OF orders FOR VALUES WITH (MODULUS 4, REMAINDER 0);
CREATE TABLE orders_p2 PARTITION OF orders FOR VALUES WITH (MODULUS 4, REMAINDER 1);
CREATE TABLE orders_p3 PARTITION OF orders FOR VALUES WITH (MODULUS 4, REMAINDER 2);
CREATE TABLE orders_p4 PARTITION OF orders FOR VALUES WITH (MODULUS 4, REMAINDER 3);
```

`FOR VALUES WITH (MODULUS 4, REMAINDER 0)`

- `MODULUS 4`: Tổng số partition con được tạo ra là 4 (tức bảng orders sẽ được chia thành 4 bảng con: orders_p1, orders_p2, orders_p3, orders_p4).
- `REMAINDER 0`: giá trị hash mod 4 = 0, sẽ được lưu vào bảng orders_p1. - Tức là PostgreSQL sẽ lấy giá trị cột dùng để partition (ví dụ: customer_id) - Tính hash(customer_id) % 4 - Nếu kết quả = 0 -> dữ liệu lưu vào bảng orders_p1.

## 3. So sánh **_Index_** và **_PARTITION_**

| Đặc điểm                | Index                    | Partition                               |
| ----------------------- | ------------------------ | --------------------------------------- |
| Tăng tốc độ truy vấn    | ✅ Có                    | ✅ Có (khi lọc theo partition key)      |
| Giảm dung lượng lưu trữ | ❌ Không                 | ✅ Có thể (nếu dữ liệu chia nhỏ)        |
| Quản lý dữ liệu lớn     | ❌ Không                 | ✅ Rất tốt (tránh full scan bảng lớn)   |
| Phù hợp khi             | truy vấn nhiều, bảng vừa | Bảng rất lớn (hàng trăm triệu bảng ghi) |

## 4. Best Practices

- Với bảng nhỏ, trung bình -> chỉ cần **Index**.
- Với bảng lớn (log, giao dịch theo thời gian) -> dùng **Partition** + **Index** trong từng partition.
- Dùng **EXPLAIN ANALYZE** để kiểm tra query có dùng Index/Partition không.
