# Observability & Logging

## Mục tiêu

Biết hệ thống đang khỏe hay lỗi ở đâu thông qua log, metric, tracing và health check.

## Cần nắm

- Structured logging.
- Centralized logging.
- Metrics.
- Tracing.
- Health check.
- Alerting.
- Dashboard.

## Checklist

- Log có timestamp, level, service, request id.
- Error log có stack trace và context.
- Health endpoint kiểm tra dependency quan trọng.
- Metric có latency, error rate, CPU, memory.
- Alert không quá nhiễu.

## Bài thực hành

- Thêm health check endpoint cho API.
- Đẩy log về Seq hoặc ELK.
- Tạo dashboard request latency và error rate.

