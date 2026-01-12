# Các Component của System Design

---

Thiết kế hệ thống bao gồm việc xem xét các yêu cầu của hệ thống, xác định các giả định và hạn chế của nó, và định nghĩa cấu trúc và các thành phần cấp cao của hệ thống.
Các yếu tố chính của thiết kế hệ thống bao gồm service, database, load balancers và messageing system

<div align="center">
  <img src="/digitaldocs/assets/componentsofasystem21.png" alt="component của system design" width="600">
</div>

## 1. Load Balancers

Một hệ thống có nhiều server và phải phân chia các yêu cầu một cách đồng điều giữa chúng, hoặc khi một hệ thống nhận được một lượng lớn request và muốn chia chúng cho nhiều service để tránh quá tải bất kì máy nào.
Load Balancers được đùng để phân bổ các request đến các server khác nhau để hạn chế việc quá tải của server.

**Một số loại Load Balancers phổ biến hiện nay**

- **Layer 4 load balancers**: Nó hoạt động ở lớp Vận chuyển của mô hình OSI, phân bổ các yêu cầu dựa trên số cổng TCP/UDP và địa chỉ IP nguồn và đích .
- **Layer 7 load balancers**: Phân phối các yêu cầu dựa trên nội dung của chúng, bao gồm URL hoặc loại phương thức URL hoặc loại phương thức HTTP, hoặt động lớp ứng dụng của mô hình OSI.
- **Global load balancers**: Được sử dụng trong các hệ thống phân tán để phân phối các request giữa nhiều máy chủ đặt tại các khu vực địa lý khác nhau
- **Application load balancers**: Là các bộ cân bằng tải chuyên dụng được thiết kế để hoạch động với các ứng dụng hoặc giao thức cụ thể, chẳng hạng như HTTP hoặc HTTPS.

---

## 2. Caching

Kỹ thuật lưu trữ tạm thời dữ liệu được yêu cầu thường xuyên, gúp tăng tốc độ truy vấn khi cần thiết, được gọi là **Caching**. Khi Caching được tích hợp vào kiến trúc hệ thống, database sẽ ít bị quá tải hơn, từ đó nâng cao hiệu suất và hiệu quả hơn.

**Cách thức hoạt động của Caching**

- **Truy cập nhanh**: Khi dữ liệu được lưu vào caching, nó lưu trữ trong bộ nhớ tạm thời nhanh hơn (như bộ nhớ RAM) thay vì phải liên tục truy xuất từ database chậm hơn.
  Điều này giúp việc truy xuất thông tin nhanh hơn.
- **Giảm tải cơ sở dữ liệu**: Hệ thống có thể xử lý nhiều yêu cầu hơn mà không gặp phải trình trạng chậm trễ bằng cách lưu trữ dữ liệu thường xuyên sử dụng hoặc phổ biến trong bộ nhớ đệm thay vì liên tục truy vấn database.
- **Cải thiện trải nghiệm người dùng**: Truy cập dữ liệu nhanh hơn đồng nghĩa với thời gian phản hồi nhanh hơn cho người dùng.

---

## 3. Content Delivery Network (CDN)

Là một mạng lưới các máy chủ trải rộng khái các khu vực khác nhau, cho phép phân phối nội dung nhanh hơn đến người dùng, chẳng hạn như trang web, phim và ảnh. CDN được sử dụng trong hầu hết các thiết kế hệ thống để tăng tốc độ và tin cậy của việc phân phối nội dung đến người dùng, đặc biệt là phân tán ở nhiều vị trí địa lý khác nhau.

**Cách thức hoạt động của CDN**
Khi người dùng yêu cầu nội dung (như video hoặc hình ảnh), thay vì lấy nội dung từ máy chủ chính, yêu cầu được gởi xử lý bởi một máy chủ CDN gần đó, nơi lưu trữ bản sao nội dung đã được biên tập. Điều này giúp giảm khoảng cách dữ liệu phải duy chuyển, giúp nội dung tải nhanh hơn cho người dùng.

---

## 4. API Gateways

API Gateways giống như một cánh cửa trung tâm hoặc "bộ điều khiển giao thông" cho các request đến hệ thống. Trong thiết kế hệ thống, nó hoạt động như một điểm duy nhất cho các máy khách để truy cập nhiều dịch vụ phụ trợ một cách có tổ chức và an toàn.

**Cách thức hoạt động của API Gateways**

- **Request Routing**: client gửi request đến API Gateway khi muốn nhận thông tin hoặc thực thực hiện các thao tác từ hệ thống có nhiều server. Sau khi chuyển tiếp request đến các service thích hợp, API Gateway sẽ tổng hợp lại các câu trả lời của chúng và cung cấp cho máy khách một phản hồi duy nhất, thống nhất.
- **Simplifies Client Interaction**: Nếu không có API Gateway, khách hàng có thể cần kết nối trực tiếp với từng server riêng lẻ, điều này có thể rất phức tạp. API Gateway đơn giản hóa điều khiển này bằng cách ẩn các chi tiết về server khỏi client.
- **Security and Monitoring**: API Gateway có thể xử lý các tính năng bảo mật như: Authencation (Kiểm tra danh tính người dùng) và ủy Authozition (Kiểm tra quyền của người dùng), phát hiện sự cố hoặc ngăn chặn các cuộc tấn công.
- **Load Management**: Trong các hệ thống lớn, API Gateway có thể giúp cân bằng và kiểm soát request để ngăn ngừa quá tải, đảm bảo hiệu suất hoạt động mượt mà hơn.

