# 💻 Giới thiệu về System Design - LLD && HLD

---

**Thiết kế hệ thống là quá trình lập kế hoạch, cấu trúc và xác định kiến trúc của một hệ thống phần mềm**

- Bao gồm việc chuyển đổi các yêu cầu của người dùng thành một bản thiết kế chi tiết để hướng dẫn giai đoạn triển khai.
- Mục tiêu là tạo ra cấu trúc được tổ chức tốt và hiệu quả, đáp ứng mục đích đã định. Đồng thời xem xét các yếu tố như khả năng mở rộng, khả năng bảo trì và hiệu suất.

**Thiết kế hệ thống trong chu kỳ phát triển phần mềm (SDLC)**

- Trong chu kỳ phát triển hệ thống, nếu không có giai đoạn thiết kế, người ta không thể chuyển sang giai đoạn triển khai hoặc thử nghiệm.
- Thiết kế hệ thống cũng là một bước quan trọng và cũng là nền tảng để xử lý các tình huống đặc biệt vì nó thể hiện logic nghiệp vụ của phần mềm.

<div align="center">
  <img src="/digitaldocs/assets/system-design2.webp" alt="Vòng đời thiết kế hệ thống" width="600">
</div>

_Thiết kế hệ thống được chia thành hai phần bổ sung cho nhau_

<div align="center">
  <img src="/digitaldocs/assets/system-design1.webp" alt="Vòng đời thiết kế hệ thống" width="600">
</div>

## Thiết kế cấp cao (High-Level Design - HLD)

- Nó trình bày kiến trúc tổng thể của hệ thống - cách các thành phần chính tương tác với nhau, những dịch vụ hoặc module nào sẽ tồn tại và cách dữ liệu luân chuyển giữa chúng.
- Cung cấp cho bạn cái nhìn tổng quan: cách hệ thống vận hành, cấu trúc cốt lõi và các quyết định quan trọng.

**Kiến thức cần thiết cho HLD**

- Kỹ năng lập trình cơ bản (Cấu trúc dữ liệu và thuật toán).
- Hiểu rõ vai trò của các thành phần như cở sở dữ liệu `(SQL và NoSQL)`, bộ nhớ đệm `(Redis, Memcached, CDN)`và API.
- Thiết kế cấp thấp `(OOP và Design Patterns)`
- Hiểu biết sâu sắc về các yêu cầu chức năng (Hệ thống phải làm gì, ví dụ: đăng kí người dùng, truyền phát dữ liệu) và các yêu cầu phi chức năng (Hệ thống nên hoạt động như thế nào - khả năng mở rộng, độ trễ, tính khả dụng, bảo mật...).
- Kiến thức cơ bản về mạng và bảo mật bao gồm `DNS`, các giao thức `(TCP/UDP, HTTP, WebSockets)`, OAuth, JWT, TLS/SSL, giới hạn tốc độ, bảo mật API và phòng chống tấn công DDoS cơ bản.
- Message Queues và các công cụ truyền dữ liệu trực tuyến như Kafka hoặc RabbitMQ.
- Kiến thức về kiến trúc Microservices so với Monoliths (Khi nào nên tách các dịch vụ và cách quản lý các phụ thuộc), khả năng chịu lỗi, chiến lược dự phòng, tính dư thừa, các loại và thuật toán cân bằng tải.
- Các công cụ giám sát như Prometheus, Grafana, ELK Stack (Elasticseach, Logstash, Kibana) và hệ thống cảnh báo (ví dụ: PagerDuty)

**Các chủ đề được đề cập trong HLD**

- Tổng quan về kiến trúc hệ thống: Mô tả các thành phần chính, các module và cách chúng tương tác với nhau (Services, queues, database).
- Data flow và tương tác giữa các thành phần: Minh hoạt cách data di chuyển giữa các module, cùng với các tích hợp và giao diện chính.
- Công nghệ ngăn xếp và cơ sở hạ tầng: Các quyết định về framework, platforms, hardware, databases và hosting.
- Chức năng của từng phần: Mô tả chức năng của mỗi phần và mối liên hệ giữa chúng.
- Hiệu năng và đánh đổi: Bao gồm các sự đánh đổi trong thiết kế, các yếu tố về hiệu năng, khả năng mở rộng, bảo mật, chi phí và các yếu tố phi chức năng khác.
- Các sơ đồ: Thường bao gồm sơ đồ kiến trúc, sơ đồ thành phần triển khai, sơ đồ luồng dữ liệu, và có thể cả sơ đồ tổng quan ER/DB.

## Thiết kế cấp thấp (Low-Level Design - LLD)

