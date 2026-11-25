---
layout: post
title: 03 Giới thiệu — bài ứng dụng tùy chọn
chapter: '03'
order: 1
owner: Nguyen Le Linh
lang: vi
categories:
- chapter03
---

Hãy tưởng tượng một nhà bếp lớn phục vụ 500 thực khách mỗi giờ. Nếu bồi bàn đứng chắn ngay cửa bếp chờ đầu bếp nấu xong từng món rồi mới đi nhận bàn tiếp theo, cả nhà hàng sẽ tắc cứng. Ngược lại, nếu chỉ có mỗi tấm bảng kẹp hóa đơn mà không có quản lý phân chia bếp trưởng, bếp phụ và lò nướng, các món sẽ chồng chéo và cháy khét. Trong điện toán đám mây, **mô hình tính toán (computing paradigms)** quyết định cách các dịch vụ trao đổi công việc, còn **bộ lập lịch cụm (cluster scheduler)** đóng vai trò người điều phối tài nguyên toàn cục.

## Kiến thức cần nhớ: Ba câu hỏi lớn của Chương 03

1. **Client–Server (Đồng bộ) vs. Event-Driven (Bất đồng bộ)**:
   Khi nào bên gọi cần giữ kết nối chờ kết quả ngay (REST, gRPC) và khi nào nên phát sự kiện vào hàng đợi (Kafka, CloudEvents) để tách rời thời gian xử lý và cô lập lỗi?
2. **Ngang hàng (P2P) & Truyền luồng (Streaming)**:
   Cách các nút trao đổi dữ liệu trực tiếp hoặc duy trì kênh truyền liên tục (gRPC streaming) cho dữ liệu thời gian thực như định vị GPS, bảng giá chứng khoán hay nhật ký hệ thống.
3. **Quản trị tài nguyên cụm (YARN → Kubernetes)**:
   Làm sao tách rời lớp *cấp phát CPU/RAM/GPU* khỏi lớp *logic ứng dụng* để hàng trăm nhóm phát triển dùng chung một cụm máy chủ một cách công bằng, tự phục hồi khi nút bị hỏng.

### Giải thích: Phân biệt nhanh 3 mô hình giao tiếp trên cùng một bài toán "Đặt vé xem phim"

Xét luồng người dùng bấm **Thanh toán vé** trên ứng dụng rạp chiếu phim:

| Bước nghiệp vụ | Mô hình phù hợp | Lý do kỹ thuật |
| :--- | :---: | :--- |
| 1. Giữ ghế `G12` và trừ tiền thẻ | **Đồng bộ (gRPC / REST)** | Người dùng phải biết *ngay lập tức* ghế có còn trống và thẻ có đủ tiền hay không trước khi rời màn hình |
| 2. Gửi mã QR vé qua email & cộng điểm thành viên | **Hướng sự kiện (Event-Driven)** | Không cần xong ngay trong 100 ms; nếu máy chủ email chậm 30 giây, vé của khách vẫn đã được xác nhận an toàn |
| 3. Cuối ngày tổng hợp doanh thu toàn cụm rạp | **Bộ lập lịch cụm (YARN / Kubernetes Job)** | Tác vụ xử lý lô kéo dài 15 phút, cần xin cấp phát tạm thời 16 vCPU và trả lại cụm ngay khi xong |

Nhận diện đúng bản chất từng bước ngay từ khâu thiết kế giúp tránh lỗi phổ biến nhất của người mới: dùng đồng bộ cho mọi thứ (gây sập dây chuyền) hoặc dùng hàng đợi cho cả thao tác cần trả lời ngay (gây trải nghiệm giật lag và khó gỡ lỗi).

> **Lưu ý lộ trình:** Phần lý thuyết bắt buộc của **Chương 03 (Mô hình tính toán và YARN)** hiện được trình bày bằng tiếng Anh. Lộ trình tiếng Việt ở chương này bổ sung **một bài ứng dụng tùy chọn**.
>
> - [03-02 Mô hình hướng sự kiện, gRPC và bộ lập lịch cụm]({{ site.baseurl }}{% multilang_post_url contents/chapter03/21-01-01-03_02_Event_Driven_and_gRPC %})
> - [Bản tiếng Anh của chương]({{ site.baseurl }}/contents/en/chapter03/)

## Video tham khảo

- **Xem trước:** [gRPC in 5 minutes \| Eric Anderson & Ivy Zhuang, Google](https://www.youtube.com/watch?v=njC24ts24Pg) — gRPC (tiếng Anh). Giới thiệu gRPC trong năm phút từ chính nhóm phát triển dự án.
- [What is the Cloud Events Specification & Why You Should Care!](https://www.youtube.com/watch?v=Sjw0PFwTvek) — James Eastham (tiếng Anh). Trình bày đặc tả CloudEvents — chuẩn hợp đồng bất đồng bộ được đặt cạnh gRPC trong bài tùy chọn.
- [Event-Driven Architecture(EDA) vs. Request/Response (RR): System Design Explained](https://www.youtube.com/watch?v=7fkS-18KBlw) — Confluent, an IBM Company (tiếng Anh). So sánh kiến trúc hướng sự kiện với mô hình yêu cầu–phản hồi (request/response).
