---
title: "Deliverable 1 - Website"
date: "2025-05-11"
updated: "2025-05-11"
categories:
  - "sveltekit"
  - "markdown"
  - "svelte"
excerpt: Đề xuất đề tài và mô tả vấn đề
---
# Báo cáo InfluxDB

## 1. Mục đích của thư viện InfluxDB
InfluxDB là một cơ sở dữ liệu chuỗi thời gian mã nguồn mở, được thiết kế để lưu trữ, quản lý và phân tích dữ liệu có dấu thời gian. Thư viện này chủ yếu được sử dụng để xử lý các loại dữ liệu như log hệ thống, dữ liệu cảm biến từ IoT, và chỉ số hiệu suất của các ứng dụng.

## 2. Vấn đề mà thư viện giải quyết
InfluxDB giải quyết bài toán lưu trữ và xử lý dữ liệu thời gian thực với hiệu suất cao. Cụ thể, nó cho phép người dùng ghi lại và phân tích một lượng lớn dữ liệu có dấu thời gian một cách nhanh chóng. Ngoài ra, thư viện này còn hỗ trợ các công cụ như Flux để thực hiện truy vấn và phân tích dữ liệu một cách linh hoạt.

## 3. Điểm mạnh của thư viện
- **Hiệu suất cao:** InfluxDB có khả năng ghi hàng triệu điểm dữ liệu mỗi giây mà không ảnh hưởng đến tốc độ truy vấn.
- **Giao diện trực quan:** Người dùng có thể dễ dàng tương tác với dữ liệu thông qua giao diện người dùng hoặc API RESTful.
- **Hỗ trợ truy vấn linh hoạt:** Ngoài ngôn ngữ truy vấn InfluxQL truyền thống, InfluxDB còn hỗ trợ Flux, một ngôn ngữ mạnh mẽ hơn để xử lý dữ liệu.
- **Dễ dàng tích hợp:** InfluxDB dễ dàng kết nối với các công cụ giám sát và phân tích như Grafana.

## 4. Điểm yếu của thư viện
- **Hạn chế trong xử lý dữ liệu phi thời gian:** Thư viện này chỉ thực sự hiệu quả với dữ liệu có dấu thời gian, còn với dữ liệu quan hệ phức tạp, nó không tối ưu.
- **Yêu cầu tài nguyên cao khi xử lý dữ liệu lớn:** Đối với dữ liệu có độ phân giải cao (như mili giây), hiệu suất có thể bị giảm sút.
- **Phiên bản mã nguồn mở bị giới hạn tính năng:** Một số tính năng nâng cao như quản lý người dùng và phân quyền chỉ có trong phiên bản thương mại.

## 5. So sánh với các thư viện framework khác
| Tiêu chí             | InfluxDB                       | TimescaleDB                | Prometheus                |
|----------------------|----------------------------------|-----------------------------|----------------------------|
| Loại cơ sở dữ liệu    | NoSQL, chuỗi thời gian         | SQL (dựa trên PostgreSQL)  | NoSQL, chuỗi thời gian     |
| Khả năng ghi dữ liệu  | Cao                            | Trung bình (tùy cấu hình)  | Cao                         |
| Truy vấn              | InfluxQL, Flux                 | SQL                         | Ngôn ngữ truy vấn riêng    |
| Hỗ trợ dữ liệu phức tạp| Hạn chế                        | Tốt                         | Hạn chế                     |
| Tích hợp              | Tốt (Grafana, Telegraf)        | Tốt (PostgreSQL)           | Tốt (Grafana)               |

## 6. Ứng dụng của thư viện
- **Giám sát hệ thống:** InfluxDB có thể được sử dụng để theo dõi hiệu suất của các máy chủ, hệ thống mạng hoặc ứng dụng phần mềm.
- **IoT và dữ liệu cảm biến:** Lưu trữ và phân tích dữ liệu từ cảm biến như nhiệt độ, độ ẩm, ánh sáng.
- **Phân tích dữ liệu tài chính:** Theo dõi giá cổ phiếu, biến động tỷ giá theo thời gian.
- **Quản lý dữ liệu thời gian thực:** Phân tích dữ liệu thời gian thực trong các hệ thống quản lý như DevOps hoặc hệ thống kiểm soát công nghiệp.

