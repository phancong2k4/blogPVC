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
2. **Phần server block**

```js
server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
- **listen 80**: NGINX lắng nghe cổng HTTP 80.  
- **server_name localhost**: Chỉ định hostname. Khi đăng ký tên miền production, ta có thể đổi `localhost` thành tên miền thực (ví dụ `myapp.example.com`).  
- **location /**: Mọi request path (ví dụ `/api/v1/...`, `/login`, `/static/...`) đều được chuyển tiếp (`proxy_pass`) đến upstream `backend`.  
- **proxy_set_header**: Giữ lại thông tin gốc của client, cần để backend biết được IP client thực tế, protocol (HTTP/HTTPS), v.v.  
### 3. Logging và Tối ưu

- **access_log /var/log/nginx/access.log main;**  
  Định dạng log theo format tên `main`, ghi ra đường dẫn, status code, user-agent, time stamp… Hữu ích khi debug hoặc phân tích traffic.  
- **sendfile on, keepalive_timeout 65;**  
  Tối ưu việc mở kết nối TCP, giảm overhead khi gửi tệp tĩnh hoặc giữ kết nối lâu hơn.  
- Nếu cần gzip tĩnh hoặc health check động (với NGINX Plus), ta có thể bật thêm directive như `gzip on;`, `health_check;` trong `upstream`.

---

#### Vai trò & Phân tích

- **Cân bằng tải & ẩn cấu trúc backend:**  
  Nhờ NGINX, client chỉ cần truy cập vào một địa chỉ duy nhất (`http://<NGINX_HOST>:80`). Mọi request được phân phối đều đặn tới các instance App-1/2/3. Điều này vừa giúp tăng khả năng chịu đựng tải (scale-out) vừa giấu IP thật, port của các service riêng lẻ.  

- **Hiệu suất cao, độ ổn định:**  
  NGINX viết bằng C, nổi tiếng xử lý hàng chục nghìn kết nối đồng thời mà tiêu tốn tài nguyên rất thấp. Việc sử dụng `upstream` và `keepalive` giúp tận dụng kết nối TCP tái sử dụng, tiết kiệm thời gian khởi handshake.  

- **Khả năng mở rộng:**  
  - Muốn tăng công suất, chỉ cần thêm entry `server app-4:3004;` vào `upstream backend` và khởi thêm container App-4. NGINX sẽ tự động gộp vào pool.  
  - Để đảm bảo high availability (HA), có thể chạy nhiều instance NGINX song song (Active-Active) phía trước, đặt một load balancer cứng (hardware LB) hoặc DNS round-robin để điều phối.  
  - Nếu dùng NGINX Plus, có thêm API để dynamic reload `upstream`, health check nâng cao, dashboard giám sát.  

---

## 3. Ứng dụng App-1/2/3

**Chức năng:**  
Ba ứng dụng App-1, App-2, App-3 (thường cùng một codebase nhưng chạy trên các port khác nhau) là tầng xử lý nghiệp vụ (business logic) chính, tiếp nhận mọi request từ NGINX Load Balancer. Mỗi App có nhiệm vụ:  
- Tiếp nhận request HTTP (GET, POST, v.v.) từ NGINX, xử lý logic ứng dụng (tính toán, truy vấn, ghi dữ liệu, xử lý form…).  
- Ghi/chuyển tiếp chỉ số (metrics) đo được (ví dụ: thời gian xử lý, số lượng requests, error counts) vào InfluxDB tương ứng để phục vụ việc giám sát và theo dõi hiệu năng.  
- Trả về response (JSON, HTML hay static file) cho NGINX, để NGINX gói lại và trả về client.  

Ba instance App-1/2/3 có thể giống nhau về code (đều là cùng một service chạy nhiều bản) hoặc mỗi app thực hiện một tính năng riêng (ví dụ App-1 chuyên xử lý tính toán, App-2 chuyên CRUD dữ liệu, App-3 chuyên publish/subscribe). Tuy nhiên, trong hầu hết kiến trúc mẫu, ba ứng dụng này là ba bản “service” giống nhau, nhằm chia đều tải và tăng khả năng chịu lỗi (nếu một instance bị down, hai instance còn lại vẫn phục vụ request).

**Công nghệ:**  
- **Node.js + Express.js (JavaScript):** Giả sử code sample như sau trong mỗi App:

