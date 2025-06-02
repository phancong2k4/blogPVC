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
3. Sơ đồ Trình tự
4. Các yêu cầu chức năng
   - 4.1. Sơ đồ Use-case
   - 4.2. Đặc tả các Use-case

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
![mohinhkientruc](/images/sodokientruc.png)
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

```js
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
- **Docker**  
  - Dockerfile trong `aggregator/` xây dựng image chứa Node.js, cài dependencies từ `package.json`, rồi khởi `node server.js`.  
  - Trong `docker-compose.yml`, khối service có thể như:
```yaml
aggregator:
  build: ./aggregator
  container_name: api-aggregator
  ports:
    - "5000:5000"
  environment:
    - PORT=5000
    - INFLUX_URL_1=http://influxdb-1:8086
    - INFLUX_TOKEN_1=${INFLUX_TOKEN_1}
    - INFLUX_ORG_1=${INFLUX_ORG}
    - INFLUX_BUCKET_1=metrics1
    - INFLUX_URL_2=http://influxdb-2:8086
    - INFLUX_TOKEN_2=${INFLUX_TOKEN_2}
    - INFLUX_ORG_2=${INFLUX_ORG}
    - INFLUX_BUCKET_2=metrics2
    - INFLUX_URL_3=http://influxdb-3:8086
    - INFLUX_TOKEN_3=${INFLUX_TOKEN_3}
    - INFLUX_ORG_3=${INFLUX_ORG}
    - INFLUX_BUCKET_3=metrics3
  depends_on:
    - influxdb1
    - influxdb2
    - influxdb3
```
- Khi container chạy, Express sẽ tự động lắng nghe cổng 5000, sẵn sàng trả dữ liệu khi có request từ UI.

## Cấu hình cơ bản
1. **config.json hoặc Biến môi trường**
- Nếu dùng `config.json` (nằm trong `aggregator/`), nội dung có thể:
```js
{
  "influxdb": [
    {
      "name": "db1",
      "url": "http://influxdb-1:8086",
      "org": "my-org",
      "bucket": "metrics1",
      "tokenEnv": "INFLUX_TOKEN_1"
    },
    {
      "name": "db2",
      "url": "http://influxdb-2:8086",
      "org": "my-org",
      "bucket": "metrics2",
      "tokenEnv": "INFLUX_TOKEN_2"
    },
    {
      "name": "db3",
      "url": "http://influxdb-3:8086",
      "org": "my-org",
      "bucket": "metrics3",
      "tokenEnv": "INFLUX_TOKEN_3"
    }
  ],
  "query": "from(bucket: \"BUCKET\") |> range(start: -1h) |> mean(column: \"duration_ms\")"
}
```
- Ở `server.js`, sẽ `require("./config.json")` rồi lặp qua `config.influxdb` để khởi client và xây flux query riêng cho mỗi bucket.  
- Hoặc bạn có thể không dùng `config.json` mà trực tiếp parse biến ENV (như ví dụ Docker Compose) để gán vào `multInfluxClient.js` và `query.js`.

### 2. Cách gọi & Tham số tùy chọn
- Frontend chỉ cần:
```js
fetch("http://<HOST_AGGREGATOR>:5000/api/metrics")
  .then(res => res.json())
  .then(data => {
    // data.db1, data.db2, data.db3
  });
