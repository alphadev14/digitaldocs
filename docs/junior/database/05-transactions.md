# Transaction trong PostgreSQL

## 1. Transaction là gì?

**Transaction** là một **tập hợp các thao tác (câu lệnh SQL)** được thực hiện như một **đơn vị logic duy nhất**.  
Tất cả các thao tác trong transaction **hoặc cùng thành công (COMMIT)**, **hoặc cùng thất bại (ROLLBACK)**.  
Điều này giúp đảm bảo tính toàn vẹn dữ liệu khi có lỗi hoặc sự cố trong quá trình xử lý.

---

## 2. Bốn tính chất ACID của Transaction

| Tính chất       | Ý nghĩa                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| **Atomicity**   | Tính nguyên tử – Toàn bộ transaction được xem như một đơn vị không thể chia nhỏ. Nếu một phần thất bại, toàn bộ transaction bị hủy. |
| **Consistency** | Sau khi transaction hoàn thành, dữ liệu trong database luôn ở trạng thái hợp lệ (đúng với ràng buộc dữ liệu).                       |
| **Isolation**   | Mỗi transaction hoạt động độc lập, không bị ảnh hưởng bởi transaction khác.                                                         |
| **Durability**  | Sau khi COMMIT, thay đổi sẽ được lưu vĩnh viễn trong hệ thống (ngay cả khi hệ thống bị tắt hoặc lỗi).                               |

---

## 3. Cách sử dụng Transaction trong PostgreSQL

### 🔹 Bắt đầu và kết thúc Transaction

```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;
```

- **BEGIN**: bắt đầu transaction.
- **COMMIT**: xác nhận và lưu các thay đổi vào database.
- **ROLLBACK**: hủy tất cả thay đổi trong transaction (nếu có lỗi).

---

### 🔹 Ví dụ minh họa với ROLLBACK

```sql
BEGIN;

UPDATE products SET quantity = quantity - 10 WHERE id = 1;
UPDATE orders SET total = total + 500 WHERE id = 99;

-- Có lỗi xảy ra ở đây
ROLLBACK;
```

➡ Khi thực hiện `ROLLBACK`, tất cả thay đổi trước đó trong transaction đều bị hủy bỏ.

---

## 4. Savepoint trong Transaction

Savepoint cho phép bạn **tạo điểm khôi phục tạm thời** trong transaction, giúp rollback **một phần** thay vì toàn bộ.

```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
SAVEPOINT sp1;

UPDATE accounts SET balance = balance - 200 WHERE id = 2;
ROLLBACK TO sp1; -- Quay lại trước khi trừ 200

COMMIT;
```

---

## 5. Isolation Levels (Mức độ cô lập)

PostgreSQL hỗ trợ 4 mức độ cô lập (Isolation Level):

| Level                           | Mô tả                                        | Nguy cơ gặp phải    |
| ------------------------------- | -------------------------------------------- | ------------------- |
| **READ UNCOMMITTED**            | Cho phép đọc dữ liệu chưa COMMIT.            | Dirty Read          |
| **READ COMMITTED** _(mặc định)_ | Chỉ đọc dữ liệu đã COMMIT.                   | Non-repeatable Read |
| **REPEATABLE READ**             | Giữ snapshot dữ liệu trong suốt transaction. | Phantom Read        |
| **SERIALIZABLE**                | Giả lập thực thi tuần tự tuyệt đối.          | Không có nguy cơ    |

Thiết lập isolation level:

```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

---

## 6. Khi nào nên dùng Transaction?

| Tình huống                   | Lý do                                          |
| ---------------------------- | ---------------------------------------------- |
| Chuyển tiền giữa 2 tài khoản | Đảm bảo 2 thao tác cập nhật diễn ra toàn vẹn   |
| Đặt hàng & trừ tồn kho       | Tránh lỗi khi trừ kho nhưng không ghi đơn hàng |
| Ghi log nhiều bảng           | Đảm bảo tất cả bảng được cập nhật nhất quán    |

---

## 7. Tổng kết

| Câu lệnh           | Chức năng                    |
| ------------------ | ---------------------------- |
| `BEGIN;`           | Bắt đầu Transaction          |
| `COMMIT;`          | Xác nhận và lưu thay đổi     |
| `ROLLBACK;`        | Hủy bỏ toàn bộ thay đổi      |
| `SAVEPOINT sp1;`   | Tạo điểm lưu trung gian      |
| `ROLLBACK TO sp1;` | Quay lại điểm lưu trung gian |

**👉 Ghi nhớ:** Transaction giúp đảm bảo **tính toàn vẹn và an toàn dữ liệu** trong các hệ thống thực tế có nhiều thao tác liên quan nhau.

# Transaction trong PostgreSQL

## 1. Transaction là gì?

**Transaction** là một **tập hợp các thao tác (câu lệnh SQL)** được thực hiện như một **đơn vị logic duy nhất**.  
Tất cả các thao tác trong transaction **hoặc cùng thành công (COMMIT)**, **hoặc cùng thất bại (ROLLBACK)**.  
Điều này giúp đảm bảo tính toàn vẹn dữ liệu khi có lỗi hoặc sự cố trong quá trình xử lý.

---

## 2. Bốn tính chất ACID của Transaction

| Tính chất       | Ý nghĩa                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| **Atomicity**   | Tính nguyên tử – Toàn bộ transaction được xem như một đơn vị không thể chia nhỏ. Nếu một phần thất bại, toàn bộ transaction bị hủy. |
| **Consistency** | Sau khi transaction hoàn thành, dữ liệu trong database luôn ở trạng thái hợp lệ (đúng với ràng buộc dữ liệu).                       |
| **Isolation**   | Mỗi transaction hoạt động độc lập, không bị ảnh hưởng bởi transaction khác.                                                         |
| **Durability**  | Sau khi COMMIT, thay đổi sẽ được lưu vĩnh viễn trong hệ thống (ngay cả khi hệ thống bị tắt hoặc lỗi).                               |

---

## 3. Cách sử dụng Transaction trong PostgreSQL

### 🔹 Bắt đầu và kết thúc Transaction

```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;
```

- **BEGIN**: bắt đầu transaction.
- **COMMIT**: xác nhận và lưu các thay đổi vào database.
- **ROLLBACK**: hủy tất cả thay đổi trong transaction (nếu có lỗi).

---

### 🔹 Ví dụ minh họa với ROLLBACK

```sql
BEGIN;

UPDATE products SET quantity = quantity - 10 WHERE id = 1;
UPDATE orders SET total = total + 500 WHERE id = 99;

-- Có lỗi xảy ra ở đây
ROLLBACK;
```

➡ Khi thực hiện `ROLLBACK`, tất cả thay đổi trước đó trong transaction đều bị hủy bỏ.

---

## 4. Savepoint trong Transaction

Savepoint cho phép bạn **tạo điểm khôi phục tạm thời** trong transaction, giúp rollback **một phần** thay vì toàn bộ.

```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
SAVEPOINT sp1;

UPDATE accounts SET balance = balance - 200 WHERE id = 2;
ROLLBACK TO sp1; -- Quay lại trước khi trừ 200

COMMIT;
```

---

## 5. Isolation Levels (Mức độ cô lập)

PostgreSQL hỗ trợ 4 mức độ cô lập (Isolation Level):

| Level                           | Mô tả                                        | Nguy cơ gặp phải    |
| ------------------------------- | -------------------------------------------- | ------------------- |
| **READ UNCOMMITTED**            | Cho phép đọc dữ liệu chưa COMMIT.            | Dirty Read          |
| **READ COMMITTED** _(mặc định)_ | Chỉ đọc dữ liệu đã COMMIT.                   | Non-repeatable Read |
| **REPEATABLE READ**             | Giữ snapshot dữ liệu trong suốt transaction. | Phantom Read        |
| **SERIALIZABLE**                | Giả lập thực thi tuần tự tuyệt đối.          | Không có nguy cơ    |

Thiết lập isolation level:

```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

---

## 6. Khi nào nên dùng Transaction?

| Tình huống                   | Lý do                                          |
| ---------------------------- | ---------------------------------------------- |
| Chuyển tiền giữa 2 tài khoản | Đảm bảo 2 thao tác cập nhật diễn ra toàn vẹn   |
| Đặt hàng & trừ tồn kho       | Tránh lỗi khi trừ kho nhưng không ghi đơn hàng |
| Ghi log nhiều bảng           | Đảm bảo tất cả bảng được cập nhật nhất quán    |

---

## 7. Tổng kết

| Câu lệnh           | Chức năng                    |
| ------------------ | ---------------------------- |
| `BEGIN;`           | Bắt đầu Transaction          |
| `COMMIT;`          | Xác nhận và lưu thay đổi     |
| `ROLLBACK;`        | Hủy bỏ toàn bộ thay đổi      |
| `SAVEPOINT sp1;`   | Tạo điểm lưu trung gian      |
| `ROLLBACK TO sp1;` | Quay lại điểm lưu trung gian |

**👉 Ghi nhớ:** Transaction giúp đảm bảo **tính toàn vẹn và an toàn dữ liệu** trong các hệ thống thực tế có nhiều thao tác liên quan nhau.