```js
// app-1/index.js (tương tự với app-2, app-3)
const express = require("express");
const { InfluxDB } = require("@influxdata/influxdb-client");
const app = express();
const port = process.env.PORT || 3001;

// Cấu hình InfluxDB client qua biến môi trường
const influxUrl = process.env.INFLUX_URL;
const influxToken = process.env.INFLUX_TOKEN;
const org = process.env.INFLUX_ORG;
const bucket = process.env.INFLUX_BUCKET;
const writeApi = new InfluxDB({ url: influxUrl, token: influxToken })
                    .getWriteApi(org, bucket);

app.use(express.json());

app.get("/api/v1/task", async (req, res) => {
  const startTime = Date.now();
  
  // Ví dụ: giả lập xử lý nghiệp vụ
  // … (tính toán, truy vấn DB, v.v.)
  
  const processingTime = Date.now() - startTime;
  // Ghi metrics về InfluxDB
  writeApi.writePoint({
    measurement: "request_metrics",
    tags: { app: process.env.APP_ID || "app-1" },
    fields: { duration_ms: processingTime, status: 200 },
    timestamp: new Date(),
  });
  await writeApi.flush();
  
  res.json({ 
    appId: process.env.APP_ID || "app-1",
    message: "Task completed",
    duration: processingTime 
  });
});

app.listen(port, () => {
  console.log(`App-${process.env.APP_ID} listening on port ${port}`);
});
```
- Mỗi instance App-X có một biến môi trường `APP_ID` (ví dụ `1`, `2`, `3`) để gắn tag khi ghi metrics vào InfluxDB.
- InfluxDB client: sử dụng thư viện chính thức `@influxdata/influxdb-client` (Node.js) để ghi dòng dữ liệu mỗi khi xử lý xong request, phục vụ monitoring.
- Cấu hình port (3001, 3002, 3003) và URL khả dụng InfluxDB truyền qua biến môi trường `INFLUX_URL`, `INFLUX_TOKEN`, v.v.

- **Database Time-series:**  
  Dùng InfluxDB để lưu metrics. Mỗi App có thể:
  - Dùng cùng một InfluxDB cluster  
  - Hoặc tách riêng ra nhiều database/bucket/topic.
# Cấu hình cơ bản

## 1. Port lắng nghe
- **App-1:** `PORT=3001`
- **App-2:** `PORT=3002`
- **App-3:** `PORT=3003`

> Cần bảo đảm không trùng lặp port và khớp với upstream định nghĩa trong NGINX.

## 2. Kết nối InfluxDB
- `INFLUX_URL` (ví dụ: `http://influxdb1:8086`, tùy mỗi cluster).
- `INFLUX_TOKEN`, `INFLUX_ORG`, `INFLUX_BUCKET` (theo cấu hình InfluxDB: authentication, bucket).
- Trong code, client InfluxDB sẽ gọi `writeApi.writePoint(...)` mỗi khi hoàn thành request, để ghi ra measurement `"request_metrics"` kèm:
  - Tag `app`: `"app-1"`, `"app-2"`, …
  - Field `duration_ms`, `status`.

---

## Vai trò & Phân tích

### Xử lý nghiệp vụ & ghi metrics
- Khi một request tới NGINX, NGINX sẽ chuyển tiếp đến App-X ngẫu nhiên (theo round-robin).
- App-X tính toán (nếu có), ghi chỉ số thời gian xử lý vào InfluxDB tương ứng, rồi trả JSON về NGINX.
- Nhờ mỗi App ghi metrics riêng, chúng ta có thể dễ dàng phân tích performance của từng instance: instance nào nhanh/slow, có bao nhiêu lỗi, v.v.

### Tăng khả năng scale-out & chịu lỗi
- NGINX cân bằng tải đều, ba instance App-1/2/3 (chạy độc lập) làm giảm thiểu tình trạng một node bị quá tải hoặc crash.
  - Nếu App-2 gặp sự cố, upstream NGINX sẽ đánh dấu “fail” (do `max_fails=2` và `fail_timeout=5s`), tạm thời không chuyển traffic đến App-2, chỉ còn App-1 & App-3 phục vụ cho đến khi App-2 “lành bệnh” (timeout quá 5s).
  - Trong tương lai, muốn nâng cao throughput, chỉ cần khởi thêm App-4, App-5 với cùng image và chỉnh NGINX upstream. Hoàn toàn không phải chỉnh code business logic.

### Tính module & dễ bảo trì
- Mỗi App là một microservice đóng gói riêng biệt. Developer có thể deploy, rollback, cập nhật App-2 mà không ảnh hưởng đến App-1, App-3.
- Nếu cần nâng cấp InfluxDB hay thay đổi cách ghi metrics, chỉ cần chỉnh code trong App, không ảnh hưởng cấu trúc NGINX.

