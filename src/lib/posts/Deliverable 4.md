---
title: "Deliverable 4"
date: "2025-06-01"
updated: "2025-06-01"
categories:
  - "sveltekit"
  - "markdown"
excerpt: Deliverable 4
---

Lê Đưc Long - 23010016 - 23010016@st.phenikaa-uni.edu.vn
Phan Văn Công -	22010158 - 22010158@st.phenikaa-uni.edu.vn

# MỤC LỤC

I. Tổng Quan Dự Án

II. Thiết Kế Kiến Trúc
1. Mô Hình Kiến Trúc
2. Công Nghệ Sử Dụng

III. Phân Tích Chi Tiết Các Thành Phần
1. Request
2. NGINX Load Balancer
3. Ứng dụng App-1/2/3
4. InfluxDB-1/2/3
5. API
6. View

IV. Các Sơ Đồ Chính Của Hệ Thống
1. Sơ đồ Kiến trúc Hệ thống
2. Sơ đồ Triển khai
3. Các yêu cầu chức năng
   3.1. Sơ đồ Use-case
   3.3. Đặc tả các Use-case

V. Phân Tích Vấn Đề và Các Rủi Ro Tiềm Ẩn
1. Vấn Đề Hiện Tại
2. Rủi Ro Tiềm Ẩn

VI. ĐỀ XUẤT GIẢI PHÁP VÀ HƯỚNG PHÁT TRIỂN
1. Giải Quyết Các Vấn Đề Hiện Tại
2. Cải Thiện và Nâng Cấp
3. Hướng Phát Triển Tương Lai

VII. ĐÁNH GIÁ THEO TIÊU CHÍ KỸ THUẬT VÀ ĐỀ XUẤT

KẾT LUẬN



# I. Tổng Quan Dự Án

Hệ thống web hiện đại thường được triển khai theo mô hình phân tán, nhằm mục tiêu mở rộng (scalability), tăng khả năng chịu lỗi (fault tolerance) và đáp ứng lượng người dùng lớn. Một vấn đề đặt ra trong các hệ thống như vậy là giám sát lưu lượng truy cập theo thời gian thực, đặc biệt là trong môi trường có cân bằng tải (load balancing), nơi mỗi máy chủ backend chỉ biết đến một phần lưu lượng.

Đề tài này đề xuất xây dựng một hệ thống giám sát lưu lượng request theo mỗi giây (requests per second – RPS) sử dụng:
- **InfluxDB**: cơ sở dữ liệu chuỗi thời gian (time-series database) để ghi nhận và phân tích dữ liệu request.
- **NGINX**: đóng vai trò cân bằng tải và có khả năng ghi log request.
- **Docker Compose**: để triển khai toàn bộ hệ thống theo kiến trúc phân tán.
- **Node.js backend** (hoặc ứng dụng web): nơi tạo ra request hoặc thu thập từ log.

# II. Thiết Kế Kiến Trúc

## 1. Mô Hình Kiến Trúc

Hệ thống được triển khai dưới dạng microservices với các thành phần chính sau, đóng gói trong Docker Container và điều phối bởi Docker Compose:
![mohinhkientruc](/images/sodokientruc.drawio%20(1).png)
# I. Tổng Quan Dự Án

Hệ thống web hiện đại thường được triển khai theo mô hình phân tán, nhằm mục tiêu mở rộng (scalability), tăng khả năng chịu lỗi (fault tolerance) và đáp ứng lượng người dùng lớn. Một vấn đề đặt ra trong các hệ thống như vậy là giám sát lưu lượng truy cập theo thời gian thực, đặc biệt là trong môi trường có cân bằng tải (load balancing), nơi mỗi máy chủ backend chỉ biết đến một phần lưu lượng.

Đề tài này đề xuất xây dựng một hệ thống giám sát lưu lượng request theo mỗi giây (requests per second – RPS) sử dụng:
- **InfluxDB**: cơ sở dữ liệu chuỗi thời gian (time-series database) để ghi nhận và phân tích dữ liệu request.
- **NGINX**: đóng vai trò cân bằng tải và có khả năng ghi log request.
- **Docker Compose**: để triển khai toàn bộ hệ thống theo kiến trúc phân tán.
- **Node.js backend** (hoặc ứng dụng web): nơi tạo ra request hoặc thu thập từ log.

