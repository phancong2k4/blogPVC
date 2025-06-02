---
title: "Deliverable 2"
date: "2025-05-05"
updated: "2025-05-05"
categories:
  - "sveltekit"
  - "markdown"
excerpt: Deliverable 2
---

## 1. Bản vẽ kiến trúc hệ thống

Hệ thống giám sát RPS gồm các thành phần chính: Client (hoặc bộ sinh request), NGINX, ba app Node.js (app-1, app-2, app-3), ba cơ sở dữ liệu InfluxDB, một API tổng hợp và giao diện Frontend. Client hoặc Request Generator gửi request đến NGINX; NGINX đóng vai trò load balancer (sử dụng phương pháp vòng tròn hoặc IP-hash)
nginx.org để phân phối đều request đến các container Node.js. Mỗi Node.js ghi nhận lưu lượng request (tính RPS) và lưu trữ số liệu vào InfluxDB riêng của nó (xác định bằng biến môi trường NAME). Sau đó, API truy vấn đồng thời ba InfluxDB để tổng hợp dữ liệu RPS trả về cho Frontend.

![sơ đồ trình tự ](/images/result_sơ%20đồ%20kiến%20trúc%20hệ%20thống.drawio.png) 



## 2. Mô tả chi tiết các thành phần trong hệ thống

### 2.1. NGINX Load Balancer
NGINX hoạt động như reverse proxy và load balancer ở tầng HTTP, tiếp nhận tất cả request từ client và Request Generator. Theo tài liệu, NGINX hỗ trợ cân bằng tải vòng tròn (round-robin) hoặc dựa trên IP-hash
`nginx.org`
. Trong cấu hình, các server backend được nhóm thành một upstream (app-1, app-2, app-3) và NGINX sẽ proxy request đến từng server trong nhóm theo thứ tự được thiết lập. NGINX cũng ghi log chi tiết từng request (phản hồi, thời gian, v.v.), tạo cơ sở cho việc phân tích và giám sát. Cấu hình NGINX có thể đặt thêm các header như `X-Real-IP` hoặc `Host` để truyền thông tin client đến server nội bộ.

### 2.2. Ứng dụng Node.js (app-1, app-2, app-3)
Ba container **Node.js** (app-1, app-2, app-3) nhận request được NGINX chuyển tiếp. Mỗi instance xử lý các yêu cầu HTTP và ghi lại số liệu về lưu lượng (RPS). Thông tin định danh server được cung cấp qua biến môi trường `NAME`, giúp đính kèm tag “server” khi ghi dữ liệu. Ứng dụng Node.js sử dụng thư viện client chính thức của InfluxDB để ghi dữ liệu thời gian-thực vào cơ sở dữ liệu `docs.influxdata.com`. Ví dụ, mỗi request có thể sinh ra một điểm đo (`Point`) trong InfluxDB với measurement như “rps”, tag là `NAME` của server, và trường (`field`) là giá trị đếm (thường tăng 1). Mỗi app Node.js chỉ ghi vào một instance InfluxDB riêng (ví dụ app-1 ghi vào influxdb-1, v.v.), hỗ trợ cơ chế phân tán dữ liệu.

### 2.3. Cụm InfluxDB (Sharding)
Hệ thống của dự án triển khai 3 node InfluxDB (influxdb-1, influxdb-2, influxdb-3), mỗi node tương ứng ghi nhận dữ liệu RPS của một app Node.js. Kiến trúc này sử dụng mô hình sharding để phân phối dữ liệu qua nhiều node. InfluxDB Enterprise (ví dụ phiên bản 1.x) cho phép các `shard group` chứa nhiều shard được phân tán giữa các node khác nhau, hỗ trợ tính toán truy vấn tổng hợp trên toàn cluster`docs.influxdata.com`. Nhờ vậy, ba cơ sở InfluxDB có thể được liên kết (qua Flux query hoặc liên kết nội bộ) để API có thể truy vấn cùng lúc và tổng hợp dữ liệu từ mọi node.