---

## 4. InfluxDB-1/2/3
**Chức năng:**  
Ba instance InfluxDB-1, InfluxDB-2 và InfluxDB-3 đóng vai trò là kho lưu trữ dữ liệu time-series riêng biệt cho mỗi ứng dụng App-1, App-2 và App-3 tương ứng.  
- Mỗi khi App-X xử lý một request, nó sẽ ghi các thông số (metrics) liên quan—như thời gian xử lý, số request, lỗi, v.v.—vào InfluxDB-X thông qua HTTP API hoặc client library chính thức.  
- Dữ liệu này lưu trữ theo thứ tự thời gian, cho phép phân tích hiệu năng, theo dõi xu hướng, hoặc cảnh báo khi lập trình viên cần giám sát.  
- Về sau, module API Aggregator (nằm ở tầng cao hơn) sẽ truy vấn lần lượt ba cơ sở InfluxDB này để tổng hợp, xử lý và trả về report chung cho người dùng.

### Công nghệ
- **InfluxDB (Open Source):** Một database chuyên dụng cho dữ liệu time-series, tối ưu cho ghi dữ liệu tốc độ cao và truy vấn theo khoảng thời gian.
- **Cấu hình Container Docker:** Thông thường mỗi instance InfluxDB được triển khai dưới dạng container. Trong `docker-compose.yml`, ta định nghĩa ba service riêng biệt:
```yaml
influxdb1:
  image: influxdb:1.8
  container_name: influxdb-1
  ports:
    - "8086:8086"
  volumes:
    - influxdb1_data:/var/lib/influxdb
  environment:
    - INFLUXDB_DB=metrics1
    - INFLUXDB_HTTP_AUTH_ENABLED=false

influxdb2:
  image: influxdb:1.8
  container_name: influxdb-2
  ports:
    - "8087:8086"
  volumes:
    - influxdb2_data:/var/lib/influxdb
  environment:
    - INFLUXDB_DB=metrics2
    - INFLUXDB_HTTP_AUTH_ENABLED=false

influxdb3:
  image: influxdb:1.8
  container_name: influxdb-3
  ports:
    - "8088:8086"
  volumes:
    - influxdb3_data:/var/lib/influxdb
  environment:
    - INFLUXDB_DB=metrics3
    - INFLUXDB_HTTP_AUTH_ENABLED=false

volumes:
  influxdb1_data:
  influxdb2_data:
  influxdb3_data:
```
- Mỗi instance khởi động với database mặc định (metrics1, metrics2, metrics3) và tắt chế độ xác thực HTTP để dễ demo.
- Dữ liệu được lưu trên volume riêng (influxdbX_data) để đảm bảo persist khi container khởi động lại.

## Cấu hình cơ bản

1. **Database & Retention Policy**
   - Khi container InfluxDB khởi, biến môi trường `INFLUXDB_DB` sẽ tự động tạo database tương ứng (`metrics1`, `metrics2`, `metrics3`).
   - Nếu cần giữ dữ liệu chi tiết theo yêu cầu, có thể thiết lập thêm retention policy. Ví dụ, trong `influx` CLI:
     ```sql
     CREATE RETENTION POLICY "rp_7days" ON "metrics1" DURATION 7d REPLICATION 1 DEFAULT;
     CREATE RETENTION POLICY "rp_30days" ON "metrics1" DURATION 30d REPLICATION 1;
     ```
   - Tương tự cho `metrics2` và `metrics3`. Retention policy giúp tự động xóa hoặc downsample dữ liệu cũ khi quá hạn.

2. **Port & Hostname**
   - Theo ví dụ Docker Compose:
     - InfluxDB-1 lắng nghe cổng `8086` bên trong container, ánh xạ ra cổng `8086` của host, hostname nội bộ trong mạng Docker là `influxdb-1`.
     - InfluxDB-2 ánh xạ cổng `8086` container → cổng `8087` host, hostname nội bộ là `influxdb-2`.
     - InfluxDB-3 ánh xạ cổng `8086` container → cổng `8088` host, hostname nội bộ là `influxdb-3`.
   - Trong code của App-X, khi khởi connect, ta sẽ dùng URL như:
     ```env
     INFLUX_URL=http://influxdb-X:8086
     ```
     (với X = 1,2,3 tương ứng).

