---
title: "Deliverable 3"
date: "2025-05-25"
updated: "2025-05-25"
categories:
  - "sveltekit"
  - "markdown"
excerpt: Deliverable 3
---


# Phần 1: Tóm tắt tiến độ dự án

## 1. Mục tiêu
Dự án xây dựng một hệ thống phân tán sử dụng mô hình microservices, triển khai bằng Docker. Mục tiêu chính là:

- Xử lý và ghi nhận log từ nhiều ứng dụng web.
- Phân phối tải bằng NGINX Load Balancer.
- Lưu trữ dữ liệu log vào các cơ sở dữ liệu phân tán (InfluxDB).
- Hiển thị dữ liệu tổng hợp thông qua giao diện Dashboard.

## 2. Các tính năng chính đã triển khai
- Load Balancing: Cân bằng tải các request đến 3 ứng dụng web Node.js bằng NGINX.
- Ghi log dữ liệu: Mỗi ứng dụng ghi dữ liệu log vào một cơ sở dữ liệu InfluxDB riêng biệt.
- Truy xuất dữ liệu tập trung: API Service truy vấn dữ liệu từ nhiều DB và trả kết quả về dashboard.
- Giao diện Dashboard: Hiển thị dữ liệu tổng hợp cho người dùng cuối.
- Sinh tải mô phỏng: Sử dụng công cụ Request Generator để kiểm thử hệ thống dưới tải lớn.

## 3. Trạng thái các thành phần

| Thành phần             | Trạng thái     | Ghi chú ngắn gọn                                  |
|------------------------|----------------|--------------------------------------------------|
| Docker Compose setup   | Hoàn thành     | Kết nối các container và cấu hình mạng           |
| NGINX Load Balancer    | Hoàn thành     | Phân phối lưu lượng đến các ứng dụng             |
| App Web (1, 2, 3 - Node.js) | Hoàn thành | Xử lý và gửi dữ liệu log đến InfluxDB           |
| InfluxDB (1, 2, 3)     | Hoàn thành     | Lưu log từ các app web tương ứng                 |
| API Service            | Hoàn thành     | Tổng hợp dữ liệu từ 3 DB                         |
| Dashboard (View)       | Hoàn thành     | Hiển thị dữ liệu từ API Service                  |
| Request Generator      | Hoàn thành     | Tạo tải để kiểm thử                              |
| Kiểm thử toàn hệ thống | Đang thực hiện | Đang đánh giá hiệu năng và độ ổn định           |

## 4. Khó khăn & cách khắc phục
- Lỗi giao tiếp giữa các container Docker  
  → Đã xử lý bằng cách cấu hình Docker Compose với mạng dùng chung (`network: bridge`).

- Dữ liệu log không đồng nhất  
  → Đã thống nhất định dạng log trong các ứng dụng Node.js để dễ xử lý khi tổng hợp.

- Dashboard không hiển thị đúng dữ liệu  
  → Đã sửa API Service để đảm bảo định dạng dữ liệu đúng khi truy xuất từ nhiều DB.


# Phần 2: demo hệ thống

# Demo hoạt động của hệ thống

Để minh họa cho hoạt động của hệ thống, em đã xây dựng một môi trường ảo sử dụng Docker gồm nhiều container hoạt động cùng nhau. Hệ thống bao gồm các thành phần: `api`, `view`, `request`, `nginx`, và các app thành phần (`app-1`, `app-2`, `app-3`). Dưới đây là mô tả hoạt động chính kèm ảnh chụp giao diện minh họa:

## 1. Giao diện tổng quan Docker Compose

Giao diện hiển thị toàn bộ các container đang chạy của hệ thống. Mỗi thành phần như `api`, `view`, `request`, `nginx` và các app thành phần đều được container hóa với cổng kết nối cụ thể, ví dụ:
- `api`: cổng 3004
- `view`: cổng 3005
- `request`: cổng 3006  
- v.v...
![docker ](/images/docker.png)
## 2. Giao diện chính của hệ thống SPA (Nginx)

Giao diện chính được truy cập tại `localhost:8080`, hiển thị trang chủ của Spa Star Group
Giao diện được phục vụ bởi dịch vụ Nginx làm reverse proxy, điều hướng đến các service phía sau.
![nginx ](/images/nginx.png)
## 3. Giao diện tạo request (Thành phần request)