```
- Nếu muốn cho phép query động (ví dụ time range do client chỉ định), có thể thêm query parameters:
```js
app.get("/api/metrics", async (req, res) => {
  const { from = "-1h", measurement = "request_metrics" } = req.query;
  const fluxQuery = `from(bucket:"BUCKET") |> range(start: ${from}) |> mean(column:"duration_ms")`;
  // Thay BUCKET tương ứng trước khi gọi fetchMetrics()
  ...
});
```
## Vai trò & Phân tích

- **Ẩn chi tiết multi-source**  
  Frontend UI (dashboard) chỉ phải thực hiện 1 HTTP request đến Aggregator, thay vì gọi đan xen 3 API InfluxDB khác nhau, parse rồi gộp dữ liệu. Điều này làm giảm độ phức tạp phía client và giảm tổng số kết nối ra mạng.

- **Giảm tải cho InfluxDB**  
  Với caching (nếu implement thêm), Aggregator có thể lưu giữ kết quả cache trong bộ nhớ (ví dụ Redis hoặc in-memory) trong khoảng thời gian ngắn (cache TTL). Khi UI gọi lại trong TTL, Aggregator trả dữ liệu cũ, tránh query liên tục. Nhờ vậy, InfluxDB không phải phục vụ cùng một câu query lặp đi lặp lại.

- **Khả năng mở rộng**  
  Khi thêm nguồn mới (ví dụ InfluxDB-4), chỉ cần thêm vào `config.json` mục tương ứng (url, bucket, tokenEnv) và khởi thêm client trong `multInfluxClient.js`. Không phải chỉnh lại logic core trong `server.js`, chỉ lặp qua danh sách nguồn động.  
  Nếu cần xử lý dữ liệu lớn (hàng trăm ngàn điểm), nên mở rộng thành microservice có thể scale-out (chạy nhiều replica), kết hợp với Kafka/Redis Streams để pre-process dữ liệu (tính toán off-line), sau đó trả về kết quả gọn hơn cho UI.

## 6. View

**Chức năng:**  
Phần giao diện là ứng dụng phía khách hàng (client) chịu trách nhiệm lấy dữ liệu từ API Aggregator và trình bày cho người dùng dưới dạng đồ thị, bảng, dashboard hoặc các thành phần tương tác khác. Khi người dùng truy cập vào URL của Frontend, trình duyệt sẽ tải file HTML/JS/CSS tương ứng (ví dụ: `chart.html`, `chart.js`), sau đó thực hiện các bước:

1. Gửi request đến API Aggregator (mặc định `http://aggregator:5000/api/metrics`) để lấy dữ liệu thời gian thực hoặc dữ liệu lịch sử (tùy vào cấu hình).
2. Nhận JSON trả về dạng:
```js
{
  "db1": { "avg_duration": 120, "count": 934 },
  "db2": { "avg_duration": 145, "count": 1021 },
  "db3": { "avg_duration": 98,  "count": 887 },
  "timestamp": "2025-06-01T10:00:00Z"
}
```
3. Xử lý dữ liệu (mapping, lọc, chuyển đổi format nếu cần) – do chart.js đảm nhiệm.  
4. Vẽ đồ thị (chart) bằng thư viện Chart.js hoặc D3.js (tùy cấu hình trong chart.js), hiển thị số liệu (hiệu năng, throughput, latency) dưới dạng biểu đồ đường, biểu đồ cột, v.v.

## Công nghệ

- **HTML / CSS / JavaScript**  
  - File chính `chart.html` (HTML5 + CSS cơ bản) để định nghĩa khung hiển thị, thẻ `<canvas>` cho Chart.js.  
  - Tập `chart.js` (ES6 hoặc ES5) chịu trách nhiệm:
  - Gọi API Aggregator (thông qua `fetch()` hoặc `axios.get()`).
 - Xử lý kết quả JSON trả về (map lên mảng data, array of labels, etc.).
 - Khởi tạo Chart.js:

```js
// Ví dụ pseudo-code:
const ctx = document.getElementById("myChart").getContext("2d");
const myChart = new Chart(ctx, {
  type: "line",
  data: {
    labels: timeLabels,          // ["10:00", "10:01", ...]
    datasets: [
      {
        label: "App-1 Avg Duration (ms)",
        data: durationsDb1,       // [120, 118, ...]
        borderColor: "rgb(75, 192, 192)",
        fill: false
      },
      {
        label: "App-2 Avg Duration (ms)",
        data: durationsDb2,
        borderColor: "rgb(255, 99, 132)",
        fill: false
      },
      {
        label: "App-3 Avg Duration (ms)",
        data: durationsDb3,
        borderColor: "rgb(54, 162, 235)",
        fill: false
      }
    ]
  },
  options: {
    responsive: true,
    scales: {
      x: { display: true, title: { text: "Time", display: true } },
      y: { display: true, title: { text: "Avg Duration (ms)", display: true } }
    }
  }
});
```
- Chart.js (đã cài qua package.json): thư viện vẽ biểu đồ đơn giản, hỗ trợ nhiều loại đồ thị (line, bar, pie, v.v.), dễ cấu hình.
- Express.js (nếu server.js dùng Express) hoặc http-server (nếu chỉ serve static mà không cần tùy chỉnh route).

- React.js / Vue.js / Angular (nếu dự án mở rộng)