- Mô tả cách thức hoạt động và triển khai nội bộ của từng bộ phận.
- Cung cấp cho các developer một bản kế hoạch rõ ràng, khả thi để xây dựng từng bộ phận.

**Điều kiện tiên quyết/Những điều bạn cần biết trước khi bắt đầu**

- Kỹ năng lập trình cơ bản (Cấu trúc dữ liệu và thuật toán).
- Nắm vững các khái niệm lập trình OOP (Đóng gói, kế thừa, đa hình và trừu tượng).

**Các chủ đề được đề cập trong LLD**

- Phân tích thành phần/module: Logic nội bộ chi tiết cho từng module bao gồm trách nhiệm: class, methods, attributes, interactions.
- Sơ đồ và cấu trúc dữ liệu: Thiết kế bảng, khóa, chỉ mục, mối quan hệ với các cải tiến SQL/NoSQL.
- Định nghĩa API và giao diện: Định dạng yêu cầu/phản hổi chính xác, mã lỗi, phương thức, điểm cuối và giao diện nội bộ.
- Logic xử lý lỗi và xác thực: Xác định cách mỗi module xử lý các input không hợp lệ, lỗi, trường hợp exception và ghi logs.
- Design patterns và SOLID: Áp dụng các design patterns và nguyên tắc SOLID để đảm bảo code sạch, có khả năng mở rộng và dễ bảo trì.
- Các thành phần UML và mã giả: Sơ đồ lớp, sơ đồ tuần tự, mã giả hoặc lưu đồ.

## Các bước để bắt đầu với System Design

_Dưới đây là một số bước để bắt đầu system design:_

<div align="center">
  <img src="/digitaldocs/assets/system-design3.webp" alt="Các bước bắt đầu thiết kế hệ thống" width="600">
</div>

- Hiểu rõ yêu cầu: Thu thập và phân tích nhu cầu nghiệp vụ.
- Xác định kiến trúc của hệ thống: Xác định thành phần chính của hệ thống và cách chúng tương tác (ví dụ: service, API, databases).
- Chọn các công nghệ phù hợp: Chọn các ngôn ngữ lập trình, database, framework và tool phù hợp.
- Thiết kế các module: Chia hệ thống thành các module, xác định chức năng và luồng dữ liệu của chúng.
- Lập kế hoạch: Thiết kế khả năng mở rộng với tầm nhìn hướng đến sự phát triển - dự đoán tải, tối ưu các điểm nghẽn và sử dụng các mô hình có khả năng mở rộng.
- Đảm bảo bảo mật và quyền riêng tư: Xác định rủi ro và triển khai các biện pháp như xác thực, mã hóa và bảo vệ dữ liệu.
- Kiểm thử và mô phỏng

System design là một quá trình phức tạp đòi hỏi sự lập kế hoạch cẩn thận và chú trọng đến từng chi tiết.
Bằng cách làm theo các bước này, có thể tạo ra một thiết kế hệ thống đáp ứng nhu cầu của người dùng và doanh nghiệp.

## Những điểm quan trọng cần xem xét khi System Design

- **1. Khả năng mở rộng:** Hệ thống cần được thiết kế để xử lý khi tải tăng lên và có khả năng mở rộng theo chiều ngang hoặc chiều dọc.
- **2. Hiệu năng:** Hệ thống cần được thiết kế để hoạt động hiểu quả và tối ưu, với độ trễ và thời gian tối thiểu.
- **3. Độ tin cậy:** Hệ thống cần phải đáng tin cậy và luôn hoạt động, với thời gian ngừng hoạt động hoặc lỗi hệ thống ở mức tối thiểu.
- **4. Bảo mật:** Hệ thống cần được thiết kế với tiêu chí bảo mật, bao gồm các biện pháp ngăn chặn truy cập trái phép và bảo vệ dữ liệu nhạy cảm.
- **5. Khả năng bảo trì:** Hệ thống cần được thiết kế để dễ bảo trì và cập nhật, với tài liệu rõ ràng và code được tổ chức tốt.
- **6. Khả năng tương tác:** Hệ thống cần được thiết kế để hoạt động liền mạch với các hệ thống và thành phần khác, với giao diện rõ ràng và được xác định tốt.
- **7. Khả năng sử dụng:** Hệ thống cần được thiết kế thân thiện và trực quan với người dùng, với giao diện người dùng rõ ràng và nhất quán.
- **8. Hiệu quả chi phí:** Hệ thống cần được thiết kế sao cho hiệu quả về chi phí, tập trung vào việc giảm thiểu chi phí phát triển và vận hành trong khi vẫn đáp ứng được các yêu cầu.