Tại `localhost:3006`, người dùng có thể tạo một số lượng request đến một domain cụ thể — trong trường hợp này là `http://nginx:80`. 
Mục đích:
- Kiểm tra tải
- Quan sát phản ứng của hệ thống khi có nhiều truy cập đồng thời
![request ](/images/request.png)

## 4. Giao diện hiển thị dữ liệu (Thành phần view)

Tại địa chỉ `localhost:3005`, giao diện biểu đồ hiển thị thông tin xử lý hoặc thống kê số lượng request theo thời gian.

Đây là công cụ trực quan giúp theo dõi:
- Hoạt động của hệ thống
- Hiệu suất thực tế
![view](/images/view.png)

## 5. Giao diện truy vấn dữ liệu bằng InfluxDB Data Explorer

Thông qua InfluxDB, hệ thống lưu trữ và truy vấn các dữ liệu thời gian thực.

Giao diện Data Explorer cho phép:

- Truy vấn các bucket dữ liệu với loại dữ liệu `web-requests`
- Lọc và trực quan hóa dữ liệu
- Xuất dữ liệu ra CSV để phân tích thêm
![influxdb](/images/influxdb.png)
# Phân 3:  Mã nguồn (Code Snippets)

# 1. Cấu hình Docker Compose (`docker-compose.yml`)

Tệp này định nghĩa các dịch vụ chính trong hệ thống, bao gồm `api`, `view`, `request`, `nginx`, và các ứng dụng backend (`app-1`, `app-2`, `app-3`).

```js
version: '3.8'

services:
  api:
    build: ./api
    ports:
      - "3004:3004"
    depends_on:
      - influxdb

  view:
    build: ./view
    ports:
      - "3005:3005"
    depends_on:
      - influxdb

  request:
    build: ./request
    ports:
      - "3006:3006"

  nginx:
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - app-1
      - app-2
      - app-3

  app-1:
    build: ./app
    ports:
      - "3001:3000"

  app-2:
    build: ./app
    ports:
      - "3002:3000"

  app-3:
    build: ./app
    ports:
      - "3003:3000"

  influxdb:
    image: influxdb:2.0
    ports:
      - "8086:8086"
    volumes:
      - influxdb_data:/var/lib/influxdb2

volumes:
  influxdb_data:
```
**Chú thích:**

`api`, `view` và `request` là các dịch vụ xử lý API, hiển thị dữ liệu, và gửi yêu cầu tương ứng.

nginx đóng vai trò là reverse proxy, phân phối lưu lượng đến các ứng dụng backend.

`app-1`, `app-2`, `app-3` là các ứng dụng backend xử lý yêu cầu.

influxdb là cơ sở dữ liệu thời gian thực để lưu trữ và phân tích dữ liệu request.
---
# 2. Cấu hình Nginx (nginx.conf)
Tệp cấu hình `Nginx` định nghĩa cách phân phối lưu lượng đến các ứng dụng backend.

```js
events {}

http {
  upstream backend {
    server app-1:3000;
    server app-2:3000;
    server app-3:3000;
  }

  server {
    listen 80;

    location / {
      proxy_pass http://backend;
    }
  }
}
```
**Chú thích:**
upstream backend định nghĩa nhóm các ứng dụng backend để Nginx phân phối lưu lượng.
proxy_pass chuyển tiếp các yêu cầu đến nhóm backend.
---

# 3. Gửi dữ liệu đến InfluxDB (api/index.js)
Đoạn mã này trong dịch vụ api gửi dữ liệu `request` đến `InfluxDB` để lưu trữ và phân tích.
```js
const { InfluxDB, Point } = require('@influxdata/influxdb-client');

const influxDB = new InfluxDB({ url: 'http://influxdb:8086', token: 'your-token' });
const writeApi = influxDB.getWriteApi('your-org', 'your-bucket');

app.post('/log', (req, res) => {
  const point = new Point('web-requests')
    .tag('app', req.body.app)
    .intField('count', req.body.count);
  writeApi.writePoint(point);
  res.sendStatus(200);
});
```
**Chú thích:**

Sử dụng thư viện InfluxDB client để gửi dữ liệu dạng Point với tag app và trường count.

