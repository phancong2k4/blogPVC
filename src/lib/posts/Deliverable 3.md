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

## 1. Ghi log vào InfluxDB từ ứng dụng Node.js  
**File:**`influxClient.js`.

**Chức năng:**Kết nối đến cơ sở dữ liệu InfluxDB và ghi lại các thông tin log như: tên ứng dụng, endpoint được truy cập, mã trạng thái HTTP, thời gian phản hồi, và request ID.

```js
// Import các lớp cần thiết từ thư viện influxdb-client
const { InfluxDB, Point } = require('@influxdata/influxdb-client');

// Khởi tạo client InfluxDB bằng các thông tin cấu hình từ biến môi trường
const influxDB = new InfluxDB({
  url: process.env.INFLUX_URL,     // URL của InfluxDB server
  token: process.env.INFLUX_TOKEN, // Token để xác thực
});

// Lấy đối tượng write API để ghi dữ liệu vào bucket cụ thể
const writeApi = influxDB.getWriteApi(
  process.env.INFLUX_ORG,          // Tên tổ chức trên InfluxDB
  process.env.INFLUX_BUCKET        // Tên bucket chứa log
);

// Hàm ghi log khi có request xảy ra
function logRequest(data) {
  const point = new Point('request_logs')         // Tạo một "point" mới thuộc measurement "request_logs"
    .tag('app_name', data.app)                    // Gắn tag tên ứng dụng
    .tag('status_code', data.status)              // Gắn tag mã trạng thái HTTP
    .tag('endpoint', data.endpoint)               // Gắn tag endpoint được truy cập
    .floatField('response_time', data.time)       // Trường dữ liệu thời gian phản hồi dạng số thực
    .stringField('request_id', data.id);          // Trường dữ liệu chuỗi là ID của request

  writeApi.writePoint(point);                     // Thực hiện ghi dữ liệu vào InfluxDB
}

module.exports = { logRequest };
```
---

## 2. Gọi hàm ghi log từ một route trong ứng dụng  
**File:**`server.js`.

**Chức năng:** Khi client gửi request tới /api/data, server sẽ xử lý, tính thời gian phản hồi, và ghi log thông tin đó vào InfluxDB.

```js
const express = require('express');
const { logRequest } = require('./influxClient'); // Import hàm ghi log

const app = express();
const port = 3000;

app.get('/api/data', (req, res) => {
  const startTime = Date.now(); // Lưu lại thời điểm bắt đầu xử lý request

  // Giả lập thời gian xử lý bất đồng bộ (delay ngẫu nhiên)
  setTimeout(() => {
    const responseTime = Date.now() - startTime; // Tính thời gian xử lý

    // Gọi hàm log để ghi lại dữ liệu log vào InfluxDB
    logRequest({
      app: 'NodeApp-1',                        // Tên ứng dụng
      status: 200,                             // HTTP Status
      endpoint: '/api/data',                   // Endpoint được gọi
      time: responseTime,                      // Thời gian phản hồi
      id: Math.random().toString(36).substring(7), // Tạo ID ngẫu nhiên cho request
    });

    res.json({ message: 'Hello from NodeApp-1' }); // Phản hồi JSON về client
  }, Math.random() * 100); // Độ trễ giả lập
});

app.listen(port, () => {
  console.log(`Server running at http://localhost:${port}`);
});
```
---
## 3. Tổng hợp dữ liệu từ nhiều node InfluxDB  
**File:**`aggregator.js`.

**Chức năng:** Gọi các hàm lấy log từ nhiều node InfluxDB khác nhau và gộp lại thành một mảng duy nhất.

```js
// Import các hàm truy vấn log từ 3 node khác nhau
const { queryNode1, queryNode2, queryNode3 } = require('./queryServices');

async function aggregateData() {
  try {
    // Gọi đồng thời cả 3 node, tăng tốc độ lấy dữ liệu
    const [data1, data2, data3] = await Promise.all([
      queryNode1(),
      queryNode2(),
      queryNode3(),
    ]);

    // Gộp dữ liệu từ cả 3 node lại
    const mergedData = [...data1, ...data2, ...data3];
    return mergedData;
  } catch (error) {
    console.error('Lỗi khi tổng hợp dữ liệu:', error);
    return []; // Trả về mảng rỗng nếu có lỗi
  }
}

module.exports = { aggregateData };
```
---
## 4. API trả về dữ liệu log đã tổng hợp  
**File:**`api.js`.

**Chức năng:**Cung cấp một API endpoint `/api/logs` để trả về toàn bộ dữ liệu log từ nhiều node.

```js
const express = require('express');
const { aggregateData } = require('./aggregator'); // Import hàm tổng hợp

const router = express.Router();

// Khi client gọi /api/logs, trả về toàn bộ log đã tổng hợp
router.get('/logs', async (req, res) => {
  const logs = await aggregateData(); // Gọi hàm lấy dữ liệu
  res.json({ logs });                 // Trả về log dạng JSON
});

module.exports = router;
```

---

## 5. Dashboard gọi API và hiển thị dữ liệu  
**File:**`dashboard.html`.

**Chức năng:** Giao diện đơn giản hiển thị dữ liệu log theo dạng bảng, được gọi từ API `/api/logs`.

```html
<!DOCTYPE html>
<html>
<head>
  <title>Dashboard Logs</title>
</head>
<body>
  <h2>Logs thu thập từ nhiều node</h2>

  <!-- Bảng để hiển thị dữ liệu -->
  <table border="1" id="logTable">
    <tr>
      <th>App</th>
      <th>Endpoint</th>
      <th>Status</th>
      <th>Response Time</th>
      <th>Request ID</th>
    </tr>
  </table>

  <script>
    // Gọi API để lấy dữ liệu log từ backend
    fetch('/api/logs')
      .then(res => res.json())
      .then(data => {
        const table = document.getElementById('logTable');
        
        // Duyệt qua từng log và thêm dòng vào bảng
        data.logs.forEach(log => {
          const row = document.createElement('tr');
          row.innerHTML = `
            <td>${log.app_name}</td>
            <td>${log.endpoint}</td>
            <td>${log.status_code}</td>
            <td>${log.response_time}</td>
            <td>${log.request_id}</td>
          `;
          table.appendChild(row);
        });
      });
  </script>
</body>
</html>
```
---
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