### 2.4. API tổng hợp dữ liệu
API là một service chịu trách nhiệm truy vấn dữ liệu từ ba instance InfluxDB và trả về kết quả cho Frontend. API sử dụng thư viện InfluxDB client hoặc HTTP API để gửi các truy vấn Flux/SQL đến từng database và kết hợp kết quả. InfluxDB JavaScript client trong Node.js cho phép xây dựng câu lệnh Flux để lấy số liệu từ bucket
docs.influxdata.com
. Ví dụ, API có thể gửi truy vấn tính tổng request trên mỗi giây hoặc trung bình RPS trong khoảng thời gian. Sau khi nhận dữ liệu từ các InfluxDB, API tổng hợp thành các tập dữ liệu mong muốn (như RPS trung bình, max RPS, biểu đồ thời gian) và trả về cho frontend dưới dạng JSON.
### 2.5. Giao diện người dùng (Frontend)
Frontend (View) là ứng dụng web hiển thị dữ liệu trực quan (biểu đồ, bảng số liệu) về RPS. Nó lấy dữ liệu từ API thông qua các endpoint REST/JSON và vẽ biểu đồ thời gian-thực (cập nhật liên tục) bằng thư viện đồ thị như Grafana, Chart.js, D3.js, v.v. Giao diện cho phép người dùng quan sát xu hướng tăng giảm của lưu lượng theo giây, cũng như các giá trị tổng hoặc trung bình. Frontend giúp hình dung trực quan số liệu RPS để dễ dàng theo dõi và phân tích hiệu năng hệ thống.

### 2.6. Bộ sinh Request (Request Generator)
Request Generator là thành phần giả lập hành vi người dùng để tạo tải thử nghiệm. Đây có thể là một dịch vụ hoặc script (ví dụ sử dụng Apache JMeter, k6, Locust, hoặc đơn giản là một chương trình tùy chỉnh) liên tục gửi các request HTTP đến NGINX với tần suất (hoặc pattern) xác định. Chúng có thể gửi request ngẫu nhiên hoặc tuần tự để kiểm tra khả năng chịu tải, đo lường RPS, và đảm bảo hệ thống vận hành ổn định dưới tải cao. Request Generator tách biệt với các thành phần chính để tránh can thiệp vào hoạt động sản xuất, và giúp mô phỏng chính xác lưu lượng thực tế.

## 3. Công nghệ và thư viện sử dụng

| Thành phần           | Công nghệ / Thư viện        | Lý do chọn                                                                 |
|----------------------|-----------------------------|----------------------------------------------------------------------------|
| Backend              | Node.js + Express           | Xử lý bất đồng bộ hiệu quả, cộng đồng lớn, dễ mở rộng                     |
| Load Balancer        | NGINX                       | Ổn định, cấu hình dễ, hỗ trợ cân bằng tải và reverse proxy                |
| Database             | InfluxDB                    | Tối ưu lưu trữ dữ liệu time-series, truy vấn nhanh, hỗ trợ sharding và replication |
| Frontend             | React                       | Tương tác linh hoạt, dễ phát triển dashboard                              |
| Containerization     | Docker + Docker Compose     | Đóng gói môi trường, triển khai đồng nhất và dễ dàng mở rộng             |
| Testing & Load       | Request Generator (custom)  | Tạo tải giả lập, kiểm tra khả năng chịu tải và tìm điểm nghẽn             |

## 4. Mô hình dữ liệu (Database Model)

Dữ liệu được lưu trữ theo mô hình time-series trong InfluxDB với các thành phần:

- **Measurement**: `request_logs` — bảng chứa log các request.
- **Tags (index)**: 
  - `app_name`: tên ứng dụng xử lý request,
  - `status_code`: mã HTTP trả về,
  - `endpoint`: API được gọi.
- **Fields (dữ liệu thực)**:
  - `response_time`: thời gian phản hồi,
  - `request_id`: mã định danh request.
- **Timestamp**: thời gian ghi nhận log (định dạng ISO 8601 hoặc Unix timestamp).

**Phân tán và đồng bộ dữ liệu**:

- Mỗi node InfluxDB chứa dữ liệu của một instance app riêng, giảm tải ghi dữ liệu.
- Các node đồng bộ dữ liệu với nhau qua replication, đảm bảo dữ liệu nhất quán và tăng khả năng phục hồi khi có sự cố.

## 5. Chiến lược triển khai và cấu hình hệ thống

- Hệ thống được triển khai bằng Docker và Docker Compose để đảm bảo tính nhất quán và dễ dàng mở rộng.
- Mỗi thành phần (App Web, InfluxDB, NGINX, API Service, Request Generator) được đóng gói trong container riêng biệt.
- Docker Compose quản lý và kết nối các container qua mạng nội bộ.
- NGINX được cấu hình làm Load Balancer, điều phối request đến các app backend.
- Các node InfluxDB cấu hình replication để đồng bộ dữ liệu.
- Hệ thống có thể mở rộng bằng cách tăng số lượng container App Web và InfluxDB, đồng thời cập nhật cấu hình NGINX.
- Dự kiến tích hợp Prometheus và Grafana để giám sát hệ thống và cảnh báo lỗi.
