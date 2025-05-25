---
title: "Deliverable 2"
date: "2025-05-05"
updated: "2025-05-05"
categories:
  - "sveltekit"
  - "markdown"
excerpt: Deliverable 2
---
Deliverable 2

## 1. Bản vẽ kiến trúc hệ thống

Hệ thống được xây dựng dựa trên kiến trúc phân tán gồm nhiều thành phần chính như Client (người dùng cuối), Load Balancer (NGINX), các instance ứng dụng backend (Node.js), cơ sở dữ liệu phân tán (InfluxDB), dịch vụ API trung gian và giao diện Dashboard. Mô hình kiến trúc này giúp hệ thống có khả năng mở rộng, chịu tải cao và đảm bảo tính sẵn sàng.

Client gửi các yêu cầu HTTP đến Load Balancer NGINX, NGINX sẽ phân phối các yêu cầu này đến các instance ứng dụng Node.js theo thuật toán cân bằng tải. Mỗi instance sẽ ghi dữ liệu log request vào một node InfluxDB riêng biệt. Các node InfluxDB có thể đồng bộ dữ liệu với nhau để tăng tính nhất quán. Dashboard lấy dữ liệu thông qua API Service, truy vấn và tổng hợp dữ liệu từ các node InfluxDB, rồi trả kết quả về cho người dùng.

![sơ đồ trình tự ](/images/Sodotrinhtu.png)
Sơ đồ trình tự hệ thống


## 2. Mô tả chi tiết các thành phần trong hệ thống

### 2.1. Client
Client là điểm cuối giao tiếp với hệ thống, có thể là trình duyệt web hoặc ứng dụng di động. Client gửi các yêu cầu dịch vụ và nhận dữ liệu kết quả, đồng thời hiển thị các báo cáo, biểu đồ trên giao diện Dashboard.

### 2.2. Request Generator
Request Generator là thành phần mô phỏng tải bằng cách tự động gửi các yêu cầu HTTP đến Load Balancer. Mục đích là kiểm thử hiệu suất, khả năng chịu tải của hệ thống và phát hiện điểm nghẽn.

### 2.3. NGINX Load Balancer
NGINX đảm nhận vai trò phân phối tải đến các instance backend. NGINX sử dụng thuật toán Round-Robin hoặc Least Connections để cân bằng số lượng request giữa các server, giúp tránh tình trạng quá tải tại một node và tăng hiệu quả xử lý đồng thời.

### 2.4. App Web (Node.js)
Ứng dụng backend được phát triển trên nền Node.js, có khả năng xử lý bất đồng bộ giúp phục vụ nhiều request đồng thời hiệu quả. Mỗi instance nhận request từ NGINX, xử lý nghiệp vụ và ghi log dữ liệu tương ứng vào node InfluxDB riêng biệt.

### 2.5. InfluxDB
InfluxDB là hệ quản trị cơ sở dữ liệu chuyên biệt cho dữ liệu time-series, tối ưu cho việc lưu trữ và truy vấn các log theo thời gian. Mỗi instance ứng dụng sẽ ghi dữ liệu vào một node InfluxDB riêng, giúp phân tán tải ghi (sharding). Các node có thể đồng bộ dữ liệu với nhau qua cơ chế replication nhằm đảm bảo dữ liệu luôn nhất quán và phục vụ truy vấn tổng hợp.

### 2.6. API Service
API Service là lớp trung gian nhận yêu cầu từ Dashboard, truy vấn song song dữ liệu từ các node InfluxDB khác nhau, tổng hợp và trả về kết quả cho Dashboard. Cách thức này giảm thời gian chờ và tăng hiệu quả truy vấn dữ liệu tổng hợp.

### 2.7. View Dashboard
Dashboard hiển thị các số liệu, biểu đồ phân tích dữ liệu thời gian thực, cung cấp cái nhìn trực quan cho người dùng. Dashboard tương tác với API Service qua các REST API để cập nhật dữ liệu liên tục.

## 3. Công nghệ và thư viện sử dụng

| Thành phần           | Công nghệ / Thư viện        | Lý do chọn                                                                 |
|----------------------|-----------------------------|----------------------------------------------------------------------------|
| Backend              | Node.js + Express           | Xử lý bất đồng bộ hiệu quả, cộng đồng lớn, dễ mở rộng                     |
| Load Balancer        | NGINX                       | Ổn định, cấu hình dễ, hỗ trợ cân bằng tải và reverse proxy                |
| Database             | InfluxDB                    | Tối ưu lưu trữ dữ liệu time-series, truy vấn nhanh, hỗ trợ sharding và replication |
| Frontend             | React hoặc Vanilla JS       | Tương tác linh hoạt, dễ phát triển dashboard                              |
| Containerization     | Docker + Docker Compose     | Đóng gói môi trường, triển khai đồng nhất và dễ dàng mở rộng             |
| Testing & Load       | Request Generator (custom)  | Tạo tải giả lập, kiểm tra khả năng chịu tải và tìm điểm nghẽn             |
| Bảo mật              | Helmet (Node.js middleware) | Tăng cường bảo mật HTTP headers                                           |
| Giám sát (dự kiến)   | Prometheus + Grafana        | Giám sát hiệu năng, cảnh báo sớm hệ thống                                |

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
