# Giới thiệu về PostgreSQL

## 1. PostgreSQL

- PostgreSQL là một RDBM (Relational Database Management System) mã nguồn mở.
- Hỗ trợ chuẩn SQL, mạnh về tính mở rộng và dữ liệu phức tạp.
- Thường dùng trong:
  - Hệ thống backend (API, web app, microservices)
  - Ứng dụng phân tích dữ liệu (analysts, BI)
  - Hệ thống tài chính, thương mại điện tử.

## 2. Các kiểu dữ liệu

#### 2.1 Kiểu số nguyên (Integer Types)

| Kiểu dữ liệu            | Kích thước (bytes)              | Khoảng giá trị                                           | Mô tả                                |
| ----------------------- | ------------------------------- | -------------------------------------------------------- | ------------------------------------ |
| `SMALLINT` (int2)       | 2 bytes                         | -32,768 đến 32,767                                       | Số nguyên nhỏ                        |
| `INTEGER` (int4, `INT`) | 4 bytes                         | -2,147,483,648 đến 2,147,483,647                         | Số nguyên thường dùng                |
| `BIGINT` (int8)         | 8 bytes                         | -9,223,372,036,854,775,808 đến 9,223,372,036,854,775,807 | Số nguyên lớn                        |
| `SERIAL`                | 4 bytes (int4 + auto increment) | 1 đến 2,147,483,647                                      | Tự tăng, thường dùng làm primary key |
| `BIGSERIAL`             | 8 bytes (int8 + auto increment) | 1 đến 9,223,372,036,854,775,807                          | Tự tăng cho khóa chính lớn           |

👉 Ghi nhớ:

- `SERIAL` và `BIGSERIAL` thực chất chỉ là shortcut = `INTEGER/BIGINT + sequence + DEFAULT`.
- Dùng `BIGINT` cho dữ liệu lớn (log, giao dịch).

#### 2.2 Kiểu số thực (Floating-point Types)

| Kiểu dữ liệu                    | Kích thước (bytes) | Độ chính xác                        | Khoảng giá trị               |
| ------------------------------- | ------------------ | ----------------------------------- | ---------------------------- |
| `REAL` (float4)                 | 4 bytes            | ~6 chữ số thập phân                 | 3.4e-38 đến 1.7e+38          |
| `DOUBLE PRECISION` (float8)     | 8 bytes            | ~15 chữ số thập phân                | 1.7e-308 đến 1.7e+308        |
| `NUMERIC(p, s) / DECIMAL(p, s)` | thay đổi           | Chính xác tuyệt đối (do định nghĩa) | Tùy p (precision), s (scale) |

👉 Ghi nhớ:

- `REAL` và `DOUBLE PRECISION` nhanh hơn nhưng có thể sai số.
- `NUMERIC` (hoặc `DECIMAL`) chậm hơn nhưng chính xác tuyệt đối → thường dùng cho tiền tệ (`NUMERIC(12,2)`).

#### 2.3 Kiểu chuỗi (Character Types)

| Kiểu dữ liệu | Kích thước (bytes)                     | Ghi chú                                      |
| ------------ | -------------------------------------- | -------------------------------------------- |
| `CHAR(n)`    | cố định n byte                         | Nếu chuỗi ngắn hơn thì pad thêm khoảng trắng |
| `VARCHAR(n)` | tối đa n byte                          | Linh hoạt, kiểm soát độ dài                  |
| `TEXT`       | không giới hạn (lên đến 1GB mỗi field) | Phổ biến nhất trong PostgreSQL               |

👉 Trong PostgreSQL, TEXT và VARCHAR hiệu năng gần như giống nhau, thường dùng TEXT trừ khi cần giới hạn độ dài.

#### 2.4 Kiểu logic (Boolean)

| Kiểu dữ liệu | giá trị                 | kích thước |
| ------------ | ----------------------- | ---------- |
| `BOOLEAN`    | `TRUE`, `FALSE`, `NULL` | 1 byte     |

#### 2.5 Kiểu ngày giờ (Date/Time)

| Kiểu dữ liệu                             | Kích thước (bytes) | Độ chính xác                | Khoảng giá trị        |
| ---------------------------------------- | ------------------ | --------------------------- | --------------------- |
| `DATE`                                   | 4 bytes            | Ngày (không có giờ)         | 4713 TCN – 587489 SCN |
| `TIME [(p)]`                             | 8 bytes            | Giờ (chính xác microsecond) | 00:00:00 – 24:00:00   |
| `TIMESTAMP [(p)]`                        | 8 bytes            | Ngày + Giờ                  | 4713 TCN – 294276 SCN |
| `TIMESTAMP WITH TIME ZONE (timestamptz)` | 8 bytes            | Có timezone                 | 4713 TCN – 294276 SCN |
| `INTERVAL`                               | 16 bytes           | Khoảng thời gian            | ±178,000,000 năm      |

👉 Ví dụ:

```sql
SELECT NOW(); -- thời gian hiện tại
SELECT CURRENT_DATE;
SELECT NOW() + INTERVAL '7 days'; -- cộng thêm 7 ngày
```

#### 2.6 JSON && JSONB

| Kiểu dữ liệu | Kích thước | Mô tả                                           |
| ------------ | ---------- | ----------------------------------------------- |
| `JSON`       | thay đổi   | Lưu raw JSON, chậm hơn                          |
| `JSONB`      | thay đổi   | Lưu JSON nhị phân (binary), nhanh hơn khi query |

