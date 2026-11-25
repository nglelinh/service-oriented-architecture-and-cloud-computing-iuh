---
layout: post
title: 11 Giới thiệu — bài ứng dụng tùy chọn
chapter: '11'
order: 1
owner: Nguyen Le Linh
lang: vi
categories:
- chapter11
---

Trong khoa học dữ liệu và kiến trúc đám mây có một quy luật nghiệt ngã: *"Garbage in, garbage out"* — rác đi vào thì rác đi ra. Một thuật toán học máy tinh vi hay bảng điều khiển BI hào nhoáng đến đâu cũng trở nên vô dụng nếu dữ liệu đầu vào bị thiếu ngày, sai múi giờ, trùng lặp đơn hàng sau mỗi lần mạng chập chờn, hoặc lệch định dạng tiền tệ giữa hai chi nhánh.

Chương 11 tập trung vào chặng đường đầu tiên nhưng chiếm tới **60–80% công sức** của mọi kỹ sư dữ liệu: **thu thập (ingestion), làm sạch (cleaning) và chuẩn hóa dữ liệu** sao cho đường ống vừa chịu lỗi tốt, vừa chạy lại được an toàn khi sự cố xảy ra.

## Kiến thức cần nhớ: Bốn nguyên lý của đường ống thu thập và làm sạch dữ liệu

1. **ETL vs. ELT (Biến đổi trước hay sau khi nạp?)**:
   Mô hình **ETL (Extract–Transform–Load)** truyền thống biến đổi và làm sạch dữ liệu trên một máy chủ trung gian trước khi nạp vào kho dữ liệu đắt tiền. Mô hình **ELT (Extract–Load–Transform)** hiện đại trên đám mây đổ thẳng dữ liệu thô vào Object Storage/Lakehouse (tầng *Bronze*), rồi dùng sức mạnh SQL phân tán (dbt, Spark) biến đổi dần sang tầng sạch (*Silver*) và tầng tổng hợp kinh doanh (*Gold* — còn gọi là **Kiến trúc Medallion**).

2. **Schema-on-Write vs. Schema-on-Read**:
   **Schema-on-Write** kiểm tra chặt kiểu dữ liệu ngay lúc ghi (tránh dữ liệu bẩn nhưng kém linh hoạt khi nguồn thay đổi). **Schema-on-Read** chấp nhận lưu dữ liệu bán cấu trúc (JSON/log) ngay lập tức và áp lược đồ lúc đọc—linh hoạt khi thu thập nhưng đòi hỏi lớp kiểm định chặt ở tầng Silver.

3. **Tính lũy đẳng (Idempotency) — Phép thử quan trọng của đường ống dữ liệu**:
   Trong hệ phân tán, mạng luôn có thể đứt đúng giây thứ 59 khiến tác vụ chạy lại (retry). Một đường ống được gọi là **lũy đẳng** nếu chạy nó **1 lần hay 5 lần** trên cùng một tệp đầu vào đều cho ra **đúng một kết quả cuối cùng**, không bao giờ nhân đôi doanh thu hay đếm đúp giao dịch.

4. **Phân biệt dữ liệu thiếu, nhiễu và ngoại lai (Missing, Noise, Outliers)**:
   Xóa mù quáng mọi dòng có `NULL` có thể làm lệch mẫu thống kê nghiêm trọng (ví dụ khách hàng không điền tuổi lại là nhóm chi tiêu cao nhất). Làm sạch đúng nghĩa luôn gắn liền với ngữ nghĩa nghiệp vụ: điền giá trị mặc định có ghi chú, chuẩn hóa múi giờ về UTC, và tách riêng bản ghi bất thường để kiểm chứng.

### Giải thích: Tính tay hậu quả khi pipeline không lũy đẳng chạy lại 2 lần trên 500 đơn hàng

Giả sử cuối ngày `2026-10-20`, tệp `orders_20261020.csv` có đúng **500 đơn hàng**, tổng doanh thu thực tế là **`250.000.000 VNĐ`**. Đường ống nạp dữ liệu đọc từng dòng và ghi vào bảng `fact_orders`. Lúc đang ghi đến đơn thứ `320`, kết nối mạng tới kho dữ liệu bị ngắt, trình lập lịch (Airflow) tự động thử chạy lại (retry) từ đầu tệp!

Hãy so sánh kết quả cuối cùng trong bảng `fact_orders` giữa hai cách viết lệnh nạp:

| Thiết kế đường ống | Lần chạy 1 (ngắt ở dòng 320) | Lần chạy 2 (retry thành công đủ 500 dòng) | Tổng dòng & Doanh thu trong bảng | Độ chính xác |
| :--- | :---: | :---: | :---: | :---: |
| **Cách ngây thơ (`INSERT APPEND`)** | Đã ghi 320 đơn (`160 triệu`) | Ghi nối thêm 500 đơn (`250 triệu`) | **`820` đơn — `410.000.000 VNĐ`** | **Sai lệch $+64\%$!** (320 đơn bị tính đúp) |
| **Cách lũy đẳng (`DELETE` phân vùng ngày + `INSERT`, hoặc `MERGE` theo `order_id`)** | Đã ghi 320 đơn (hoặc rollback chưa commit) | Ghi đè/cập nhật khớp đúng khóa `order_id` từ `1..500` | **`500` đơn — `250.000.000 VNĐ`** | **Chính xác $100\%$** dù retry 1 hay 10 lần |

Chỉ cần nhìn phép cộng `160 triệu + 250 triệu = 410 triệu`, bạn sẽ hiểu ngay vì sao mọi kỹ sư dữ liệu chuyên nghiệp đều kiểm tra câu hỏi đầu tiên trước khi duyệt code: *"Nếu job này chết giữa chừng và chạy lại lúc 3 giờ sáng, bảng đích có bị nhân đôi dữ liệu không?"*

> **Lưu ý lộ trình:** Phần lý thuyết bắt buộc của **Chương 11 (Thu thập và làm sạch dữ liệu)** hiện được trình bày bằng tiếng Anh. Lộ trình tiếng Việt ở chương này bổ sung **một bài ứng dụng tùy chọn**.
> 
> - [11-02 Hợp đồng dữ liệu, cổng chất lượng và pipeline AI]({{ site.baseurl }}{% multilang_post_url contents/chapter11/21-01-01-11_02_Data_Contracts_and_AI_Pipelines %})
> - [Bản tiếng Anh của chương]({{ site.baseurl }}/contents/en/chapter11/)

## Video tham khảo

- **Xem trước:** [What is Data Cleaning? \| Data Fundamentals for Beginners](https://www.youtube.com/watch?v=WpX2F2BS3Qc) — Alex The Analyst (tiếng Anh). Giới thiệu khái niệm làm sạch dữ liệu — nền tảng quyết định chất lượng phân tích.
- [What is a Data Contract?](https://www.youtube.com/watch?v=-n3OD-ml_k0) — IBM Technology (tiếng Anh). Định nghĩa ngắn gọn về hợp đồng dữ liệu trong khoảng ba phút.
- [Data Cleaning Explained in 1 Video. Missing Values, Outliers & Imputation \| #datascience](https://www.youtube.com/watch?v=MgjxaTqXdRE) — Neural Notes (tiếng Anh). Tổng hợp xử lý giá trị thiếu, giá trị ngoại lai và kỹ thuật điền khuyết (imputation).