API /log nhận dữ liệu từ các ứng dụng backend và ghi vào InfluxDB.
---

# 4. Giao diện gửi yêu cầu (request/index.html)
Giao diện người dùng cho phép gửi nhiều yêu cầu đến hệ thống để kiểm tra khả năng xử lý.
```html
<form id="request-form">
  <label for="url">URL:</label>
  <input type="text" id="url" name="url" value="http://nginx:80/">
  <label for="count">Số lượng request:</label>
  <input type="number" id="count" name="count" value="100">
  <button type="submit">Gửi</button>
</form>

<script>
  document.getElementById('request-form').addEventListener('submit', async (e) => {
    e.preventDefault();
    const url = document.getElementById('url').value;
    const count = document.getElementById('count').value;
    for (let i = 0; i < count; i++) {
      fetch(url);
    }
  });
</script>
```
**Chú thích:**
Form cho phép người dùng nhập URL và số lượng request muốn gửi.
Script JavaScript gửi các request đến URL đã nhập để kiểm tra hệ thống.
---

# 5. Hiển thị dữ liệu từ InfluxDB (view/index.js)
Dịch vụ view truy vấn dữ liệu từ InfluxDB và hiển thị biểu đồ thống kê.
```c
const { InfluxDB } = require('@influxdata/influxdb-client');

const influxDB = new InfluxDB({ url: 'http://influxdb:8086', token: 'your-token' });
const queryApi = influxDB.getQueryApi('your-org');

app.get('/data', async (req, res) => {
  const fluxQuery = `from(bucket:"your-bucket")
    |> range(start: -1h)
    |> filter(fn: (r) => r._measurement == "web-requests")`;
  const data = [];
  for await (const { values, tableMeta } of queryApi.iterateRows(fluxQuery)) {
    data.push(tableMeta.toObject(values));
  }
  res.json(data);
});
```
**Chú thích:**
Truy vấn dữ liệu từ InfluxDB trong khoảng thời gian 1 giờ gần nhất.
API /data trả về dữ liệu JSON để hiển thị trên giao diện người dùng.
# Phần 4: Danh sách tính năng đã hoàn thành

## 1. Giao tiếp phân tán giữa các node InfluxDB
- **Mô tả**:  
  Hệ thống đã triển khai thành công việc gọi dữ liệu log từ nhiều node InfluxDB khác nhau song song, đảm bảo thu thập dữ liệu nhanh và đồng bộ.

- **Ví dụ hoạt động**:  
  Khi gọi API tổng hợp log, hàm `aggregateData()` sẽ gửi truy vấn đồng thời đến 3 node InfluxDB khác nhau và nhận về dữ liệu log từ cả ba. Dữ liệu này sau đó được gộp lại thành một mảng duy nhất, phục vụ cho việc phân tích và hiển thị.

- **Đạt được mục tiêu**:  
  Tăng tốc độ truy xuất log, tránh tắc nghẽn khi chỉ lấy dữ liệu từ một node duy nhất, nâng cao độ tin cậy và khả năng mở rộng của hệ thống.

---

## 2. Ghi log chi tiết từ ứng dụng Node.js lên InfluxDB
- **Mô tả**:  
  Ứng dụng Node.js có thể ghi lại các thông tin log chi tiết như tên ứng dụng, endpoint được truy cập, trạng thái HTTP, thời gian phản hồi và ID của request vào cơ sở dữ liệu InfluxDB.

- **Ví dụ hoạt động**:  
  Mỗi khi client gọi endpoint `/api/data`, server sẽ tính toán thời gian phản hồi và gọi hàm `logRequest()` để gửi thông tin này vào InfluxDB. Điều này giúp theo dõi hiệu năng và hoạt động thực tế của ứng dụng.

- **Đạt được mục tiêu**:  
  Cung cấp dữ liệu giám sát realtime, hỗ trợ phát hiện nhanh các lỗi hoặc điểm nghẽn trong hệ thống.

---

## 3. Xử lý lỗi cơ bản và đảm bảo độ tin cậy
- **Mô tả**:  
  Trong quá trình tổng hợp dữ liệu, hệ thống đã thêm các đoạn code xử lý lỗi, đảm bảo khi một node không phản hồi hoặc có lỗi, ứng dụng vẫn có thể trả về dữ liệu từ các node còn lại mà không bị gián đoạn.

