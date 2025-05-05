---
title: "Tiến trình & Luồng"
date: "2025-05-05"
updated: "2025-05-05"
categories:
  - "sveltekit"
  - "markdown"
coverImage: "/images/Untitled1.png"
coverWidth: 16
coverHeight: 9
excerpt: tiến trình và luồng
---

Phan Văn Công - 22010158

## 1. Phân tích hiệu năng máy cá nhân
**Thông tin máy:**
 **CPU**: Intel Core i5-1135G7 (4 nhân vật lý, 8 luồng xử lý)
- **RAM**: 8 GB DDR4
- **GPU**: Intel Iris Xe Graphics (tích hợp)
![Ảnh](/images/Untitled.png)

- **CPU**: Vi xử lý Intel Core i5 thế hệ 11 có công nghệ **Hyper-Threading**, nghĩa là mỗi nhân vật lý xử lý 2 luồng cùng lúc. Do đó:
  - Máy có thể xử lý **8 luồng song song**, giúp tăng hiệu quả trong các tác vụ sử dụng đa luồng như trình duyệt, IDE, ứng dụng văn phòng...
  - Tuy nhiên, hiệu năng vẫn bị giới hạn nếu ứng dụng cần nhiều nhân vật lý thực thụ.

- **RAM**:
  - Dung lượng RAM 8GB là mức phổ thông.
  - Qua ảnh Task Manager: **RAM đang sử dụng 87%**, tức là **gần đầy** → nếu mở thêm ứng dụng nặng (IDE, máy ảo, trình duyệt nhiều tab), hệ thống sẽ bắt đầu **dùng bộ nhớ ảo (paging)**, làm **giảm hiệu năng rõ rệt**.

- **GPU**:
  - Không có GPU rời nên hiệu năng xử lý đồ họa (3D, render video, AI) sẽ rất giới hạn.
  - GPU tích hợp Iris Xe phù hợp với các tác vụ nhẹ như xem video HD, làm việc văn phòng.

- **Ứng dụng tiêu tốn tài nguyên** (theo ảnh):
  - Cốc Cốc: ~500MB RAM.
  - VS Code: ~220MB RAM.
  - Ngoài ra, có nhiều tiến trình nền nhỏ (Runtime Broker, Search, System...) cùng tiêu tốn RAM.
**Phân tích:**

- RAM đang gần đầy (87%), có thể gây chậm khi chạy thêm ứng dụng nặng → **bộ nhớ là điểm nghẽn**.
- CPU hoạt động nhẹ (11%), chưa bị quá tải.
- GPU không được sử dụng hiệu quả trong các tác vụ hiện tại.
- Trình duyệt Cốc Cốc chiếm nhiều tài nguyên (chạy nhiều tab hoặc tiến trình phụ).
- Hệ thống hiện đang xử lý đa tiến trình khá nhiều (Cốc Cốc, Edge, Runtime Broker...), phù hợp với cấu hình đa luồng.

**Kết luận:**

- Máy phù hợp để chạy các ứng dụng đa luồng nhẹ như lập trình, duyệt web nhẹ, làm việc văn phòng.
- Không phù hợp chạy đa tiến trình nặng (như video render, AI training).
- Nên nâng cấp RAM nếu thường xuyên dùng nhiều ứng dụng cùng lúc.
---
##  2. 12 bài toán CNTT sử dụng đa tiến trình hoặc đa luồng

| STT | Bài toán                              | Thread / Process |
|-----|----------------------------------------|------------------|
| 1   | Server web xử lý nhiều client cùng lúc | Thread           |
| 2   | Chương trình chat thời gian thực       | Thread           |
| 3   | Tải nhiều file cùng lúc (Download Manager) | Thread       |
| 4   | Encode nhiều video song song           | Process          |
| 5   | Phân tích log server (log analyzer)    | Process          |
| 6   | Game online đa người chơi              | Thread           |
| 7   | MapReduce xử lý dữ liệu lớn            | Process          |
| 8   | Render đồ họa 3D                        | Thread           |
| 9   | Đào coin (mining)                      | Process          |
| 10  | Crawl dữ liệu từ web                   | Process          |
| 11  | Huấn luyện AI / Deep Learning          | Cả hai           |
| 12  | IDE biên dịch + kiểm tra lỗi realtime  | Thread           |

---

##  3. Bảng so sánh khi nào dùng Thread, khi nào dùng Process
![Ảnh](/images/anhbaitap55.jpg)
---

##  4. Tìm hiểu cách ChatGPT huấn luyện bằng hệ thống phân tán

**Tóm tắt:**

- **Dữ liệu huấn luyện**: hàng trăm tỷ từ (token).
- **Data Parallelism**: chia dữ liệu ra nhiều phần, mỗi GPU xử lý một phần giống nhau.
- **Model Parallelism**: chia mô hình lớn (GPT-3, GPT-4) ra các phần, mỗi GPU xử lý một tầng (layer).
- Cần cụm hàng nghìn GPU → tốc độ nhanh và đảm bảo bộ nhớ đủ lớn.
- **Hệ thống training**: sử dụng các supercomputer như V100, A100 GPU cluster (do Microsoft Azure cung cấp cho OpenAI).
- Mỗi mô hình có hàng trăm tỷ tham số → nếu không dùng hệ thống phân tán thì không thể huấn luyện.

**Tài liệu tham khảo:**

- OpenAI Blog: [https://openai.com/research](https://openai.com/research)
- Arxiv Paper GPT-3: [https://arxiv.org/abs/2005.14165](https://arxiv.org/abs/2005.14165)
- Microsoft về hạ tầng cho OpenAI: [https://blogs.microsoft.com](https://blogs.microsoft.com)