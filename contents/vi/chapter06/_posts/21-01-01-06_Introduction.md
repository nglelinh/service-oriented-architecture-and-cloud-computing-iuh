---
layout: post
title: 06 Giới thiệu — bài ứng dụng tùy chọn
chapter: '06'
order: 1
owner: Nguyen Le Linh
lang: vi
categories:
- chapter06
---

Ở Chương 05, bạn đã làm quen với động cơ cốt lõi của Apache Spark. Nhưng sức mạnh thực sự khiến Spark trở thành tiêu chuẩn trong các nền tảng dữ liệu lớn nằm ở **hệ sinh thái thống nhất** xây trên cùng động cơ đó: từ truy vấn bảng bằng **Spark SQL**, xử lý dòng sự kiện vô tận bằng **Structured Streaming**, đến học máy phân tán với **MLlib** — tất cả đều chia sẻ chung mô hình DataFrame và bộ tối ưu Catalyst.

Dữ liệu tĩnh trong kho giống như một bức ảnh chụp quá khứ; dữ liệu luồng (streaming data) giống như một đoạn phim trực tiếp đang diễn ra từng mili-giây. Chương 06 đưa bạn từ xử lý theo lô (batch) sang **kiến trúc hướng sự kiện và xử lý thời gian thực** — nơi mỗi cú click, giao dịch thanh toán hay cảm biến IoT đều kích hoạt ngay lập tức chuỗi tính toán và suy luận ML ở biên phục vụ (serving edge).

---

## Bản đồ kiến thức nền: So sánh ba nhịp độ xử lý dữ liệu trên Đám mây

1. **Xử lý theo lô (Batch — hàng giờ/ngày):** Tối ưu thông lượng cực đại và chi phí thấp nhất (báo cáo tài chính, huấn luyện lại mô hình lớn), nhưng độ trễ cao.
2. **Vi lô (Micro-batch — 100 ms đến vài giây):** Structured Streaming gom sự kiện thành từng lô cực nhỏ, tái sử dụng cùng một engine tối ưu SQL/DataFrame với batch.
3. **Hướng sự kiện thuần túy (Event-Driven Streaming — < 50 ms):** Xử lý ngay từng bản ghi qua message broker (Kafka/Pulsar) kết hợp kho trạng thái độ trễ thấp — bắt buộc cho chống gian lận, định giá động và cảnh báo sự cố.

---

## Bản đồ kiến thức nền: Ba trụ cột của Hệ sinh thái Spark (Chương 06)

1. **Spark SQL & DataFrame**:
   Đưa truy vấn SQL quen thuộc lên dữ liệu phân tán hàng terabyte, tự động tối ưu thứ tự lọc, chiếu và nối bảng nhờ bộ tối ưu Catalyst thay vì bắt lập trình viên tự viết tay từng bước RDD.
2. **Structured Streaming (Xử lý luồng như bảng vô hạn)**:
   Thay vì coi dữ liệu luồng là một thế giới hoàn toàn khác với dữ liệu lô, Structured Streaming mô hình hóa dòng sự kiện như một *bảng liên tục nối thêm dòng mới*, cho phép dùng chung cú pháp DataFrame kèm cơ chế checkpoint và watermark.
3. **Cầu nối sang Học máy & Tầng phục vụ trực tuyến (Online Serving)**:
   Spark xuất sắc ở việc biến đổi dữ liệu, tính đặc trưng và cập nhật **Feature Store**, nhưng không trực tiếp đứng ở đầu API trả lời người dùng trong 20 ms — phần đó được bàn giao cho tầng phục vụ mô hình chuyên biệt.

### Khung giải thích (nên đọc chậm): So sánh ngân sách độ trễ giữa nhánh Streaming và nhánh Serving trực tuyến

Giả sử ứng dụng thương mại điện tử cần gợi ý sản phẩm ngay khi người dùng vừa xem 3 món đồ mới:

| Nhánh kiến trúc | Thành phần | Độ trễ điển hình | Vai trò cụ thể |
| :--- | :--- | :---: | :--- |
| **Nhánh Luồng (Nearline)** | Kafka → Spark Structured Streaming → Redis Feature Store | $$2\text{–}10\text{ giây}$$ | Liên tục cập nhật vector sở thích gần nhất của người dùng vào bộ nhớ đệm |
| **Nhánh Trực tuyến (Online)** | API Gateway → Recommendation Service → đọc Redis + gọi Model Server | $$15\text{–}40\text{ ms}$$ | Trả danh sách gợi ý ngay khi người dùng tải trang mà không chờ Spark chạy |