- **Ví dụ hoạt động**:  
  Hàm `aggregateData()` sử dụng `try-catch` để bắt lỗi khi truy vấn dữ liệu. Nếu một node gặp sự cố, hệ thống sẽ log lỗi và trả về dữ liệu còn lại thay vì bị dừng hoặc crash.

- **Đạt được mục tiêu**:  
  Nâng cao tính ổn định và khả năng chịu lỗi của hệ thống trong môi trường thực tế.

---

## 4. API trả về dữ liệu log tổng hợp cho frontend
- **Mô tả**:  
  Đã phát triển API `/api/logs` giúp frontend hoặc các dịch vụ khác dễ dàng lấy dữ liệu log tổng hợp từ nhiều node.

- **Ví dụ hoạt động**:  
  Khi frontend gọi API này, server sẽ trả về dữ liệu log dưới dạng JSON để hiển thị hoặc xử lý tiếp.

- **Đạt được mục tiêu**:  
  Giúp phân tách rõ ràng giữa backend và frontend, nâng cao modularity, đồng thời tiện lợi cho việc mở rộng giao diện người dùng.

---

## 5. Giao diện Dashboard hiển thị log realtime
- **Mô tả**:  
  Xây dựng dashboard đơn giản, trực quan giúp người dùng xem được log phân tán trên nhiều node dưới dạng bảng với các trường thông tin chính.

- **Ví dụ hoạt động**:  
  Dashboard gọi API `/api/logs`, nhận dữ liệu và cập nhật bảng log liên tục khi người dùng tải lại trang.

- **Đạt được mục tiêu**:  
  Tăng trải nghiệm người dùng, hỗ trợ việc giám sát hệ thống dễ dàng và nhanh chóng hơn.

---

# Phần 5: Kế hoạch tiếp theo

## 1. Các bước tiếp theo sẽ thực hiện

- **Hoàn thiện tính năng đồng bộ dữ liệu**:  
  Hiện tại hệ thống mới chỉ tập trung vào tổng hợp và ghi log, bước tiếp theo là xây dựng chức năng đồng bộ dữ liệu giữa các node InfluxDB để đảm bảo dữ liệu được sao chép chính xác và kịp thời trên toàn hệ thống.

- **Tối ưu hiệu năng truy vấn và ghi log**:  
  Sẽ nghiên cứu và áp dụng các kỹ thuật caching, batching ghi dữ liệu để giảm tải cho server và tăng tốc độ phản hồi.

- **Phát triển giao diện dashboard nâng cao**:  
  Thiết kế thêm các biểu đồ, thống kê trực quan (charts, graphs) để người dùng dễ dàng theo dõi hiệu năng, phân tích xu hướng dữ liệu theo thời gian.

---

## 2. Tính năng dự định hoàn thành trong giai đoạn tiếp theo

- **Hỗ trợ phân mảnh dữ liệu (sharding)**:  
  Phân chia dữ liệu log theo các tiêu chí nhất định để phân tán tải và tăng khả năng mở rộng hệ thống.

- **Cảnh báo và tự động xử lý lỗi**:  
  Xây dựng hệ thống cảnh báo khi có sự cố hoặc bất thường trong log (ví dụ: response time cao bất thường), đồng thời tự động khởi động lại các node hoặc dịch vụ bị lỗi.

- **Xác thực và bảo mật nâng cao**:  
  Bổ sung các lớp bảo mật như xác thực token nâng cao, mã hóa dữ liệu khi truyền tải để đảm bảo an toàn cho hệ thống.

---

## 3. Vấn đề cần giải quyết để hoàn thiện dự án

- **Đồng bộ dữ liệu đa node phức tạp**:  
  Việc đảm bảo dữ liệu nhất quán giữa các node phân tán vẫn là thử thách lớn, cần nghiên cứu thêm các giải pháp consensus, replication phù hợp.

- **Xử lý lỗi và sự cố phức tạp**:  
  Hệ thống phân tán thường gặp lỗi mạng, lỗi node, vì vậy phải xây dựng cơ chế phục hồi nhanh và giảm thiểu downtime.
