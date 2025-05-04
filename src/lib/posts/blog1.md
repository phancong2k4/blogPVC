---
title: Hệ Thống Phân Tán Là Gì?
date: 2025-04-29
description: Tổng quan hệ thống phân tán, các khái niệm và kiến trúc phổ biến
---

> **Tác giả**: Phan Văn Công  
> **Ngày đăng**: 29/04/2025

![Hệ thống phân tán](/static/images/anh1.png)

## 🧠 Hệ thống phân tán là gì?

Hệ thống phân tán (Distributed System) là tập hợp các máy tính độc lập hoạt động như một hệ thống thống nhất đối với người dùng. Các thành phần có thể phân tán về mặt địa lý và giao tiếp với nhau qua mạng.

Ví dụ điển hình: Google, Facebook, Amazon đều sử dụng các hệ thống phân tán để xử lý và lưu trữ dữ liệu trên quy mô toàn cầu.

---

## 📱 Các ứng dụng của hệ thống phân tán

- Dịch vụ web như Google Search, Gmail, Facebook.
- Lưu trữ phân tán: Google Drive, Dropbox.
- Thương mại điện tử: Shopee, Tiki, Lazada.
- Tính toán hiệu năng cao: Mô phỏng vật lý, dữ liệu lớn.
- Mạng xã hội, trò chơi trực tuyến, ứng dụng IoT.

---

## 🔑 Các khái niệm chính

| Thuật ngữ | Giải thích |
|----------|-----------|
| **Scalability** | Khả năng mở rộng hệ thống khi số lượng người dùng hoặc dữ liệu tăng lên. |
| **Fault Tolerance** | Khả năng tiếp tục hoạt động khi có thành phần bị lỗi. |
| **Availability** | Mức độ hệ thống có thể truy cập và hoạt động ổn định. |
| **Transparency** | Ẩn đi sự phức tạp của hệ phân tán với người dùng (ví dụ: vị trí, lỗi, truy cập). |
| **Concurrency** | Khả năng xử lý đồng thời nhiều yêu cầu. |
| **Parallelism** | Thực thi các tác vụ cùng lúc để cải thiện hiệu năng. |
| **Openness** | Tính mở, tương thích với nhiều nền tảng và giao thức. |
| **Vertical Scaling** | Tăng cường phần cứng máy chủ (CPU, RAM, SSD…). |
| **Horizontal Scaling** | Thêm nhiều máy chủ để chia tải. |
| **Load Balancer** | Phân phối lưu lượng đến các máy chủ để tối ưu hóa hiệu suất. |
| **Replication** | Nhân bản dữ liệu ở nhiều nơi để tăng tính sẵn sàng và hiệu suất. |

---

## 📌 Ví dụ thực tế: YouTube

- **Scalability**: YouTube mở rộng qua việc dùng nhiều máy chủ để xử lý video hàng tỷ lượt xem mỗi ngày.
- **Fault Tolerance**: Nếu một server chết, server khác tiếp quản.
- **Availability**: Dịch vụ gần như luôn sẵn sàng (24/7).
- **Load Balancer**: Phân chia người dùng đến các máy chủ khác nhau.
- **Replication**: Video được lưu ở nhiều data center toàn cầu.

---

## 🏗️ Kiến trúc hệ thống phân tán

Một số mô hình kiến trúc chính:

1. **Client-Server**: Máy khách gửi yêu cầu, máy chủ xử lý.
2. **Peer-to-Peer (P2P)**: Các nút giao tiếp trực tiếp với nhau.
3. **Three-tier Architecture**: Gồm giao diện người dùng, lớp logic nghiệp vụ, và cơ sở dữ liệu.
4. **Microservices**: Ứng dụng chia thành các dịch vụ nhỏ độc lập.
5. **Event-driven Architecture**: Các thành phần giao tiếp qua sự kiện.
6. **Service-Oriented Architecture (SOA)**.

![Kiến trúc Microservices](/static/images/images.jpg)

---

## 📚 Tham khảo

- [Wikipedia - Distributed Systems](https://en.wikipedia.org/wiki/Distributed_computing)
- [Slide bài giảng môn Hệ thống phân tán]
- [Tài liệu GitHub Markdown](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)

---

👉 Sau khi hoàn tất, bạn có thể **triển khai blog lên Vercel**, sau đó **gửi link blog lên Canvas (Bài tập 1)**.

---

**Phan Văn Công**  
Sinh viên năm 3, Khoa CNTT, Đại học Phenikaa  