Mặc dù folder hiện tại chỉ có HTML + JS thuần với Chart.js, bạn có thể chuyển sang React (hoặc Vue, Angular) để xây dựng SPA, ví dụ:
- Cài create-react-app, viết component `<Dashboard />`, `<Chart />`, `<FilterPanel />`
- Sử dụng Chart.js qua react-chartjs-2 hoặc D3.js.
- Quản lý state bằng Context API, Redux, hoặc Vuex (với Vue).
- Build ra bundle tĩnh (`npm run build`) và serve bằng NGINX static.

## Cấu hình cơ bản

1. **API Endpoint**

   - Trong file `chart.js`, mặc định URL của Aggregator có thể khai báo:
     ```js
     const API_URL = process.env.REACT_APP_API_URL || "http://localhost:5000/api/metrics";
     ```
   - Nếu không dùng React, bạn có thể để thẳng:
     ```js
     const API_URL = "http://aggregator:5000/api/metrics";
     ```
   - Khi chạy container, truyền biến môi trường `REACT_APP_API_URL` hoặc `API_URL` (tuỳ cách implement) để UI biết gọi đúng địa chỉ backend.

2. **Port & Hosting**

   - Trong `server.js`, UI lắng nghe port `PORT` (mặc định 3000). Có thể override bằng biến môi trường `PORT=4000`.
   - Nếu build static và serve qua NGINX, UI sẽ lắng nghe port 80 trong container, ánh xạ ra host tuỳ ý.

3. **Tham số tương tác (filter, time range)**

- `chart.html` có thể chứa form hoặc dropdown:
```html
<label>Chọn khoảng thời gian:</label>
<select id="timeRange">
  <option value="-1h">Last 1h</option>
  <option value="-6h">Last 6h</option>
  <option value="-24h">Last 24h</option>
</select>
<button id="refreshButton">Refresh</button>
```
- Trong chart.js, lắng nghe sự kiện change của dropdown và click của button để gọi lại API với query parameter:
```js
const timeRangeSelect = document.getElementById("timeRange");

document.getElementById("refreshButton").addEventListener("click", () => {
  const from = timeRangeSelect.value;
  fetch(`${API_URL}?from=${encodeURIComponent(from)}`)
    .then((res) => res.json())
    .then((data) => updateChart(data))
    .catch((err) => showError(err));
});
```
# Vai trò & Phân tích

- **Điểm tiếp xúc duy nhất cho người dùng**
  - Frontend đóng vai trò hiển thị toàn bộ dashboard: biểu đồ thời gian thực, số liệu tổng hợp, bảng phân tích chi tiết. Người dùng chỉ cần mở một URL (ví dụ: `http://ui-host:3000/`) để xem toàn bộ số liệu mà không cần biết backend phức tạp.
  - Khi có nhiều người dùng truy cập, các file tĩnh (HTML/JS/CSS) hoàn toàn có thể phân tán qua CDN hoặc nhiều instance web server, giúp tăng khả năng chịu tải (scale-out).

- **Tách biệt với backend**
  - UI hoàn toàn độc lập so với API Aggregator và các service phía sau. Lần deploy UI mới chỉ cần build lại file tĩnh và reload container/nginx, không ảnh hưởng đến API hoặc database.
  - Nếu muốn thay đổi giao diện (chuyển từ Chart.js sang D3.js, hoặc đổi layout), chỉ cần chỉnh `chart.html`/`chart.js` hoặc code React; backend không cần sửa.
# IV. Các Sơ Đồ Chính Của Hệ Thống

Phần này trình bày các sơ đồ trực quan hóa cấu trúc và luồng hoạt động của hệ thống.

## 1. Sơ đồ Kiến trúc Hệ thống
![mohinhkientruc](/images/sodokientruc.png)
## 2. Sơ đồ Triển khai
![sodotrienkhai](/images/sodotrienkhai1.png)
## 3. Sơ đồ Trình tự
![sodotrinhtu](/images/Sodotrinhtu.png)
## 4. Các yêu cầu chức năng

| Nhóm chức năng        | Chức năng                                                | Tác nhân  |
|----------------------|---------------------------------------------------------|-----------|
| Sinh tải kiểm thử     | Gửi các HTTP request đến NGINX để kiểm tra hiệu năng    | Tester    |
| Giám sát trạng thái dịch vụ | Theo dõi tình trạng (health) của các service (App-1/2/3, InfluxDB-1/2/3) |           |
| Xem dữ liệu thô InfluxDB | Truy vấn trực tiếp vào InfluxDB (qua giao diện Data Explorer) để xem dữ liệu chưa tổng hợp |           |
| Xem Dashboard giao diện (view) | Mở Frontend UI để kiểm tra biểu đồ, số liệu cơ bản       |           |

