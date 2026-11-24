---
layout: post
title: 07 Giới thiệu — bài ứng dụng tùy chọn
chapter: '07'
order: 1
owner: Nguyen Le Linh
lang: vi
categories:
- chapter07
---

Khi ứng dụng chỉ có vài nghìn người dùng, một máy chủ cơ sở dữ liệu quan hệ duy nhất (PostgreSQL/MySQL) hoạt động gọn gàng và tin cậy. Nhưng khi dữ liệu phình lên hàng chục terabyte trải khắp nhiều trung tâm dữ liệu, một định luật bất khả xâm phạm của hệ phân tán lập tức xuất hiện: **Định lý CAP**. Nếu dây cáp nối giữa hai trung tâm dữ liệu bị đứt, bạn bắt buộc phải chọn một trong hai: hoặc tạm từ chối yêu cầu để giữ dữ liệu thống nhất tuyệt đối (**CP**), hoặc tiếp tục phục vụ mọi người và chấp nhận hai bên tạm thời lệch nhau vài giây (**AP**).

Trong kiến trúc dịch vụ, tầng ứng dụng có thể dễ dàng nhân bản thành 50 container không trạng thái (*stateless*), nhưng **cơ sở dữ liệu (stateful)** mới là nơi quyết định hệ thống đứng vững hay sụp đổ khi lưu lượng tăng vọt! Chương 07 mở rộng tư duy lưu trữ từ CSDL quan hệ đơn nút sang ba công nghệ trụ cột của giai đoạn 2022–2026: **CSDL Serverless**, **SQL phân tán** và **CSDL Vector cho ứng dụng AI/RAG**.

---

## Bản đồ kiến thức nền: Bức tranh chọn lựa tầng Lưu trữ Đám mây

1. **Tách Tính toán khỏi Lưu trữ (Storage–Compute Disaggregation):** Cho phép mở rộng dung lượng đĩa và sức mạnh CPU theo hai trục độc lập, mở đường cho CSDL serverless tự ngủ đông khi không có tải.
2. **Định lý CAP & Đồng thuận Phân tán:** Khi dữ liệu trải rộng nhiều máy chủ và mạng có thể chia cắt (Partition), bạn phải chọn rõ đánh đổi giữa nhất quán mạnh (CP — Distributed SQL dùng Raft) và sẵn sàng tối đa (AP — NoSQL nhất quán cuối cùng).
3. **Truy vấn Ngữ nghĩa (Vector Search):** Bổ sung khả năng tìm kiếm theo ý nghĩa toán học (embedding) bên cạnh tìm kiếm khóa chính xác của SQL.

---

## Bản đồ kiến thức nền: Ba trụ cột của Chương 07 — Cơ sở dữ liệu phân tán & NoSQL

1. **Định lý CAP & Đánh đổi ACID vs. BASE**:
   Hiểu giới hạn vật lý khi xảy ra chia cắt mạng (*Partition tolerance* là điều tất yếu trong hệ phân tán): khi nào hệ thống bắt buộc phải chọn nhất quán mạnh (ACID/CP như ngân hàng, đăng ký tín chỉ) và khi nào có thể chấp nhận nhất quán cuối cùng (BASE/AP như giỏ hàng, lượt thích).
2. **Bốn họ NoSQL cổ điển & Quy tắc Quorum**:
   Mỗi họ giải quyết một hình thái truy vấn khác nhau: **Key–Value** (Redis/DynamoDB), **Document** (MongoDB), **Wide-Column** (Cassandra) và **Graph** (Neo4j), kết hợp phân mảnh (*sharding*) và nhân bản (*replication*).
3. **Thế hệ mới 2022–2026 (Serverless, Distributed SQL & Vector DB)**:
   Kết nối lý thuyết CAP kinh điển với ba bước tiến hiện đại: cơ sở dữ liệu tự co giãn serverless, SQL phân tán dùng đồng thuận Raft/Paxos, và chỉ mục vector phục vụ tìm kiếm ngữ nghĩa cho AI.

### Khung giải thích (nên đọc chậm): Kiểm tra nhanh quy tắc Quorum $$R + W > N$$ bằng tay

Giả sử một cơ sở dữ liệu phân tán lưu mỗi bản ghi thành **$$N = 3$$ bản sao** trên 3 máy chủ độc lập. Ta chọn số bản sao phải xác nhận khi ghi là $$W$$ và số bản sao phải hỏi khi đọc là $$R$$:

| Cấu hình $$(N, W, R)$$ | Kiểm tra $$R + W > N$$ | Ý nghĩa thực tế |
| :---: | :---: | :--- |
| $$N=3, W=2, R=2$$ | $$2 + 2 = 4 > 3$$ (Đạt) | Luôn có ít nhất 1 máy giao nhau giữa tập ghi và tập đọc → **đọc được dữ liệu mới nhất** (chịu hỏng 1 máy) |
| $$N=3, W=3, R=1$$ | $$3 + 1 = 4 > 3$$ (Đạt) | **Đọc cực nhanh**, nhưng chỉ cần 1 trong 3 máy bảo trì là toàn bộ lệnh ghi bị chặn |
| $$N=3, W=1, R=1$$ | $$1 + 1 = 2 \le 3$$ (Không đạt) | **Đọc và ghi đều cực nhanh, sẵn sàng cao (AP)**, nhưng người đọc có thể nhận dữ liệu cũ vài mili-giây trước khi đồng bộ xong |

Chỉ một bất đẳng thức đơn giản $$R + W > N$$ đã cho bạn thấy trọn vẹn bản chất điều chỉnh độ nhất quán so với độ trễ của các hệ NoSQL phân tán.

> **Lưu ý lộ trình:** Phần lý thuyết bắt buộc của **Chương 07 (NoSQL và Cơ sở dữ liệu phân tán)** hiện được trình bày bằng tiếng Anh. Lộ trình tiếng Việt ở chương này bổ sung **một bài ứng dụng tùy chọn**.
>
> - [07-02 CSDL serverless, SQL phân tán và cơ sở dữ liệu vector]({{ site.baseurl }}{% multilang_post_url contents/chapter07/21-01-01-07_02_Serverless_and_Vector_Databases %})
> - [Bản tiếng Anh của chương]({{ site.baseurl }}/contents/en/chapter07/)

---

### Ví dụ thực tế cầm tay chỉ việc: Bản đồ quyết định nhanh cho 3 kịch bản lưu trữ

| Kịch bản hệ thống | Nhu cầu cốt lõi | Lựa chọn CSDL tối ưu |
| :--- | :--- | :--- |
| **Hệ thống chuyển khoản đa vùng** | Tuyệt đối không mất tiền, giao dịch ACID | Distributed SQL (CockroachDB / Spanner) |
| **Web app khởi nghiệp tải thất thường** | Tiết kiệm chi phí lúc vắng khách, cần SQL | Serverless PostgreSQL (Neon / Aurora Serverless) |
| **Trợ lý AI tìm kiếm 5 triệu đoạn tài liệu** | Tìm tương đồng ngữ nghĩa < 20 ms | `pgvector` (HNSW) hoặc Qdrant / Milvus |

### Khung giải thích: Ba câu hỏi vàng trước khi chọn CSDL đám mây (nên đọc chậm)

1. **Tỷ lệ Đọc / Ghi và mô hình truy cập của bạn là gì?** Thiết kế chỉ mục và kiến trúc replica luôn bắt đầu từ truy vấn nóng nhất, không phải từ sơ đồ ER trên giấy.
2. **Bạn thực sự cần đa vùng ghi (multi-region write) không?** Nếu 95% người dùng ở một khu vực, một cụm đơn vùng có read-replica rẻ hơn và nhanh hơn nhiều so với Distributed SQL toàn cầu.
3. **Dữ liệu có cần kết hợp lọc thuộc tính (metadata) với tìm kiếm vector không?** Nếu có, kiểm tra kỹ hỗ trợ *pre-filtering* của chỉ mục vector để tránh trả về kết quả rỗng!

**Kiểm tra nhanh.** Nếu hệ thống của bạn có 200.000 tài liệu và đã chạy sẵn PostgreSQL, vì sao bật mở rộng `pgvector` thường là bước khởi đầu khôn ngoan hơn việc vận hành thêm một cụm CSDL Vector tách biệt ngay từ ngày đầu?

---

## Video tham khảo

- **Xem trước:** [Cơ Sở Dữ Liệu  Là Gì? Phân Biệt SQL và NoSQL \| Nhập Môn SQL](https://www.youtube.com/watch?v=Olx-rvKtUnA) — Thanh Lê (tiếng Việt). Phân biệt SQL và NoSQL bằng tiếng Việt trước khi đọc phần lý thuyết tiếng Anh của chương.
- [What is a Vector Database? Powering Semantic Search & AI Applications](https://www.youtube.com/watch?v=gl1r1XV0SLw) — IBM Technology (tiếng Anh). Giải thích cơ sở dữ liệu vector và tìm kiếm ngữ nghĩa cho ứng dụng AI.
- [What is a serverless database? (in under 3 minutes)](https://www.youtube.com/watch?v=GxjKsKSOWJk) — CockroachDB (tiếng Anh). Tóm tắt khái niệm cơ sở dữ liệu serverless trong dưới ba phút.