# II. Thiết Kế Kiến Trúc

## 1. Mô Hình Kiến Trúc

Hệ thống được triển khai dưới dạng microservices với các thành phần chính sau, đóng gói trong Docker Container và điều phối bởi Docker Compose:
- Xử lý và ghi nhận request (các node App-1, App-2, App-3)
- Lưu trữ time-series (cụm InfluxDB gồm 3 node)
- Tổng hợp và hiển thị dữ liệu thống kê (API Aggregator + Frontend UI)
- Scale ngang (Horizontal Scaling):
  - Khi lượng request tăng, hệ thống chỉ cần mở thêm các node App (ví dụ App-4, App-5…) và thêm vào upstream của NGINX.
  - Tương tự, cụm InfluxDB có thể mở rộng bằng cách tăng số node replica/shard để vừa chịu tải ghi, vừa duy trì độ sẵn sàng.
  - NGINX Load Balancer chịu trách nhiệm phân phối request đồng đều cho các node App. Nếu một node App “chết”, NGINX tự động đẩy request đến node còn lại.
  - InfluxDB Cluster (InfluxDB-1, InfluxDB-2, InfluxDB-3) hoạt động ở chế độ replication/sharding, đảm bảo không single-point-of-failure khi lưu trữ dữ liệu.

### Kênh luồng dữ liệu (Data Flow)

1. **Request Generator**: Có thể là browser thực (người dùng) hoặc công cụ stress-test (JMeter, Locust) gửi hàng loạt HTTP request vào hệ thống.  
2. **NGINX Load Balancer**: Tiếp nhận tất cả request đầu vào và cân bằng tải (Round-Robin hoặc Least-Conn) qua các node App.  
3. **App-1 / App-2 / App-3**: Mỗi node nhận request, thực hiện logic nghiệp vụ (validate, enrich, v.v.) rồi ghi vào InfluxDB cluster.  
4. **InfluxDB-1 / InfluxDB-2 / InfluxDB-3**: Ba node này chia nhau lưu trữ dữ liệu time-series (lần lượt theo cơ chế replication hoặc sharding) để đảm bảo chịu được lượng ghi lớn và luôn có dữ liệu sẵn sàng đọc.  
5. **API Aggregator**: Khi Frontend UI cần hiển thị thống kê, Aggregator đồng loạt query vào cả ba node InfluxDB, ghép kết quả thành một tập dữ liệu thống nhất (sum, avg, max, v.v.) rồi trả về cho giao diện.  
6. **Frontend UI**: Nhận dữ liệu đã tổng hợp từ API Aggregator, thực hiện render dashboard (biểu đồ, bảng số liệu, v.v.) để người quản trị dễ dàng theo dõi lưu lượng, xu hướng request theo thời gian thực.

## 2. Công Nghệ Sử Dụng

- **Backend**:  
  - Node.js/Express (REST API, Aggregator, Dashboard Service)  
  - Python/Flask (Request Service)  

- **Frontend**:  
  - EJS + HTML/CSS (render dashboard)  
  - Chart.js + Fetch API (vẽ biểu đồ, gọi AJAX)  

- **Database**:  
  - InfluxDB 2.x (3 node cluster) (time-series storage)  

- **Triển khai**:  
  - Docker & Docker Compose (containerized stack)  
  - NGINX (reverse proxy & load balancer)  

- **Build Tools**:  
  - npm (Node.js)  
  - pip (Python)  

# III. Phân Tích Chi Tiết Các Thành Phần

## 1. Request

