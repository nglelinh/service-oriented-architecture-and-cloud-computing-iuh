---
layout: post
title: 00 Lộ trình tiếng Việt
chapter: '00'
order: 1
owner: Nguyen Le Linh
lang: vi
categories:
- chapter00
---

Mỗi lần bạn mở ứng dụng gọi xe, quét mã QR thanh toán hay nhấn "Đặt hàng" trong ngày siêu sale 11/11, yêu cầu nhỏ bé từ chiếc điện thoại ấy không chạy vào một máy chủ duy nhất. Trong vòng chưa đầy 200 mili-giây, nó đi qua DNS, tường lửa biên, bộ cân bằng tải, hàng chục vi dịch vụ độc lập (xác thực, tồn kho, thanh toán, thông báo), ghi nhận sự kiện vào hàng đợi phân tán và lưu xuống cụm cơ sở dữ liệu đặt ở nhiều vùng khả dụng khác nhau. **Kiến trúc hướng dịch vụ (SOA) và Điện toán đám mây** chính là bộ ngôn ngữ kỹ thuật giải thích cách những hệ thống khổng lồ ấy được thiết kế, vận hành và giữ vững độ tin cậy 24/7.

---

## Kiến thức cần nhớ: Toàn cảnh khóa học SOA & Điện toán đám mây

### 1. Hạ tầng đám mây & Mạng phân tán (Chương 1–2)
Điện toán đám mây biến máy chủ vật lý, mạng và lưu trữ thành tài nguyên lập trình được qua API (VPC, subnet, TLS, autoscaling), đi kèm hệ thống **quan sát (observability)** bằng logs, metrics và distributed traces để nhìn xuyên suốt hành trình của một yêu cầu.
- **Câu hỏi lớn:** Làm sao cấp phát, cô lập mạng và giám sát hàng nghìn tiến trình chạy phân tán trên nhiều trung tâm dữ liệu?
- **Ứng dụng thực tế:** Hạ tầng AWS/GCP/Azure, thiết kế mạng trường học/doanh nghiệp, SRE và vận hành 99.99%.

### 2. Giao tiếp dịch vụ & Xử lý dữ liệu phân tán (Chương 3–6)
Các dịch vụ trao đổi dữ liệu qua hai nhịp: **đồng bộ** (REST/HTTP, gRPC) khi cần phản hồi ngay và **bất đồng bộ hướng sự kiện** (Kafka, CloudEvents) để tách rời tải đỉnh. Ở tầng dữ liệu lớn, mô hình MapReduce, Apache Spark và luồng thời gian thực xử lý hàng terabyte log mỗi giờ.
- **Câu hỏi lớn:** Khi nào dùng gRPC thay vì hàng đợi sự kiện? Làm sao cân bằng độ trễ với thông lượng?
- **Ứng dụng thực tế:** Cổng thanh toán, sàn thương mại điện tử, hệ thống chống gian lận thời gian thực.

### 3. Lưu trữ phân tán, Serverless & Ảo hóa thế hệ mới (Chương 7–8)
Dữ liệu không còn nằm trong một máy SQL đơn lẻ mà phân mảnh trên CSDL serverless, NewSQL đồng thuận phân tán (Raft/Paxos) và CSDL vector cho AI; trong khi tầng tính toán tiến hóa từ máy ảo nặng nề sang container, **microVM (Firecracker)** và **WebAssembly (WASM)**.
- **Câu hỏi lớn:** Định lý CAP đánh đổi gì khi mạng bị chia cắt? Làm sao khởi chạy hàm serverless trong vài mili-giây?
- **Ứng dụng thực tế:** CSDL đa vùng, ứng dụng RAG/AI, nền tảng FaaS (AWS Lambda, Cloudflare Workers).

---

Lộ trình tiếng Việt của môn học **Kiến trúc hướng dịch vụ và Điện toán đám mây** hiện đang được xây dựng theo từng giai đoạn.

Hiện nay, **Chương 01 — Nền tảng điện toán đám mây** đã hoàn thiện bản tiếng Việt (gồm sáu bài lý thuyết và các bài ứng dụng tùy chọn giai đoạn 2022–2026). Các chương từ 02 đến 14 **chưa có bản dịch đầy đủ cho phần lý thuyết**; mỗi chương hiện có **một bài ứng dụng tùy chọn** bằng tiếng Việt (về Kubernetes/service mesh, serverless, gRPC, kiến trúc hướng sự kiện, khả năng quan sát, đa đám mây, phục vụ AI...) kèm liên kết dẫn về bản tiếng Anh.

- [Mở Chương 01 — Nền tảng điện toán đám mây]({{ site.baseurl }}/contents/vi/chapter01/)
- Bộ tài liệu tiếng Anh đầy đủ 14 chương vẫn là lộ trình lý thuyết chính. Ở các bài tùy chọn, bạn có thể dùng nút **Chuyển sang tiếng Anh** để đối chiếu trực tiếp cặp bài EN/VI (khớp nhau theo `chapter` và `order`).

---

## Ví dụ làm từng bước & Giải thích

### Ví dụ làm từng bước 1: Tính nhanh độ khả dụng (Availability) của chuỗi dịch vụ nối tiếp

