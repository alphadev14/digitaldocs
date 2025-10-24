# JSONB in PostgreSQL

## 1. Giới thiệu

- **JSONB** là kiểu dữ liệu trong PostgreSQL dùng để lưu trữ dữ liệu **JSON ở dạng nhị phân (binary)**.
- Khác với `JSON` (lưu ở dạng text gốc), `JSONB` được **parse, nén và tối ưu hóa truy vấn**, giúp tốc độ xử lý cao hơn đáng kể.
- Ưu điểm:
  - Có thể truy vấn trực tiếp vào key bên trong JSON.
  - Có thể tạo **index GIN/GiST** để tìm kiếm nhanh.
  - Không cần cấu trúc bảng cứng nhắc → linh hoạt với dữ liệu thay đổi.

---

## 2. Tạo bảng với JSONB

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT,
    profile JSONB
);
```

### Ví dụ dữ liệu:

```sql
INSERT INTO users (name, profile) VALUES
('Phan', '{"age": 25, "city": "HCM", "skills": ["C#", "SQL", "React"]}'),
('Tam', '{"age": 30, "city": "Danang", "skills": ["Python", "PostgreSQL"]}');
```

---

## 3. Truy cập và truy vấn dữ liệu JSONB

| Toán tử | Mô tả                             | Ví dụ                          |
| ------- | --------------------------------- | ------------------------------ |
| `->`    | Lấy giá trị JSON (dạng JSON)      | `profile -> 'city'`            |
| `->>`   | Lấy giá trị JSON (dạng text)      | `profile ->> 'city'`           |
| `#>`    | Truy cập vào key lồng nhau (JSON) | `profile #> '{skills,0}'`      |
| `#>>`   | Truy cập vào key lồng nhau (text) | `profile #>> '{skills,1}'`     |
| `@>`    | Kiểm tra chứa JSON con            | `profile @> '{"city": "HCM"}'` |
| `?`     | Kiểm tra key tồn tại              | `profile ? 'age'`              |

### Ví dụ:

```sql
-- Lấy danh sách thành phố
SELECT name, profile ->> 'city' AS city FROM users;

-- Lấy kỹ năng đầu tiên
SELECT name, profile #>> '{skills,0}' AS first_skill FROM users;

-- Lọc user ở Hà Nội
SELECT * FROM users WHERE profile @> '{"city": "HCM"}';

-- Kiểm tra có key "skills" hay không
SELECT * FROM users WHERE profile ? 'skills';
```

---

## 4. Cập nhật JSONB

### Cập nhật giá trị trong JSONB:

```sql
UPDATE users
SET profile = jsonb_set(profile, '{city}', '"VIETNAM"')
WHERE name = 'Tam';
```

### Thêm phần tử vào mảng:

```sql
UPDATE users
SET profile = jsonb_set(profile, '{skills}', (profile->'skills') || '"NodeJS"')
WHERE name = 'Tam';
```

---

## 5. Truy vấn nâng cao

### Tìm tất cả user có kỹ năng "React"

```sql
SELECT *
FROM users
WHERE profile->'skills' ? 'React';
```

### Tìm user có độ tuổi > 25

```sql
SELECT *
FROM users
WHERE (profile->>'age')::int > 25;
```

### Lọc user có nhiều kỹ năng hơn 2

```sql
SELECT *
FROM users
WHERE jsonb_array_length(profile->'skills') > 2;
```

---

## 6. Tạo Index cho JSONB

### Index GIN

```sql
CREATE INDEX idx_users_profile_gin ON users USING GIN (profile);
```

Sau đó bạn có thể truy vấn cực nhanh:

```sql
SELECT * FROM users WHERE profile @> '{"city": "HCM"}';
```

---

## 7. Các hàm tiện ích JSONB phổ biến

| Hàm                                   | Mô tả                           | Ví dụ                                            |
| ------------------------------------- | ------------------------------- | ------------------------------------------------ |
| `jsonb_build_object(key, value, ...)` | Tạo JSONB từ cặp key-value      | `jsonb_build_object('age', 25, 'city', 'HCM')`   |
| `jsonb_each(jsonb)`                   | Tách JSONB thành từng key-value | `SELECT * FROM jsonb_each(profile)`              |
| `jsonb_array_elements(jsonb)`         | Tách từng phần tử trong mảng    | `SELECT jsonb_array_elements(profile->'skills')` |
| `jsonb_object_keys(jsonb)`            | Trả về danh sách key            | `SELECT jsonb_object_keys(profile)`              |

---

## 8. Ưu & Nhược điểm của JSONB

### Ưu điểm:

- Linh hoạt: không cần định nghĩa schema cứng.
- Truy vấn sâu bên trong dữ liệu JSON.
- Hỗ trợ index mạnh mẽ (GIN).
- Tích hợp tốt với ứng dụng web (API, microservices).

### Nhược điểm:

- Không có ràng buộc kiểu dữ liệu → dễ sai sót nếu không kiểm tra.
- Truy vấn phức tạp có thể chậm hơn bảng quan hệ nếu dữ liệu lớn.
- Khó maintain khi schema thay đổi liên tục.

---

## 9. Khi nào nên dùng JSONB?

✅ **Nên dùng khi:**

- Dữ liệu có cấu trúc linh hoạt, không cố định.
- Dữ liệu đến từ API ngoài hoặc log event.
- Không cần JOIN phức tạp giữa nhiều bảng.

❌ **Không nên dùng khi:**

- Dữ liệu quan hệ rõ ràng, cần JOIN thường xuyên.
- Muốn tận dụng constraint, foreign key, và index chi tiết.

---

## 10. Ví dụ thực tế

### Lưu thông tin đơn hàng (Order metadata linh hoạt)

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INT,
    metadata JSONB
);

INSERT INTO orders (customer_id, metadata)
VALUES
(1, '{"delivery": {"type": "express", "fee": 30000}, "coupon": "SALE20"}'),
(2, '{"delivery": {"type": "standard", "fee": 15000}}');

-- Truy vấn tất cả đơn có mã giảm giá
SELECT * FROM orders WHERE metadata ? 'coupon';
```
