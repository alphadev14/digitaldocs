# Senior Roadmap

Phần Senior đang là khu vực mở rộng cho các chủ đề kiến trúc, vận hành hệ thống lớn và ra quyết định kỹ thuật trong bối cảnh có nhiều ràng buộc.

## Chủ đề sẽ phát triển

| Nhóm | Nội dung |
| --- | --- |
| Architecture Decision | Ghi ADR, phân tích trade-off, chọn hướng triển khai phù hợp bối cảnh |
| Distributed System | Consistency, availability, idempotency, message delivery, distributed transaction |
| Reliability | SLO/SLA, incident review, graceful degradation, fallback và disaster recovery |
| Performance & Cost | Capacity planning, profiling, bottleneck analysis, tối ưu chi phí hạ tầng |
| Engineering Leadership | Code review, mentoring, technical roadmap, quality gate và team convention |

## Cách đọc trước khi phần này hoàn thiện

Nếu bạn đang ở Mid-Level, nên hoàn thành trước các mảng sau:

1. [Backend Performance](../mid-level/backend/performance-in-backend.md)
2. [Resilience & Observability](../mid-level/backend/05-resilience-and-observability.md)
3. [Database Performance](../mid-level/database/05-database-performance.md)
4. [Deployment Strategy](../mid-level/devops/05-deployment-strategy.md)
5. [System Design Introduction](../mid-level/system-design/01-system-design-introduction.md)

## Checklist định hướng Senior

- Giải thích được vì sao chọn kiến trúc hiện tại, không chỉ mô tả kiến trúc.
- Biết đánh đổi giữa latency, consistency, reliability, cost và complexity.
- Có thói quen ghi lại quyết định kỹ thuật quan trọng bằng ADR.
- Thiết kế được hệ thống có cơ chế quan sát, rollback và phục hồi khi lỗi.
- Review code theo rủi ro sản phẩm, không chỉ theo style.