Giả sử một yêu cầu "Thanh toán" phải gọi nối tiếp **3 dịch vụ độc lập**: Cổng API ($$99.9\%$$), Dịch vụ Đơn hàng ($$99.9\%$$) và Cổng Thanh toán ($$99.9\%$$). Độ khả dụng tổng của toàn chuỗi là tích xác suất cả 3 cùng sống:

$$A_{	ext{tổng}} = 0.999 	imes 0.999 	imes 0.999 = 0.997003 pprox 99.7\%$$

- Một dịch vụ $$99.9\%$$ chỉ gián đoạn tối đa **~43.8 phút/tháng**.
- Chuỗi 3 dịch vụ nối tiếp tụt xuống $$99.7\%$$, tức gián đoạn tới **~2.16 giờ/tháng** (gấp 3 lần!).
- **Ngược lại**, nếu bạn chạy **2 bản sao dự phòng song song** cho cùng một dịch vụ $$99.9\%$$, xác suất cả 2 cùng chết chỉ còn $$(1 - 0.999)^2 = 10^{-6}$$, đẩy độ khả dụng lên **$$99.9999\%$$** (31 giây/năm)!

### Ví dụ làm từng bước 2: Trace ngân sách độ trễ 200 ms của một yêu cầu đặt hàng

| Chặng xử lý | Giao thức / Thành phần | Độ trễ (ms) | Tỷ trọng |
|---|---|---|---|
| 1. Phân giải DNS + bắt tay TLS biên | DNS + TLS 1.3 | 25 ms | 12.5% |
| 2. API Gateway xác thực JWT | HTTPS / REST | 10 ms | 5.0% |
| 3. Order Service gọi Inventory Service | gRPC nội bộ | 15 ms | 7.5% |
| 4. Ghi đơn hàng vào CSDL chính | SQL ACID | 35 ms | 17.5% |
| 5. Đẩy sự kiện `OrderCreated` vào Kafka | Async Event | 5 ms | 2.5% |
| **Tổng thời gian phản hồi người dùng** | **Đồng bộ + Bất đồng bộ** | **90 ms** | **Đạt SLA < 200 ms!** |

*(Lưu ý: Gửi email hóa đơn và cập nhật điểm thưởng mất 400 ms nhưng được tách sang consumer bất đồng bộ sau bước 5 nên người dùng không phải chờ!)*

### Giải thích: Cách đọc khóa học SOA & Cloud theo tư duy kiến trúc sư

#### Bước 1: Luôn bắt đầu từ yêu cầu phi chức năng (SLA, độ trễ, chi phí)
Mọi quyết định kiến trúc đều trả lời câu hỏi định lượng: hệ thống chịu bao nhiêu yêu cầu/giây (RPS), chấp nhận độ trễ p99 bao nhiêu mili-giây, và chi phí tháng tối đa là bao nhiêu?

#### Bước 2: Phân biệt rạch ròi mặt phẳng điều khiển (Control Plane) và mặt phẳng dữ liệu (Data Plane)
Khi đọc về Cloud, Kubernetes hay Kafka, hãy luôn hỏi: thành phần nào đang **điều phối/cấu hình** (Control Plane) và thành phần nào đang **trực tiếp đẩy gói tin/byte dữ liệu** (Data Plane)?

#### Bước 3: Chấp nhận lỗi mạng là chuyện bình thường (Design for Failure)
Trong hệ phân tán, cáp mạng đứt, gói tin trễ và nút máy chủ khởi động lại xảy ra hàng ngày. Timeout, retry có backoff, idempotency và cô lập lỗi là yêu cầu bắt buộc, không phải phần phụ.

#### Bước 4: Nối liền lý thuyết cổ điển với công cụ hiện đại
Đừng học thuộc tên sản phẩm AWS/GCP. Một khi nắm vững 5 đặc trưng NIST, mạng VPC, gRPC vs. Event, CAP và observability, bạn có thể chuyển đổi giữa mọi nhà cung cấp đám mây chỉ trong vài ngày.

## Video tham khảo

- **Xem trước:** [Cloud Computing - Điện toán đám mây là gì? \| Hiểu rõ trong 5 phút \| TING3S](https://www.youtube.com/watch?v=5JLBwLilDVE) — TING 3S (tiếng Việt). Giúp nắm nhanh khái niệm điện toán đám mây bằng tiếng Việt trước khi bước vào Chương 01.
- [Điện toán đám mây là gì? 3 mô hình dịch vụ của điện toán đám mây](https://www.youtube.com/watch?v=wb-ebOI3IXM) — VDI (tiếng Việt). Hệ thống lại ba mô hình dịch vụ (IaaS, PaaS, SaaS) được trình bày chi tiết trong Chương 01.
- [What is Cloud Computing? \| Amazon Web Services](https://www.youtube.com/watch?v=mxT233EdY5c) — Amazon Web Services (tiếng Anh). Đối chiếu định nghĩa điện toán đám mây theo cách trình bày của AWS, chuẩn bị cho việc đọc tiếp tài liệu tiếng Anh ở các chương sau.
