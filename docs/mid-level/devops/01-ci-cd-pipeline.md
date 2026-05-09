# CI/CD Pipeline

## Mục tiêu

Tự động hóa quá trình build, test và deploy để giảm lỗi thủ công.

## Cần nắm

- Pipeline stages.
- Build artifact.
- Test automation.
- Environment variables.
- Secret management.
- Deployment job.
- Manual approval.

## Checklist

- Pipeline có stage build, test, deploy.
- Secret không hard-code trong repo.
- Deploy production có approval nếu cần.
- Artifact/version được trace rõ.
- Khi fail có log đủ để debug.

## Bài thực hành

- Tạo GitLab CI build .NET API.
- Thêm stage test.
- Deploy lên Render hoặc VPS qua SSH.