## 7. Kết luận
InfluxDB là một công cụ mạnh mẽ và linh hoạt cho việc quản lý và phân tích dữ liệu chuỗi thời gian. Tuy nhiên, người dùng cần cân nhắc các hạn chế của nó, đặc biệt khi xử lý dữ liệu phi thời gian hoặc dữ liệu quan hệ phức tạp.

# Đề xuất đề tài: Giám sát lượng Request của trang Web 

## 1. Giới thiệu
Trong ứng dụng web microservices (API, Request Handler, Nginx), biết mỗi giây có bao nhiêu request, endpoint nào “hot” nhất, và khi nào hệ thống ngắc ngoải là cực kỳ cần thiết. Thay vì “đào log” thủ công, ta xây dựng hệ thống giám sát hoàn chỉnh chạy trên Docker, dùng InfluxDB lưu metric, Grafana vẽ biểu đồ—tất cả không cần code Python.

## 2. Vấn đề cần giải quyết
- **Log phân tán:**  
  Nginx, API, Request Handler có log riêng lẻ, khó tổng hợp real-time.
- **Chưa có time-series DB:**  
  Không lưu trữ lịch sử request dưới dạng chuỗi thời gian để phân tích xu hướng (RPS, RPM).
- **Thiếu trực quan:**  
  Dev/DevOps không có dashboard chung, thường “nhảy container – grep log”, mất thời gian.
- **Không cảnh báo:**  
  Khi traffic vượt ngưỡng (ví dụ RPS > 500), không tự động thông báo để scale hay điều chỉnh.

## 3. Mục tiêu
- Thu thập metric request từ Nginx (stub_status) và API (Prometheus endpoint) thông qua Telegraf trên Docker.
- Lưu trữ time-series vào InfluxDB (độ phân giải 10s/1m).
- Trực quan hóa bằng Grafana:
  - Biểu đồ RPS tổng (Nginx)
  - Biểu đồ RPM từng endpoint (`/login`, `/products`, …)
  - Top 5 endpoint “hot”
- Alerting: Khi RPS hoặc RPM vượt ngưỡng → gửi email/Slack.

## 4. Phạm vi
- **Thu thập:**
  - Nginx: bật `stub_status` → Telegraf pull metric.
  - API Service (Node.js/Express hoặc Flask): expose `/metrics` chuẩn Prometheus → Telegraf pull.
- **Lưu trữ:**  
  InfluxDB (container), retention policy 30 ngày.
- **Trực quan:**  
  Grafana (container) dùng data source InfluxDB.
- **Alert:**  
  Grafana Alerting (email/Slack).
- **Không bao gồm:**  
  Giám sát CPU/RAM, auto-scaling.

## 5. Công nghệ sử dụng
- **Docker & Docker Compose:** Quản lý đồng thời các container:
  1. Nginx Reverse Proxy  
  2. API Service (Node.js/Express – expose `/metrics`)  
  3. InfluxDB OSS  
  4. Telegraf (inputs: `nginx`, `prometheus`; output: InfluxDB)  
  5. Grafana OSS
- **Telegraf:** Thu thập metric Nginx và API, đẩy vào InfluxDB.
- **InfluxDB:** Lưu trữ metric time-series (measurement: `web_requests`, tags: `service`, `endpoint`, fields: `count`/`request_rate`).
- **Grafana:** Tạo dashboard (RPS, RPM, top endpoint), cấu hình alert.

## 6. Kỳ vọng kết quả
- **Data flow tự động:**  
  Telegraf pull mỗi 10–15s, InfluxDB lưu, Grafana hiển thị.
- **Dashboard trực quan:**  
  - RPS tổng (Nginx)  
  - RPM cho `/login`, `/register`, `/products`, `/orders`  
  - Top 5 endpoint “hot”
- **Alerting:** Gửi email/Slack khi:
  - RPS tổng > 500 (1 phút)
  - RPM `/login` > 300 (5 phút)
- **Lịch sử 30 ngày:**  
  Retention policy để phân tích xu hướng hàng tuần/tháng.
