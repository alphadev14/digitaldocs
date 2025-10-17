# 03 - PostgreSQL Transaction Isolation Levels

## 1. Giới thiệu

Isolation Level xác định cách các transaction tương tác với nhau trong cơ sở dữ liệu.
Giúp đảm bảo tính nhất quán và tránh xung đột khi nhiều transaction chạy song song.

## 2. Các hiện tượng có thể xảy ra

| Hiện tượng              | Mô tả                           | Ví dụ                                                                            |
| ----------------------- | ------------------------------- | -------------------------------------------------------------------------------- |
| **Dirty Read**          | Đọc dữ liệu chưa commit         | Transaction A update giá = 2000 (chưa commit), B đọc thấy 2000. A rollback → sai |
| **Non-repeatable Read** | Đọc 2 lần thấy khác nhau        | A đọc giá = 1000, B update & commit giá = 2000, A đọc lại thấy 2000              |
| **Phantom Read**        | Số dòng thay đổi giữa 2 lần đọc | A SELECT > 100, B insert thêm dòng thỏa điều kiện, A đọc lại thấy nhiều hơn      |

## 3. Các mức Isolation trong PostgreSQL

| Level                        | Dirty Read | Non-repeatable Read | Phantom Read     |
| ---------------------------- | ---------- | ------------------- | ---------------- |
| **Read Uncommitted**         | ❌ Có thể  | ❌ Có thể           | ❌ Có thể        |
| **Read Committed (default)** | ✅ Không   | ❌ Có thể           | ❌ Có thể        |
| **Repeatable Read**          | ✅ Không   | ✅ Không            | ❌ Có thể (hiếm) |
| **Serializable**             | ✅ Không   | ✅ Không            | ✅ Không         |

## 4. Chi tiết từng mức

### 4.1 Read Uncommitted

- Cho phép đọc dữ liệu chưa commit.
- PostgreSQL thực tế xử lý giống Read Committed.

### 4.2 Read Committed (mặc định)

- Mỗi câu SELECT chỉ thấy dữ liệu commit tại thời điểm chạy.
- Nếu transaction khác commit sau đó, lệnh SELECT kế tiếp sẽ thấy dữ liệu mới.

```sql
BEGIN;
SELECT amount FROM orders WHERE id = 1;  -- 1000

-- Session 2
UPDATE orders SET amount = 2000 WHERE id = 1;
COMMIT;

-- Session 1
SELECT amount FROM orders WHERE id = 1;  -- 2000
COMMIT;
```

### 4.3 Repeatable Read

- Snapshot cố định trong suốt transaction.
- Mọi SELECT đều thấy cùng dữ liệu.

```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT amount FROM orders WHERE id = 1;  -- 1000

-- Session 2
UPDATE orders SET amount = 2000 WHERE id = 1;
COMMIT;

-- Session 1
SELECT amount FROM orders WHERE id = 1;  -- 1000 (không đổi)
COMMIT;
```

### 4.4 Serializable

- Mô phỏng như chạy tuần tự từng transaction.
- PostgreSQL sẽ rollback nếu phát hiện xung đột logic.

```sql
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT SUM(balance) FROM accounts WHERE type = 'user';
-- Transaction khác INSERT thêm dòng
COMMIT; -- Có thể bị serialization_failure
```

## 5. Thiết lập Isolation Level

```sql
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- hoặc
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SHOW TRANSACTION ISOLATION LEVEL;
```

## 6. Khi nào dùng

| Mức                 | Khi nào dùng                      | Đặc điểm                  |
| ------------------- | --------------------------------- | ------------------------- |
| **Read Committed**  | Mặc định, đủ cho hầu hết ứng dụng | Cân bằng tốc độ & an toàn |
| **Repeatable Read** | Báo cáo, truy vấn đọc nhiều       | Snapshot cố định          |
| **Serializable**    | Giao dịch tài chính               | An toàn tối đa, chậm hơn  |

## 7. Tổng kết

| Level           | Nhất quán  | Hiệu năng  | Ứng dụng           |
| --------------- | ---------- | ---------- | ------------------ |
| Read Committed  | Trung bình | Nhanh      | Web API, CRUD      |
| Repeatable Read | Cao        | Trung bình | Báo cáo            |
| Serializable    | Rất cao    | Thấp       | Ngân hàng, kế toán |