---

## 5. Key-Value Stores

NoSQL được thiết kế để lưu trữ dữ liệu dưới dạng `KEY - VALUES`. Mỗi dữ liệu được lưu trữ với khóa duy nhất, cho phép truy cập nhanh chóng bằng KEY. NoSQL được lưu trữ dữ liệu được truy cập thường xuyên.

---

## 6. Blob Storage & Databases

Blob storage và Databases là hai loại hệ thống lưu trữ khác nhau có thể được sử dụng để lưu trữ và quản lý dữ liệu.

Khối lượng lớn dữ liệu phi cấu trúc, bao gồm tài liệu, ảnh, video và tệp âm thanh, có thể được lưu trữ trong Blob storage. Nhìn chung, hệ thống Blob storage có khả năng mở rộng rất cao và có thể xử lý nhiều yêu cầu cùng một lúc. Chúng được sử dụng rộng rãi để lưu trữ các tài liệu được truy cập thường xuyên, chẳng hạn như nội dung do người dùng tạo hoặc các tệp phương tiện.

Mặt khác, Databases được tạo ra để lưu trữ dữ liệu có cấu trúc đã được sắp xếp theo một cách cụ thể. Databases thường được sử dụng để lưu trữ dữ liệu cần được truy vấn và truy cập theo cách có cấu trúc, chẳng hạn như hồ sơ khách hàng hoặc giao dịch tài chính.

---

## 7. Rate Limiters

Được sử dụng để hạn chế số lượng phản hồi request hoặc thực hiện các tác vụ cụ thể của hệ thống hoặc ứng dụng. Điều này có thể hữu ích trong nhiều tình huống, chẳng hạn như khi hệ thống cần phải ngăn chặn việc nhận quá nhiều request từ người dùng hoặc hacker có thể ảnh hướng đến hiệu suất của hệ thống.

**Một số loại Rate limiters phổ biến**

- **Request rate limiters are**: Được sử dụng để giới hạn số lượng request mà một hệ thống hoặc ứng dụng xử lý trong một khoảng thời gian nhất định.
- **User rate limiter**: Được sử dụng để giới hạn tốc độ mà một người dùng cụ thể hoặc một nhóm người dùng có thể gửi request đến hệ thống hoặc ứng dụng.
- **Token bucket rate limiters (Token Bucket Rate Limiters)**: Được sử dụng để giới hạn tốc độ xử lý yêu cầu của hệ thống bằng cách cho phép cách cho phép xử lý một số lượng request nhất định trong mỗi khoảng thời gian, với bất kì request nào vượt quá số lượng đó sẽ được giữ lại trong một **"thùng" (bucket)** cho đến khoảng thời gian tiếp theo.

---

## 8. Monitoring System

Monitoring System được sử dụng để thu thập, phân tích và báo cáo về các chỉ số và dữ liệu hiệu suất khác nhau liên quan đến một hệ thống hoặc ứng dụng. Điều này có thể hữu ích trong nhiều tính huống khác nhau, chẳng hạn như khi một hệ thống cần theo dõi hiệu suất và tính khả dụng của chính nó để đảm bào rằng chúng đáp ứng được mức độ dịch vụ mong muốn.

**Một số Monitoring System phổ biến**

- **Network monitoring systems**: Được sử dụng để theo dõi hiệu suất của network và các thành phần khác nhau của nó như: router, switch, server.
- **System monitoring systems**: Được sử dụng để theo dõi hiệu suất của system và các thành phần khác nhau của nó như: CPU, memory, disk usage.
- **Application monitoring systems**: Được sử dụng để theo dõi hiệu suất của các ứng dụng hoặc dịch vụ cụ thể như: server web hoặc database.

---

## 9. Messaging Queue

Messaging Queue là một hệ thống cho phép trao đổi message giữa các nodes khác nhau trong một hệ thống phân tán. Messaging Queue cho phép các nút giao tiếp bất đồng bộ, tách rời người gửi và người nhận tin nhắn, cho phép mỗi nodes hoạt động độc lập.

**Một số loại Messaging Queue**

- **Point-to-point queues**: Các thông điệp được gửi đến một người nhận cụ thể.
- **Publish-subscribe queues**: Các tin nhắn được published lên một topic và được gửi đến tất cả những subscribers chủ đề đó..
- **Hybrid queues**: Kết hợp các yếu tố của cả **Point-to-point queues** và **Publish-subscribe queues**, cho phép gửi tin nhắn đến người nhận cụ thể hoặc đến tất cả người đăng ký một chủ đề.

---

Ngoài ra mình còn có **Distributed unique id generator**, **Distributes search**, **Distributed logging services**, **Distributes task scheduler**.