### 4.1. Sơ đồ Use-case:
![sodousecase](/images/sodousecase.png)
### 4.2. Đặc tả các Use-case
#### a. Tạo yêu cầu kiểm thử

| Thành phần     | Mô tả |
|----------------|------|
| **Mô tả**      | Chức năng cho phép **Tester** cấu hình và khởi tạo một loạt HTTP request đến **NGINX Load Balancer**, nhằm kiểm tra hiệu năng, độ ổn định và khả năng chịu tải của hệ thống. |
| **Tác nhân**   | Tester |
| **Luồng chính** | - Tester truy cập giao diện **“Request”**. <br> - Hệ thống hiển thị form với các trường nhập: <ul><li>URL đích (mặc định: `http://nginx:80/`)</li><li>Số lượng request</li></ul> - Tester nhấn nút **“Submit”** để bắt đầu kiểm thử. |
| **Luồng con** | - Nếu Tester **để trống hoặc nhập sai giá trị** “Số lượng request” (ví dụ: không phải số dương), hệ thống hiển thị **Validation Error** và không cho phép submit. <br> - Nếu Tester nhập **URL sai định dạng** (không phải `http://...`), hệ thống hiển thị lỗi “URL không hợp lệ”. |
| **Tiền điều kiện** | Giao diện Request đã được build, frontend có thể kết nối backend để gửi request. |
| **Hậu điều kiện** | Hệ thống đã gửi các HTTP request và ghi nhận kết quả: <ul><li>Thành công / Thất bại</li><li>Thời gian phản hồi</li></ul> Những dữ liệu này được dùng cho đánh giá hiệu năng. |
![request](/images/request.png)
#### b. Giám sát trạng thái dịch vụ

| Thành phần     | Mô tả |
|----------------|------|
| **Mô tả**      | Cho phép **Tester** theo dõi **tình trạng (health status)** của các container: `App-1`, `App-2`, `App-3`, `InfluxDB-1`, `InfluxDB-2`, `InfluxDB-3`. Từ đó, Tester biết container nào đang chạy, đang lỗi (crash), hoặc gặp sự cố tài nguyên. |
| **Tác nhân**   | Tester |
| **Luồng chính** | - Hệ thống hiển thị **Danh sách container** kèm theo các thông tin:<ul><li>Tên container: `App-1`, `App-2`, `App-3`, `InfluxDB-1`, `InfluxDB-2`, `InfluxDB-3`</li><li>Trạng thái: `Running`, `Exited`, `Restarting`, `Error`</li></ul> |
| **Luồng con** | - Nếu **CPU > 80%** hoặc **RAM > 80%**, hệ thống hiển thị cảnh báo **"⚠ High CPU/RAM"** bên cạnh container đó. <br> - Tester có thể căn cứ vào cảnh báo để quyết định **giảm tải** hoặc **tạm dừng kiểm thử**. |
| **Tiền điều kiện** | - Các container đã được bật **health check** (trong Docker Compose hoặc Kubernetes). <br> - Tester có quyền truy cập dashboard (như Portainer, Grafana) hoặc CLI (`docker ps`, `docker stats`) để xem trạng thái. |
| **Hậu điều kiện** | - Tester đã **nắm được trạng thái vận hành** của từng service/container. <br> - Có thể **báo cáo lỗi** cho Admin hoặc **điều chỉnh chiến lược kiểm thử** cho phù hợp. |
![docker](/images/docker.png)
#### c. Xem dữ liệu thô trong InfluxDB

