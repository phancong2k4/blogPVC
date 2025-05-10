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

# Đề xuất đề tài: Giám Sát Thời Tiết Đơn Giản với InfluxDB

## 1. Giới thiệu
Trong bối cảnh biến đổi khí hậu và thời tiết thay đổi nhanh chóng, việc theo dõi và phân tích dữ liệu thời tiết trở nên quan trọng. Đề tài này tập trung vào việc xây dựng hệ thống giám sát thời tiết đơn giản sử dụng InfluxDB để lưu trữ dữ liệu thời tiết theo thời gian và Grafana để trực quan hóa thông tin.

## 2. Vấn đề cần giải quyết
Thời tiết là yếu tố ảnh hưởng trực tiếp đến cuộc sống hàng ngày, từ việc lựa chọn trang phục, kế hoạch hoạt động ngoài trời đến các quyết định nông nghiệp. Tuy nhiên, việc theo dõi và phân tích xu hướng thời tiết thường chỉ có trên các nền tảng lớn, và người dùng cá nhân không có hệ thống đơn giản để tự mình theo dõi dữ liệu thời tiết trong khu vực của họ. Đề tài này nhằm xây dựng một hệ thống thu thập và lưu trữ dữ liệu thời tiết từ API thời tiết, giúp người dùng theo dõi và phân tích xu hướng thời tiết theo ngày, tuần.

## 3. Mục tiêu
- Thu thập dữ liệu thời tiết (nhiệt độ, độ ẩm) từ API thời tiết.
- Lưu trữ dữ liệu thời gian thực vào cơ sở dữ liệu InfluxDB.
- Phân tích và theo dõi xu hướng thời tiết theo ngày, tuần.
- Trực quan hóa dữ liệu thời tiết bằng Grafana để dễ dàng theo dõi.

## 4. Phạm vi
- Thu thập dữ liệu từ API thời tiết (như OpenWeather API, WeatherAPI).
- Lưu trữ thông tin thời tiết như nhiệt độ, độ ẩm, thời gian.
- Xây dựng giao diện trực quan hóa trên Grafana.
- Hệ thống đơn giản, phục vụ mục đích học tập và nghiên cứu.

## 5. Công nghệ sử dụng
- **InfluxDB:** Lưu trữ dữ liệu chuỗi thời gian (time-series data).
- **Python:** Thu thập dữ liệu từ API thời tiết, xử lý và lưu vào InfluxDB.
- **Grafana:** Trực quan hóa dữ liệu thời tiết từ InfluxDB.

## 6. Kỳ vọng kết quả
- Hệ thống có thể thu thập và lưu trữ dữ liệu thời tiết liên tục.
- Giao diện trực quan trên Grafana thể hiện xu hướng thời tiết (nhiệt độ, độ ẩm) theo ngày, tuần.
- Người dùng có thể dễ dàng theo dõi và phân tích thông tin thời tiết.

## 7. Ý nghĩa
Đề tài giúp sinh viên hiểu cách thu thập, lưu trữ và phân tích dữ liệu thời gian thực. Ngoài ra, còn giúp phát triển kỹ năng sử dụng InfluxDB và Grafana trong các ứng dụng thực tế.