3. **Authentication & Security**
   - Ở bản demo đơn giản, `auth-enabled = false` (không bắt xác thực).
   - Khi lên production, cần bật `INFLUXDB_HTTP_AUTH_ENABLED=true`, tạo user và gán quyền:
     ```sql
     CREATE USER "userX" WITH PASSWORD 'passX';
     GRANT ALL ON "metricsX" TO "userX";
     ```
   - Nếu yêu cầu bảo mật cao, có thể bật TLS bằng cách bổ sung chứng chỉ vào `influx.conf` trong `[http]` hoặc truyền biến môi trường.
   ## Vai trò & Phân tích
- **Chia nhỏ dữ liệu, cô lập dễ quản lý**
  - Mỗi App-X viết thẳng vào InfluxDB-X—dữ liệu metric của App-1, App-2, App-3 được lưu tách biệt hoàn toàn. Điều này giúp dễ triển khai phân tán (nếu App-1 thuộc cluster A, App-2 cluster B), đồng thời giảm nguy cơ nghẽn đường ống khi lưu trữ quá nhiều metric chung trên một instance duy nhất.
  - Khi cần truy xuất, API Aggregator sẽ kết nối lần lượt đến ba endpoint InfluxDB-X, chạy các query `SELECT`/`FLUX`, sau đó tổng hợp kết quả (merge theo timestamp hoặc theo tag). Mặc dù cần nhiều bước hơn so với chỉ một database, nhưng nhờ cách cấu trúc này, ta có thể scale-out InfluxDB riêng lẻ, tăng tài nguyên cho từng node nếu một App-X phát sinh lượng metric quá lớn.

- **Tối ưu cho time-series, tốc độ ghi cao**
  - InfluxDB thiết kế riêng để chịu được hàng chục nghìn điểm dữ liệu mỗi giây. Mỗi khi App-X trả response, việc ghi một điểm dữ liệu (field = `duration`, tag = `appX`) diễn ra rất nhanh, không làm chậm quá trình xử lý chính.

## API
**Chức năng**: API Aggregator (microservice nằm trong thư mục `aggregator/`) chịu trách nhiệm thu thập dữ liệu time-series từ ba nguồn InfluxDB (`InfluxDB-1`, `InfluxDB-2`, `InfluxDB-3`) và trả về cho Frontend chỉ qua một endpoint duy nhất. Cụ thể:
1. Khi Frontend gọi HTTP đến `GET /api/metrics` (định nghĩa trong `server.js`), Aggregator sẽ tuần tự hoặc song song thực thi các truy vấn lên từng database `InfluxDB-1`, `InfluxDB-2`, `InfluxDB-3`.
2. Mỗi InfluxDB có một client riêng được khởi trong `multInfluxClient.js`, dùng token, URL và bucket tương ứng để kết nối (InfluxDB 2.x).
3. Module `query.js` sẽ chứa các hàm xây dựng và chạy câu Flux Query (hoặc InfluxQL nếu cần), ví dụ lấy trung bình `duration_ms` trong 1 giờ vừa qua, hoặc đếm số points.
4. Cuối cùng, Aggregator trả JSON này về cho Frontend, giúp Frontend chỉ gọi một lần mà có dữ liệu tổng hợp từ ba nguồn, dễ dàng hiển thị trong dashboard hoặc biểu đồ.

## Công nghệ
- **Node.js (v18+) + Express.js**
  - `server.js` khởi một Express server lắng nghe cổng (mặc định 5000).
  - Định nghĩa route chính `/api/metrics` (GET).
  - Sử dụng cơ chế bất đồng bộ (`async/await`) để gọi hàm lấy dữ liệu từ InfluxDB và trả về kết quả.

- **InfluxDB Client**
  - Sử dụng official InfluxDB 2.x client cho Node.js, cấu hình trong `multInfluxClient.js`.
  - Mỗi client được khởi với `token`, `url` và `bucket` tương ứng (`metrics1`, `metrics2`, `metrics3`).
  - Thực hiện các truy vấn Flux thông qua module `query.js`.
  - Tệp multInfluxClient.js export ba client riêng biệt, ví dụ:
  ```javascript
// multInfluxClient.js
const { InfluxDB } = require("@influxdata/influxdb-client");

// Mỗi biến môi trường sẽ được inject trong Docker Compose hoặc .env
const client1 = new InfluxDB({
  url: process.env.INFLUX_URL_1, 
  token: process.env.INFLUX_TOKEN_1
});
const client2 = new InfluxDB({
  url: process.env.INFLUX_URL_2, 
  token: process.env.INFLUX_TOKEN_2
});
const client3 = new InfluxDB({
  url: process.env.INFLUX_URL_3, 
  token: process.env.INFLUX_TOKEN_3
});

module.exports = {
  client1,
  client2,
  client3
};
```