| Thành phần     | Mô tả |
|----------------|------|
| **Mô tả**      | Cho phép **Tester** truy vấn trực tiếp các bucket (ví dụ: `web-requests`, `cpu-usage`...) trong `InfluxDB-1`, `InfluxDB-2`, `InfluxDB-3` để kiểm tra **dữ liệu dạng time-series** đã được các `App-1/2/3` ghi vào có đầy đủ, đúng định dạng và chính xác hay không. |
| **Tác nhân**   | Tester |
| **Luồng chính** | - Tester mở **Data Explorer** của InfluxDB (ví dụ: http://influxdb1:8086). <br> - Đăng nhập (nếu hệ thống bật authentication). <br> - Chọn bucket cần xem (ví dụ: `web-requests`). <br> - Nhấn **Run/Submit** để thực thi truy vấn. <br> - Hệ thống trả kết quả dưới dạng bảng hoặc biểu đồ. <br> - Tester xem các thông tin: thời gian, giá trị (count, latency, error...), tags (app, method...). <br> - Nếu dữ liệu bất thường hoặc sai lệch, Tester ghi nhận để **báo lỗi hoặc điều chỉnh App**. |
| **Luồng con** | - **Không có dữ liệu**: Nếu phạm vi thời gian query quá ngắn (ví dụ `last 1m`), kết quả trả về là bảng trống. <br> → Tester chỉnh lại thời gian (ví dụ `last 24h`) và chạy lại. |
| **Tiền điều kiện** | - Các instance InfluxDB (1/2/3) **đang chạy ổn định** và đã cấu hình đầy đủ `bucket`, `retention policy`, và `authentication` (nếu có). <br> - Tester có **tài khoản** hoặc **API Token** để truy cập. <br> - Biết rõ URL kết nối từng InfluxDB:<ul><li>InfluxDB-1: http://influxdb1:8086/</li><li>InfluxDB-2: http://influxdb2:8086/</li><li>InfluxDB-3: http://influxdb3:8086/</li></ul> |
| **Hậu điều kiện** | - Tester đã **kiểm tra được tính chính xác và định dạng của dữ liệu** ghi vào. <br> - Có cơ sở để xác định lỗi ở phía App ghi dữ liệu, hoặc phát hiện thiếu dữ liệu, phục vụ việc tối ưu hệ thống. |
![influxdb](/images/influxdb.png)
#### d. Xem Dashboard giao diện (View)

| Thành phần       | Mô tả |
|------------------|------|
| **Mô tả**        | Cho phép **Tester** truy cập trang Dashboard (giao diện View) để xem các số liệu đã được **tổng hợp từ API Aggregator**, bao gồm: <br> - Tổng số request <br> - Độ trễ trung bình (average latency) <br> - Tỷ lệ lỗi (error rate) <br> Dữ liệu được hiển thị trực quan dưới dạng biểu đồ cho các App (`App-1`, `App-2`, `App-3`) hoặc dưới dạng tổng hợp. |
| **Tác nhân**     | Tester |
| **Luồng chính**  | - Tester mở trình duyệt và truy cập `http://<host>:3005`. <br> - Frontend gửi yêu cầu đến API Aggregator để lấy dữ liệu. <br> - Frontend nhận lại JSON chứa dữ liệu tổng hợp. <br> - Giao diện View hiển thị dữ liệu dưới dạng biểu đồ, bảng, hoặc các chỉ số tổng quan. |
| **Luồng con**    | - Trường hợp **API trả về JSON rỗng** (không có dữ liệu): <br> → Frontend hiển thị thông báo "Không có dữ liệu", hoặc hiển thị dashboard rỗng. <br> → Tester có thể kiểm tra lại thời gian, App đang test, hoặc báo lỗi. |
| **Tiền điều kiện** | - API Aggregator đang hoạt động ổn định và trả dữ liệu đúng định dạng. <br> - Frontend UI đã được build và host tại địa chỉ `http://<host>:3005`. <br> - Tester có trình duyệt và biết chính xác URL truy cập dashboard. |
| **Hậu điều kiện** | - Tester đã xem được số liệu hiệu năng một cách **trực quan**. <br> - Có cơ sở để **điều chỉnh kịch bản kiểm thử**, báo cáo lỗi với Admin, hoặc đánh giá khả năng chịu tải của hệ thống. |
![View](/images/view.png)

# V. Phân Tích Vấn Đề và Rủi Ro Tiềm Ẩn

## 1. Vấn Đề Hiện Tại

- **Thiếu độ chịu lỗi (Fault Tolerance)**
  - Mỗi instance App-1/2/3 và InfluxDB-1/2/3 chạy đơn lẻ, không có cơ chế health check và auto-restart.
  - Nếu container gặp lỗi, toàn bộ dữ liệu hoặc luồng request sẽ bị gián đoạn, gây mất dữ liệu hoặc ngưng trệ.

- **Không có phân mảnh (Sharding) và sao chép (Replication) dữ liệu**
  - Mỗi InfluxDB chỉ gán cho một App riêng biệt, không có sharding để phân tán dữ liệu khi lưu trữ lớn.
  - Không có cơ chế replica giữa các node, dẫn tới nguy cơ mất dữ liệu khi node bị lỗi.
  - Hiệu năng có thể giảm nghiêm trọng khi InfluxDB phải chịu tải lớn.

- **Điểm nghẽn trung tâm (Single Point of Failure)**
  - NGINX Load Balancer và API Aggregator chỉ chạy dưới dạng một container đơn lẻ.
  - Nếu một trong các container này lỗi, toàn bộ hệ thống mất khả năng xử lý request hoặc lấy dữ liệu.

- **Khả năng mở rộng (Scalability) hạn chế**
  - Hệ thống dùng Docker Compose để khởi động container tĩnh, không có dynamic scaling.
  - Việc tăng thêm container hoặc tách node phải làm thủ công, kém linh hoạt khi tải tăng.

## 2. Rủi Ro Tiềm Ẩn

- **Mất dữ liệu khi InfluxDB gặp sự cố**
  - Nếu InfluxDB-2 sập, dữ liệu metrics của App-2 sẽ bị mất.
  - Không có replica để khôi phục, dẫn đến thiếu thông tin trong báo cáo.

- **Downtime của NGINX hoặc API Aggregator**
  - Nếu NGINX container lỗi, request không thể tới các App, hệ thống ngưng hoạt động.
  - API Aggregator lỗi dẫn đến giao diện frontend không thể hiển thị dữ liệu.

- **Performance Bottleneck tại API Aggregator**
  - Tăng đột biến người dùng hoặc lượng dữ liệu lớn khiến Aggregator quá tải.
  - Không có caching hay queue, có thể dẫn đến chậm hoặc crash.

- **Thiếu khả năng mở rộng tự động**
  - Không thể tự động scale khi tải tăng, dẫn đến hiện tượng latency spike hoặc timeout.

- **Kém khả năng bảo mật và nhất quán**
  - Chưa có RBAC mạnh mẽ, token API có thể bị hardcode.
  - Nếu container InfluxDB bị tấn công, dữ liệu có thể bị thay đổi hoặc xóa trái phép.

- **Khó khăn đồng bộ và đồng nhất dữ liệu khi mở rộng**
  - Thiếu chiến lược sharding, replication phức tạp khi mở rộng số lượng instance InfluxDB.
# VI. ĐỀ XUẤT GIẢI PHÁP VÀ HƯỚNG PHÁT TRIỂN

Nhằm khắc phục các vấn đề nêu trên và áp dụng đúng các khái niệm phân tán đã học (fault tolerance, sharding, replication, giao tiếp phân tán…), đề xuất các giải pháp và hướng phát triển như sau:

## 1. Giải Quyết Các Vấn Đề Hiện Tại

### Tăng cường độ chịu lỗi (Fault Tolerance)

- Sử dụng Docker Swarm / Kubernetes để triển khai NGINX và API Aggregator ở chế độ HA (High Availability). Cụ thể:
  - Chạy ít nhất 2 replica cho NGINX (sau đó đặt phía trước là một load balancer cứng hoặc DNS Round-Robin).
  - Triển khai 2–3 replica cho API Aggregator, dùng Ingress Controller (trên Kubernetes) hoặc nginx-proxy khác làm cân bằng tải nội bộ giữa các node Aggregator.
- Health check định kỳ: cấu hình probe (liveness/readiness) cho container App-1/2/3 và InfluxDB. Nếu container hỏng, Kubernetes sẽ tự động khởi động lại.
- Circuit Breaker (nếu API Aggregator gọi tới App khác hoặc các service bên ngoài): sử dụng thư viện như resilience4j (Java) hoặc opossum (Node.js) để ngăn chặn “thác lũ request” vào service đã gặp lỗi.

### Replication và Backup cho InfluxDB

- Triển khai InfluxDB Cluster (Enterprise) hoặc InfluxDB OSS với Chronograf + Kapacitor:
  - Dùng tính năng replication factor ≥ 2 để mỗi đoạn dữ liệu (shard) được sao chép trên tối thiểu hai node.
  - Nếu một node InfluxDB bị lỗi, node khác vẫn có thể phục vụ yêu cầu ghi/đọc.
- Backup định kỳ: thiết lập task để xuất dữ liệu (thường xuyên, ví dụ mỗi 6 giờ) sang blob storage hoặc file backup lưu trữ ngoài. Khi cần khôi phục, có thể import lại toàn bộ hoặc một phần bucket.

### Giải quyết điểm nghẽn (Single Point of Failure)

- Thay vì chỉ dùng một container NGINX, deploy HAProxy hoặc NGINX Plus ở chế độ active-active.
- Triển khai thêm API Gateway (ví dụ Kong hoặc Traefik) để cân bằng tải giữa nhiều instance API Aggregator, đồng thời xử lý chứng thực, rate-limiting.
- Đảm bảo Frontend UI có fallback: khi một API Aggregator gặp lỗi, ứng dụng tự động chuyển sang API Aggregator khác.

### Cải thiện giao tiếp phân tán (Distributed Communication)

- Đưa message broker (Kafka, RabbitMQ) vào giữa App và InfluxDB. Khi App-1/2/3 ghi metrics, chúng đẩy message vào topic tương ứng. Từ đó, một consumer riêng (InfluxDB Writer Service) đọc và ghi batch vào InfluxDB, giảm tần suất ghi trực tiếp.
- Tương tự, khi API Aggregator phải tổng hợp dữ liệu, có thể dùng cache layer (Redis) để lưu tạm kết quả query phổ biến trong khoảng thời gian ngắn (ví dụ 10s). Khi user gọi lại, Aggregator trả dữ liệu từ cache mà không query trực tiếp 3 DB, giảm độ trễ.

## 2. Cải Thiện và Nâng Cấp

### Phân mảnh (Sharding) cho InfluxDB

- Nếu dữ liệu time-series quá lớn (hàng trăm triệu điểm trên mỗi giờ), cấu hình shard group sao cho mỗi Shard Duration hợp lý (ví dụ 1d hoặc 6h) để InfluxDB tự động tách và phân phối file dữ liệu.
- Xây dựng sẵn retention policy khác nhau cho dữ liệu detail (1 tuần) và dữ liệu tổng hợp (1 tháng), giúp giảm dung lượng lưu trữ.
- Khảo sát sử dụng Chronograf để tự động giám sát shard health, cân bằng disk I/O giữa các node.

### Sao chép (Replication) cho App Service

- Với App-1/2/3, triển khai ReplicaSet (Kubernetes) hoặc Swarm Service (Docker Swarm) để mỗi service có ít nhất 2–3 bản sao, đảm bảo khi một pod/container lỗi, các pod khác vẫn sẵn sàng nhận request.
- Sử dụng Service Mesh (Istio/Linkerd) để quản lý lưu lượng nội bộ giữa các instance App, cung cấp tính năng retry, timeout, circuit breaker.

### Cải thiện bảo mật (Security Hardening)

- Mã hóa TLS giữa các thành phần: NGINX ↔ App, App ↔ InfluxDB, Frontend ↔ API Aggregator.
- Triển khai Vault (HashiCorp) hoặc Kubernetes Secrets để lưu trữ an toàn các token/API Key thay vì hardcode.
- Áp dụng RBAC (Role-Based Access Control) cho InfluxDB, giới hạn quyền ghi/đọc theo từng service.

## 3. Hướng Phát Triển Tương Lai

### Áp dụng kiến trúc Event-Driven & Micro-Frontends

- Để tối ưu hóa giao tiếp giữa các service, xây dựng thành phần Event Bus (Kafka/RabbitMQ) để App-1/2/3 publish event khi có request mới hoặc metrics, Aggregator subscribe và xử lý bất đồng bộ.
- Phân chia Frontend UI thành các Micro-Frontend chuyên trách hiển thị từng loại số liệu (ví dụ one micro-frontend cho biểu đồ request, one cho logs, one cho cảnh báo).

### Triển khai Machine Learning cho dự đoán & cảnh báo sớm

- Thu thập dữ liệu time-series dài hạn, đào tạo mô hình Machine Learning (ví dụ ARIMA, Prophet) để dự đoán về độ trễ, request load trong tương lai.
- Tích hợp mô hình vào Kapacitor (InfluxData) hoặc MLflow để tạo cảnh báo sớm khi số liệu vượt ngưỡng bất thường.

### Kết nối với hệ sinh thái IoT hoặc Big Data

- Nếu hệ thống mở rộng để thu thập hàng triệu điểm metrics/giây (use case IoT), có thể tích hợp Apache Pulsar hoặc InfluxData Telegraf agents để thu thập và chuyển tiếp dữ liệu.
- Đưa dữ liệu vào Hadoop HDFS / Apache Druid để thực hiện phân tích dạng Big Data, báo cáo logs và trend dài hạn.

---

# VII. ĐÁNH GIÁ THEO TIÊU CHÍ KỸ THUẬT VÀ ĐỀ XUẤT

## 1. Tiêu chí khả năng mở rộng (Scalability)

- **Hiện tại:** Dự án chỉ sử dụng Docker Compose, scale tĩnh. Khó xử lý tải tăng đột biến.
- **Đề xuất:** Triển khai Autoscaling (Horizontal Pod Autoscaler trên Kubernetes) đảm bảo tự động tăng/giảm số instance App và API Aggregator dựa trên CPU/Memory hoặc custom metrics (ví dụ: request rate).

## 2. Tiêu chí chịu lỗi (Fault Tolerance & High Availability)

- **Hiện tại:** Mỗi thành phần chạy đơn lẻ, không có replica hoặc cluster.
- **Đề xuất:**
  - NGINX & API Aggregator: chạy ít nhất 2–3 replica, cấu hình Ingress/Load Balancer HA.
  - InfluxDB: thiết lập cluster replication (Replication Factor ≥ 2), hoặc dùng dịch vụ Managed InfluxDB có sẵn HA.
  - App-1/2/3: dùng ReplicaSet (K8s) hoặc Service Scale (Swarm), health check và readiness probe.

## 3. Tiêu chí phân mảnh và sao chép dữ liệu (Sharding & Replication)

- **Hiện tại:** Dữ liệu time-series chỉ được ghi độc lập vào từng instance InfluxDB, không có khoá vùng dữ liệu.
- **Đề xuất:**
  - Sử dụng InfluxDB Enterprise hoặc TiDB/ClickHouse để tự động sharding và replication.
  - Nếu giữ InfluxDB OSS, cấu hình shard duration và retention policy hợp lý, đồng thời backup định kỳ.

## 4. Tiêu chí giao tiếp phân tán (Distributed Communication)

- **Hiện tại:** App-1/2/3 ghi metrics trực tiếp, API Aggregator gọi trực tiếp. Thiếu layer trung gian, dễ bị nghẽn khi tải cao.
- **Đề xuất:**
  - Dùng Message Broker (Kafka/RabbitMQ) để App publish message và InfluxDB Writer subscribe, đảm bảo decoupling, giảm áp lực ghi trực tiếp.
  - Caching tại API Aggregator (Redis hoặc in-memory cache) để giảm truy vấn dư thừa vào InfluxDB.

## 5. Tiêu chí bảo mật (Security)

- **Hiện tại:** Token/API Key có thể hardcode, TLS chưa được mã hóa đầy đủ.
- **Đề xuất:**
  - Mã hóa toàn bộ giao tiếp (TLS) giữa các service.
  - Sử dụng Vault hoặc Kubernetes Secrets để lưu trữ thông tin nhạy cảm.
  - Áp dụng RBAC cho InfluxDB, phân quyền cụ thể (chỉ App có quyền ghi, chỉ Tester/Admin có quyền đọc raw data).

## 6. Tiêu chí bảo trì và Vận hành (Maintainability & Operability)

- **Hiện tại:** Chưa có CI/CD, việc build và deploy vẫn thủ công.
- **Đề xuất:**
  - Xây dựng pipeline CI/CD (GitHub Actions/GitLab CI) để tự động build–test–deploy khi có thay đổi mã nguồn.
  - Triển khai Prometheus + Grafana + Alertmanager để giám sát hệ thống, cảnh báo kịp thời.
  - Dùng Helm Chart để quản lý Kubernetes manifest, dễ versioning và rollback.

---

# KẾT LUẬN

Đồ án “THEO DÕI LƯỢNG REQUEST CỦA TRANG WEB” đã hoàn thành mô hình giám sát đơn giản: NGINX phân phối yêu cầu, ba dịch vụ (App-1, App-2, App-3) ghi số liệu vào ba cơ sở dữ liệu InfluxDB, API Aggregator gom mọi dữ liệu và giao diện web hiển thị các biểu đồ về số lượng yêu cầu, độ trễ và tỉ lệ lỗi.

Nhóm đã khởi động đồng loạt các thành phần bằng Docker Compose và áp dụng các khái niệm cơ bản về cân bằng tải, thu thập số liệu và hiển thị dashboard.

Trong tương lai, có thể tiếp tục đơn giản hóa giao diện, phân bổ tài nguyên tự động và bổ sung cảnh báo thông minh dựa trên dữ liệu thu thập được.