**Chức năng:**  
Thành phần “Request Generator” (nằm trong thư mục `request/`) chịu trách nhiệm sinh tải (load) lên toàn bộ hệ thống thông qua NGINX Load Balancer. Khi được khởi động, script `request.js` sẽ liên tục tạo ra các HTTP request đến endpoint của ứng dụng (ví dụ: `http://nginx-lb:80/api/v1/task`). Mục đích chính là kiểm thử khả năng xử lý đồng thời, độ ổn định và độ trễ của hệ thống phân tán khi có nhiều client ảo cùng truy cập. Ở cùng thư mục còn có file `server.js` dùng để “fake backend” (giả lập dịch vụ xử lý) nhằm mục đích test end-to-end nếu cần, nhưng phần sinh tải thực sự được thực hiện bởi `request.js`.

**Công nghệ:**
- **Node.js:** Toàn bộ logic sinh tải và server giả lập được viết bằng JavaScript chạy trên Node.js.
- **Axios:** Thư viện HTTP client để gửi các GET request bất đồng bộ.
- **Express:** (trong `server.js`) được dùng để khởi một HTTP server đơn giản giả lập API backend khi cần.
- **Docker:** Cho phép đóng gói thành một image độc lập. Khi build thành công, chỉ cần chạy container để tự động start cả `server.js` (nếu cấu hình) và/hoặc `request.js`.
- **package.json:** Khai báo các script (ví dụ `"start": "node request.js"`, `"serve": "node server.js"`).

**Thay đổi URL và số lượng request:**
1. Mở file `request/request.js`.  
2. Thay giá trị `url` (khi gọi hàm) thành địa chỉ Load Balancer hoặc API cụ thể bạn muốn test.  
3. Truyền `rn` tương ứng với số request đồng thời (ví dụ 50, 100, 500…) để xác định độ mạnh tải.

**Mục đích:**  
Module này nhằm kiểm tra sơ bộ khả năng chịu tải (stress test) của NGINX Load Balancer và các service backend phía sau. Bằng cách gộp nhiều lời gọi `axios.get` vào một mảng `promises` rồi đợi cùng lúc với `Promise.all`, ta có thể “dội” cùng lúc hàng trăm (hoặc nhiều hơn) request lên server để xem ứng xử khi tải đồng thời cao.

---

## 2. NGINX Load Balancer

**Chức năng:**  
NGINX đóng vai trò là load balancer và reverse proxy, đứng trước ba service App-1, App-2, App-3. Khi có bất kỳ yêu cầu HTTP nào (từ Request Generator hoặc người dùng thật), NGINX sẽ nhận request đó và phân phối đều (theo thuật toán round‐robin mặc định) đến một trong ba backend server là `app-1:3001`, `app-2:3002` hoặc `app-3:3003`. Nhờ vậy, NGINX giúp cân bằng tải, giảm độ trễ và tránh tình trạng một instance ứng dụng bị quá tải.

**Công nghệ:**
- **Phần mềm NGINX** (viết bằng C): Ở đây sử dụng bản Open Source để làm load balancer; nếu cần thêm tính năng health check động hoặc dashboard theo dõi, có thể cân nhắc NGINX Plus.
- **Cấu hình dưới Docker:** Toàn bộ NGINX được đóng gói trong container, sử dụng `nginx.conf` do dự án cung cấp, rồi mount file cấu hình vào `/etc/nginx/nginx.conf` trong container.

**Directive cơ bản trong `nginx.conf`:**
1. **Khối upstream**  
   ```js
   upstream backend {
       server app-1:3001 max_fails=2 fail_timeout=5s;
       server app-2:3002 max_fails=2 fail_timeout=5s;
       server app-3:3003 max_fails=2 fail_timeout=5s;
   }
    ```
- server app-1:3001: hostname hoặc container name `app-1` trên mạng Docker, port `3001` là cổng mà service App-1 đang lắng nghe. Tương tự cho App-2, App-3.  
- max_fails=2 fail_timeout=5s: nếu backend không phản hồi hoặc trả lỗi (timeout hoặc status code ≥ 500) hai lần liên tiếp trong vòng 5 giây, NGINX tạm thời sẽ đánh dấu là “unhealthy” và ngừng gửi request đến backend đó trong 5 giây.
