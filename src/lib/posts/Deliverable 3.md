---
title: "Deliverable 3"
date: "2025-05-25"
updated: "2025-05-25"
categories:
  - "sveltekit"
  - "markdown"
excerpt: Deliverable 2
---
Deliverable 3

# Phần 1: Tóm tắt tiến độ dự án

## Các tính năng chính đã triển khai:

- **Ghi log phân tán**:  
  Mỗi ứng dụng Node.js đã ghi log thành công vào các node InfluxDB riêng biệt, bao gồm các thông tin như: tên app, endpoint, mã trạng thái, thời gian phản hồi, request ID.

- **Tổng hợp dữ liệu từ nhiều node**:  
  Sử dụng module `aggregator.js` để gom dữ liệu log từ nhiều InfluxDB lại thành một mảng duy nhất.

- **API trả log tổng hợp**:  
  Endpoint `/api/logs` đã hoạt động ổn định, cung cấp dữ liệu log tổng hợp cho frontend hoặc các dịch vụ khác.

- **Dashboard hiển thị log**:  
  Giao diện HTML đơn giản được xây dựng để hiển thị bảng log trực tiếp từ dữ liệu API.

---

## Tiến độ công việc:

| Hạng mục | Tình trạng |
|----------|------------|
| Ghi log vào InfluxDB từ Node.js | Đã hoàn thành |
| Kết nối và tổng hợp log từ nhiều node | Đã hoàn thành |
| API cung cấp dữ liệu log tổng hợp | Đã hoàn thành |
| Giao diện hiển thị dữ liệu | Đã hoàn thành |
| Retry khi lỗi mạng hoặc DB | Đang xử lý |
| Phân mảnh hoặc sao chép dữ liệu | Dự kiến tích hợp |
| Bảo mật API bằng token | Chưa thực hiện |

---

## Vấn đề đã gặp & hướng giải quyết:

1. **Trễ khi tổng hợp log từ nhiều node**
   - *Vấn đề*: Một số node phản hồi chậm làm chậm toàn bộ hệ thống.
   - *Cách xử lý*: Dùng `Promise.all` để tăng tốc độ truy vấn đồng thời.

2. **Không đồng nhất cấu trúc log giữa các node**
   - *Vấn đề*: Dữ liệu bị thiếu field hoặc sai kiểu.
   - *Cách xử lý*: Chuẩn hóa schema trước khi ghi vào DB và khi hiển thị.

3. **Lỗi xác thực khi kết nối InfluxDB**
   - *Vấn đề*: Cấu hình sai token hoặc URL.
   - *Cách xử lý*: Kiểm tra `.env`, thêm cơ chế kiểm tra cấu hình trước khi khởi động app.

---

## Kết luận ngắn gọn:

Dự án đã hoàn thành phần lớn chức năng cốt lõi.  
Hệ thống có thể ghi log phân tán, tổng hợp và hiển thị dữ liệu real-time.  
Nhóm đang tập trung vào việc cải tiến độ ổn định, bảo mật và mở rộng khả năng xử lý trong giai đoạn tiếp theo.


# Phần 2: demo hệ thống

---

## 1. Ghi log từ ứng dụng Node.js lên InfluxDB

### Mục đích demo
Hiển thị quá trình ứng dụng Node.js nhận request, xử lý, rồi ghi thông tin log (tên app, endpoint, status, response time, request ID) lên cơ sở dữ liệu InfluxDB.

### Hình ảnh/Video minh họa _(chèn tại đây)_

> Gợi ý:
> - Ảnh chụp màn hình terminal hoặc console đang chạy Node.js, có log in ra thời gian xử lý request.
> - Ảnh dashboard InfluxDB thể hiện các điểm dữ liệu mới (measurement “request_logs”).
> - Video quay quá trình gửi request bằng Postman/curl và log xuất hiện trong InfluxDB.

### Mô tả minh họa
“Khi client gửi request tới `/api/data`, server Node.js sẽ xử lý và ghi lại log vào InfluxDB với các thông tin chi tiết như: tên ứng dụng, endpoint được gọi, mã trạng thái HTTP, thời gian phản hồi và một ID duy nhất cho request.”

---

## 2. Tổng hợp dữ liệu log từ nhiều node InfluxDB

### Mục đích demo
Minh họa cách hệ thống đồng thời truy vấn log từ 3 node InfluxDB khác nhau rồi gộp chung thành một bộ dữ liệu duy nhất.

### Hình ảnh/Video minh họa _(chèn tại đây)_

> Gợi ý:
> - Ảnh chụp code trong `aggregator.js` gọi song song 3 node.
> - Bảng hoặc biểu đồ hiển thị log lấy từ từng node rồi hợp nhất.
> - Video demo terminal chạy `aggregateData()` và kết quả log trả về.

### Mô tả minh họa
“Hệ thống gọi song song 3 node InfluxDB để lấy dữ liệu log phân tán, rồi gộp lại thành một mảng duy nhất. Giúp tổng hợp dữ liệu nhanh và hiệu quả, đảm bảo không bị trễ hoặc thiếu sót.”

---

## 3. API trả về dữ liệu log tổng hợp

### Mục đích demo
Trình bày hoạt động của API `/api/logs` trả về toàn bộ log đã tổng hợp cho frontend hoặc các dịch vụ khác.

### Hình ảnh/Video minh họa _(chèn tại đây)_

> Gợi ý:
> - Ảnh giao diện Postman gọi API `/api/logs` và JSON log trả về.
> - Screenshot console backend khi API được gọi.
> - Video trình duyệt gọi API `/api/logs` và xem dữ liệu log.

### Mô tả minh họa
“API `/api/logs` sẽ nhận yêu cầu từ client, gọi hàm tổng hợp dữ liệu từ các node, rồi trả về toàn bộ log đã hợp nhất dưới dạng JSON.”

---

## 4. Dashboard hiển thị dữ liệu log

### Mục đích demo
Trình bày giao diện người dùng (frontend) gọi API để hiển thị log trong bảng, giúp người dùng dễ dàng xem và theo dõi dữ liệu log phân tán.

### Hình ảnh/Video minh họa _(chèn tại đây)_

> Gợi ý:
> - Ảnh chụp màn hình trang web dashboard hiển thị bảng log.
> - Video quay thao tác tải lại trang, bảng log tự động cập nhật.
> - GIF minh họa các dòng log được thêm vào liên tục.

### Mô tả minh họa
“Dashboard đơn giản này sử dụng fetch API để gọi endpoint `/api/logs`, nhận dữ liệu log dạng JSON và hiển thị lên bảng. Người dùng có thể theo dõi thông tin realtime của các request trên nhiều node.”

---

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
