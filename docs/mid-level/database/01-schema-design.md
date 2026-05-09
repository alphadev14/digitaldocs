# Schema Design

## Mục tiêu

Thiết kế database schema rõ ràng, đúng quan hệ dữ liệu và dễ thay đổi khi nghiệp vụ phát triển.

## Cần nắm

- Primary key và foreign key.
- Unique constraint.
- Normalization và denormalization.
- Audit columns.
- Soft delete.
- Naming convention.
- Data type phù hợp.

## Checklist

- Table có primary key rõ ràng.
- Relationship có foreign key nếu cần enforce consistency.
- Column có data type đúng, không dùng text/string cho mọi thứ.
- Unique rule được enforce ở database.
- Có index cho foreign key thường join/filter.
- Soft delete có chiến lược filter và index phù hợp.

## Bài thực hành

- Thiết kế schema order, order item, product, customer.
- Thêm constraint chống trùng email.
- So sánh schema normalized và denormalized cho report.