👉 Ví dụ:

```sql
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  data JSONB
);

INSERT INTO products(data)
vALUES ('{"name": "Laptop", "price": 1500, "tags": ["tech","computer"]}');

-- Lấy giá trị từ JSONB

SELECT data ->>'name' as product_name from products;
```

#### 2.7 UUID

| Kiểu dữ liệu | Kích thước | Ví dụ                                  |
| ------------ | ---------- | -------------------------------------- |
| `UUID`       | 16 bytes   | '550e8400-e29b-41d4-a716-446655440000' |

👉 Thường dùng làm khóa chính thay cho SERIAL, tránh trùng lặp trong hệ thống phân tán.

```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
SELECT uuid_generate_v4();
```

#### 2.8 ARRAY

| Kiểu dữ liệu | Kích thước | Ghi chú                               |
| ------------ | ---------- | ------------------------------------- |
| `UUID`       | thay đổi   | Có thể chứa list giá trị, `'{1,2,3}'` |

```sql
CREATE TABLE students (
  id SERIAL PRIMARY KEY,
  name TEXT,
  scores INT[]
);

INSERT INTO students (name, scores)
VALUES ('Alice', '{85,90,92}');

-- Trích xuất mảng
SELECT name, unnest(scores) FROM students;
```

#### 2.9 MONEY

| Kiểu dữ liệu | Kích thước | Khoảng giá trị                |
| ------------ | ---------- | ----------------------------- |
| `MONEY`      | 8 bytes    | -922 trilion đến +922 trilion |

👉 Không khuyến nghị dùng (do phụ thuộc locale). Nên dùng NUMERIC(12,2) thay thế.

## 3. làm việc với Database & Table

#### 3.1 Tạo Database

```sql
CREATE DATABASE shopdb;
\c shopdb;
```

#### 3.2 Tạo bảng

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email TEXT UNIQUE,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### 3.3 Sửa bảng

```sql
ALTER TABLE users ADD COLUMN age INT;
ALTER TABLE users DROP COLUMN age;
```

#### 3.4 Xóa bảng

## 4. CURD cơ bản

#### 4.1 INSERT

```sql
INSERT INTO users(name, email)
VALUES
  ('Phan Van Tam', 'phanvantam2020@gmail.com'),
  ('Alpha Dev', 'alphadev01@gmail.com');
```

#### 4.2 SELECT

```sql
SELECT
  name,
  email
FROM users
WHERE is_active = true
ORDER BY created_at DESC
LIMIT 10;

```

#### 4.3 UPDATE

```sql
UPDATE users
SET is_active = false
WHERE id = 1;
```

#### 4.4 DELETE

```sql
DELETE FROM users
WHERE id = 1;
```

## 5. Constraints

- `PRIMARY KEY`: khóa chính
- `UNIQUE`: không trùng lặp
- `NOT NULL`: bắt buộc có giá trị
- `DEFAULT`: giá trị mặc định
- `OREIGN KEY`: tham chiếu bảng khác

## 6. Truy vấn

#### 6.1 Toán tử & điều kiện

```sql
  SELECT * FROM users WHERE id > 5;
  SELECT * FROM users WHERE name LIKE 'A%';
  SELECT * FROM users WHERE id IN (1,2,3);
  SELECT * FROM users WhERE created_at BETWEEN '2025-01-01' AND '2025-12-31';
```

#### 6.2 Aggregate functions

```sql
SELECT COUNT(*) FROM users;
SELECT AVG(total) FROM orders;
SELECT SUM(total) FROM orders;
SELECT MAX(total), MIN(total) from orders;
```

#### 6.3 GROUP BY & HAVING

```sql
SELECT user_id, COUNT(*) as order_count, SUM(total) as total_amount
FROM orders
GROUP BY user_id
HAVING SUM(total) > 1000;
```

## 7. JOIN

Trong SQL, JOIN được dùng để kết hợp dữ liệu từ nhiều bảng dựa trên một điều kiện chung (thường là khóa ngoại). PostgreSQL hỗ trợ nhiều loại JOIN khác nhau.

#### 7.1 INNER JOIN

- Chỉ trả về những dòng có dữ liệu khớp ở cả 2 bảng.
- Là loại join phổ biến nhất.

```sql
SELECT emp.id, em.name, dep.id, dep.name
FROM employee AS emp
INNER JOIN department as dep
ON emp.department_id = dep.id
```

👉 Kết quả: chỉ những nhân viên có `department_id` tồn tại trong bảng **department**

#### 7.2 LEFT JOIN (hoặc LEFT OUTER JOIN)

- Trả về tất cả bản ghi bên trái (left table) và dữ liệu khớp bên phải (right table)
- Nếu không khớp, cột bên phải sẽ có giá trị `NULL`.

```sql
SELECT e.id, e.name, d.name AS department
FROM employee e
LEFT JOIN department d ON e.department_id = d.id;
```

👉 Kết quả: tất cả nhân viên, kể cả những người chưa có department (cột department = NULL).

#### 7.3 RIGHT JOIN (hoặc RIGHT OUTER JOIN)

- Ngược với LEFT JOIN: lấy tất cả bản ghi bên phải và chỉ những dữ liệu khớp từ bảng bên trái.

```sql
SELECT e.id, e.name, d.name AS department
FROM employee e
RIGHT JOIN department d ON e.department_id = d.id;
```

👉 Kết quả: tất cả phòng ban, kể cả những phòng chưa có nhân viên nào.
