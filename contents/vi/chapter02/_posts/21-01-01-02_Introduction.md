---
layout: post
title: 02 Giới thiệu — bài ứng dụng tùy chọn
chapter: '02'
order: 1
owner: Nguyen Le Linh
lang: vi
categories:
- chapter02
---

Trên một chiếc máy tính đơn lẻ, chương trình chạy tuần tự rất dễ đoán: nếu máy mất điện, cả hệ thống dừng cùng một lúc; nếu hai hàm gọi nhau, đồng hồ hệ thống luôn cho biết chính xác việc nào xảy ra trước. Nhưng ngay khi bạn tách ứng dụng ra chạy trên **nhiều máy chủ nối mạng với nhau**, mọi giả định êm đềm đó sụp đổ. Leslie Lamport từng đúc kết một câu nổi tiếng: *"Hệ thống phân tán là nơi một máy tính mà bạn thậm chí không biết nó tồn tại có thể khiến chính máy tính của bạn không thể sử dụng được."*

## Bản đồ kiến thức nền: Ba đặc trưng cốt lõi của Hệ thống phân tán

Trước khi đi sâu vào các kỹ thuật quan sát và chịu lỗi, hãy nắm vững ba tính chất nền tảng định nghĩa mọi hệ thống phân tán:

1. **Tính tương tranh (Concurrency)**:
   Hàng trăm máy chủ và hàng nghìn luồng cùng xử lý dữ liệu tại một thời điểm mà không chia sẻ bộ nhớ chung. Điều này đem lại khả năng mở rộng lớn nhưng cũng kéo theo bài toán tranh chấp tài nguyên (race condition) và đồng bộ trạng thái.
2. **Không có đồng hồ toàn cục (No Global Clock)**:
   Đồng hồ vật lý trên các máy chủ luôn trôi lệch nhau vài mili-giây dù đã đồng bộ qua NTP. Do đó, không thể chỉ dựa vào dấu thời gian (timestamp) cục bộ để kết luận sự kiện nào là nguyên nhân của sự kiện nào — ta phải dùng đồng hồ logic (Lamport/Vector clock) hoặc mã truy vết nhân quả (`trace_id`).
3. **Lỗi độc lập (Independent Failures)**:
   Một máy chủ có thể treo, một cáp mạng có thể đứt gây chia cắt mạng (network partition), trong khi các máy khác vẫn hoạt động bình thường. Hệ thống bắt buộc phải chấp nhận lỗi cục bộ thay vì để một mắt xích nhỏ kéo sập toàn bộ.

### Khung giải thích (nên đọc chậm): Minh họa "không có đồng hồ toàn cục" qua bài toán ghi log hai máy chủ

Giả sử **Máy A (API Gateway)** nhận yêu cầu chuyển khoản và gọi sang **Máy B (Payment Service)**. Do lệch đồng hồ NTP, đồng hồ của Máy A chạy nhanh hơn Máy B đúng **15 ms**, còn thời gian truyền gói tin qua mạng nội bộ là **4 ms**:

| Sự kiện thực tế (thời gian tuyệt đối) | Thời gian ghi trên log cục bộ | Chuyện gì gây hiểu nhầm khi đọc log? |
| :--- | :---: | :--- |
| 1. Máy A gửi yêu cầu thanh toán tại $$t = 100\text{ ms}$$ | Máy A ghi: `10:00:00.115` | Đồng hồ A nhanh hơn 15 ms |
| 2. Máy B nhận yêu cầu tại $$t = 104\text{ ms}$$ | Máy B ghi: `10:00:00.104` | Nhìn log thô thấy **Máy B nhận tin TRƯỚC khi Máy A gửi 11 ms!** |
| 3. Máy B xử lý xong sau 10 ms ($$t = 114\text{ ms}$$) | Máy B ghi: `10:00:00.114` | Nhìn log thấy Máy B xử lý xong *vẫn sớm hơn* lúc Máy A bắt đầu gửi! |

Đây chính là lý do bài ứng dụng tùy chọn của Chương 02 giới thiệu **Distributed Tracing (OpenTelemetry)**: thay vì sắp xếp log theo đồng hồ vật lý dễ đánh lừa, mỗi lời gọi mang theo cặp `(trace_id, parent_span_id)` để dựng lại chính xác cây nhân quả A → B bất chấp đồng hồ hai máy có lệch nhau bao nhiêu mili-giây.

> **Lưu ý lộ trình:** Phần lý thuyết bắt buộc của **Chương 02 (Hệ thống phân tán)** hiện được trình bày bằng tiếng Anh. Lộ trình tiếng Việt ở chương này bổ sung **một bài ứng dụng tùy chọn** (giai đoạn 2022–2026) thay vì dịch lại toàn bộ phần lý thuyết.
>
> - [02-02 Quan sát hệ thống phân tán và ứng dụng hiện đại]({{ site.baseurl }}{% multilang_post_url contents/chapter02/21-01-01-02_02_Observability_and_Distributed_Applications %})
> - [Bản tiếng Anh của chương]({{ site.baseurl }}/contents/en/chapter02/)

## Video tham khảo

- **Xem trước:** [Distributed System - Lesson 01 - Hiểu Hệ thống Phân tán](https://www.youtube.com/watch?v=IqAk3_PoLwA) — Mindstone Organization (tiếng Việt). Giới thiệu khái niệm hệ thống phân tán bằng tiếng Việt trước khi đọc phần lý thuyết tiếng Anh.
- [Explaining Distributed Systems Like I'm 5](https://www.youtube.com/watch?v=CESKgdNiKJw) — HashiCorp, an IBM Company (tiếng Anh). Giải thích ngắn gọn sự khác biệt giữa hệ thống đơn máy và hệ thống phân tán.
- [OpenTelemetry: OpenTelemetry in Five Minutes \| Project Lightning Talk](https://www.youtube.com/watch?v=l8xiNOCIdLY) — CNCF [Cloud Native Computing Foundation] (tiếng Anh). Giới thiệu nhanh trong năm phút về OpenTelemetry — công cụ quan sát hệ thống được trình bày trong bài tùy chọn.