Tách đúng hai nhánh này giúp hệ thống vừa phản ứng nhanh với hành vi mới nhất của người dùng, vừa giữ vững độ trễ p99 mượt mà ở tầng API.

> **Lưu ý lộ trình:** Phần lý thuyết bắt buộc của **Chương 06 (Hệ sinh thái Spark)** hiện được trình bày bằng tiếng Anh. Lộ trình tiếng Việt ở chương này bổ sung **một bài ứng dụng tùy chọn**.
>
> - [06-05 Spark thời gian thực, hướng sự kiện và tầng phục vụ]({{ site.baseurl }}{% multilang_post_url contents/chapter06/21-01-01-06_05_Realtime_Event_Driven_ML %})
> - [Bản tiếng Anh của chương]({{ site.baseurl }}/contents/en/chapter06/)

---

### Ví dụ thực tế cầm tay chỉ việc: Chọn nhịp độ xử lý cho 3 bài toán nghiệp vụ khác nhau

| Bài toán thực tế | Ngân sách độ trễ | Kiến trúc phù hợp | Lý do cốt lõi |
| :--- | :--- | :--- | :--- |
| **Chặn thẻ tín dụng gian lận** | $$< 100	ext{ ms}$$ | Event-driven + Online Feature Store | Chậm 5 giây là giao dịch đã hoàn tất! |
| **Cập nhật bảng xếp hạng xu hướng** | $$5	ext{–}30	ext{ giây}$$ | Micro-batch (Spark Structured Streaming) | Cân bằng giữa độ tươi và chi phí cụm |
| **Đối soát sổ cái kế toán cuối ngày** | $$1	ext{–}6	ext{ giờ}$$ | Batch (Parquet / Lakehouse) | Cần tính chính xác tuyệt đối 100%, không cần tức thì |

### Khung giải thích: Ba câu hỏi trước khi chuyển một hệ thống sang thời gian thực (nên đọc chậm)

1. **Độ trễ thấp có thực sự tạo ra giá trị kinh doanh không?** Hệ thống streaming đắt và khó vận hành hơn batch từ 3–5 lần — chỉ xây thời gian thực khi quyết định chậm vài phút thực sự gây mất tiền hoặc mất trải nghiệm.
2. **Trạng thái (state) được lưu và khôi phục ở đâu khi máy chủ khởi động lại?** Luôn cần cơ chế checkpoint bền vững để không mất cửa sổ đếm đang chạy dở.
3. **Hệ thống chịu được đỉnh lưu lượng gấp 10 lần không?** Đưa hàng đợi sự kiện (Kafka) vào giữa nhà sản xuất và người tiêu thụ giúp hấp thụ cơn bão lưu lượng mà không làm sập dịch vụ ML phía sau.

**Kiểm tra nhanh.** Giữa bài toán *cảnh báo máy chủ quá nhiệt (CPU > 95% liên tục 1 phút)* và bài toán *gửi email tổng kết chi tiêu hàng tháng*, bài toán nào cần kiến trúc streaming và bài toán nào chỉ cần batch?

---

## Video tham khảo

- **Xem trước:** [Event-Driven Architecture(EDA) vs. Request/Response (RR): System Design Explained](https://www.youtube.com/watch?v=7fkS-18KBlw) — Confluent, an IBM Company (tiếng Anh). So sánh kiến trúc hướng sự kiện với mô hình yêu cầu–phản hồi (request/response), phục vụ nội dung bài tùy chọn.
- [What is a Feature Store for Machine Learning?](https://www.youtube.com/watch?v=DESBDukN9gw) — Hopsworks (tiếng Anh). Giải thích vai trò của feature store trong nhánh phục vụ trực tuyến tách khỏi Spark driver.
- [Spark Streaming APIs   DStream Vs Structured Streaming](https://www.youtube.com/watch?v=YDbYmaMSEic) — Learning Journal (tiếng Anh). Phân biệt DStream và Structured Streaming, bổ trợ cho phần xử lý luồng trong bản tiếng Anh.
