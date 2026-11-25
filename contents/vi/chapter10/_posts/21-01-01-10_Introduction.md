---
layout: post
title: 10 Giới thiệu — bài ứng dụng tùy chọn
chapter: '10'
order: 1
owner: Nguyen Le Linh
lang: vi
categories:
- chapter10
---

Khi cơ sở dữ liệu PostgreSQL của bạn có 5 triệu dòng, một câu lệnh `GROUP BY` chạy trong chưa đầy một giây. Nhưng khi hệ thống thương mại điện tử hoặc mạng xã hội sinh ra **500 GB log mỗi ngày**, dữ liệu không còn vừa trong RAM hay thậm chí ổ cứng của một máy chủ đơn lẻ nữa—một máy mạnh nhất thế giới cũng sẽ nghẽn cổ chai ở tốc độ đọc đĩa và băng thông mạng.

**Nền tảng dữ liệu lớn (Big Data Platforms)** ra đời từ một tư duy đảo ngược: *thay vì kéo dữ liệu khổng lồ về nơi có CPU, hãy gửi mã tính toán đến thẳng các máy đang chứa dữ liệu*, chia nhỏ công việc lên hàng chục hoặc hàng trăm máy phổ thông chạy song song và chịu được việc một vài máy hỏng giữa chừng.

## Kiến thức cần nhớ: Bốn trụ cột tư duy của hệ thống dữ liệu lớn phân tán

Trước khi đi sâu vào Spark, Kafka hay kiến trúc Lakehouse, bạn cần nắm vững bốn khái niệm nền tảng:

1. **Lưu trữ phân tán & Tính cục bộ dữ liệu (HDFS / Object Storage)**:
   Dữ liệu lớn được cắt thành các khối (*blocks* / *partitions*, thường `128 MB`) và phân tán nhân bản trên nhiều máy chủ. Trong cụm tại chỗ (HDFS), bộ lập lịch cố gắng đặt tác vụ tính toán ngay trên máy chứa khối dữ liệu; trên đám mây (S3/GCS), mạng tốc độ cao cho phép tách rời lưu trữ và tính toán linh hoạt.

2. **Đồ thị thực thi có hướng không chu trình (DAG) & Shuffle (MapReduce → Spark)**:
   Các phép toán cục bộ (`map`, `filter`) chạy song song hoàn toàn không cần máy này nói chuyện với máy kia (*narrow transformations*). Ngược lại, các phép gộp nhóm (`groupByKey`, `join`) buộc phải trao đổi dữ liệu qua mạng giữa mọi máy—gọi là **Shuffle** (*wide transformations*), vốn là điểm nghẽn đắt giá nhất về thời gian và băng thông.

3. **Kiến trúc xử lý theo lô (Batch) vs. Xử lý luồng (Streaming)**:
   **Batch** xử lý tập dữ liệu hữu hạn đã đóng sổ (ví dụ báo cáo doanh thu cuối ngày) với thông lượng cực cao. **Streaming** xử lý sự kiện liên tục theo từng giây (ví dụ phát hiện gian lận thẻ tín dụng tức thì). Kiến trúc **Lambda** duy trì song song cả hai nhánh, còn kiến trúc **Kappa** thống nhất mọi thứ qua một log sự kiện duy nhất (như Kafka) và xử lý lại từ đầu khi cần.

4. **Khả năng chịu lỗi bằng Dòng dõi (Lineage)**:
   Nếu một máy trong cụm 50 máy đột ngột mất điện ở phút thứ 15 của tác vụ, hệ thống không cần chạy lại toàn bộ 100% công việc—nhờ ghi nhớ đồ thị biến đổi (DAG lineage), Spark chỉ cần tính lại đúng các phân vùng bị mất trên một máy khác.

### Giải thích: Trace một tác vụ đếm đơn hàng trên 100 GB dữ liệu — Vì sao Combiner giảm 99% lưu lượng Shuffle?

Giả sử bạn có **100 GB** log đơn hàng (tổng cộng $1\text{ tỷ}$ dòng, mỗi dòng `100 B`) phân bố đều trên **10 máy chủ** (`10 GB/máy`) và cần tính tổng số đơn theo **63 tỉnh/thành phố** trên mạng nội bộ `1 Gbps` ($\approx 125\text{ MB/s}$ mỗi máy).

So sánh hai cách chạy cùng một câu lệnh đếm:

| Giai đoạn | Cách 1: Không gộp cục bộ (`groupByKey` thô) | Cách 2: Có Map-side Combiner (`reduceByKey`) |
| :--- | :--- | :--- |
| **Stage 1 (Map cục bộ trên mỗi máy)** | Mỗi máy phát ra $100\text{ triệu}$ cặp `(tỉnh, 1)` ($\approx 10\text{ GB}$ dữ liệu trung gian/máy) | Mỗi máy cộng dồn ngay trong RAM thành đúng **63 cặp** `(tỉnh, tổng_cục_bộ)` ($\approx \mathbf{2\text{ KB/máy}}$!) |
| **Giai đoạn Shuffle qua mạng** | Đẩy tổng cộng **$100\text{ GB}$** qua mạng giữa 10 máy | Đẩy tổng cộng $10 \times 2\text{ KB} = \mathbf{20\text{ KB}}$ qua mạng |
| **Thời gian truyền mạng thuần túy** | $\frac{10\text{ GB/máy}}{125\text{ MB/s}} \approx \mathbf{80\text{ giây}}$ (chưa kể nén/ghi đĩa tạm) | $\frac{20\text{ KB}}{125\text{ MB/s}} < \mathbf{0{,}001\text{ giây}}$ (tức thì!) |
| **Stage 2 (Reduce cuối)** | Mỗi reducer phải gộp hàng chục triệu bản ghi lẻ | Mỗi reducer chỉ cộng đúng **10 con số** từ 10 máy! |

Bài toán nhỏ này cho thấy quy tắc vàng của mọi kỹ sư Big Data: **luôn lọc sớm (`filter`), chọn đúng cột cần dùng và gộp cục bộ trước khi Shuffle**—sự khác biệt giữa một câu truy vấn chạy 2 giây và một tác vụ treo cả tiếng đồng hồ nằm ở chính lưu lượng Shuffle này.

> **Lưu ý lộ trình:** Phần lý thuyết bắt buộc của **Chương 10 (Nền tảng dữ liệu lớn)** hiện được trình bày bằng tiếng Anh. Lộ trình tiếng Việt ở chương này bổ sung **một bài ứng dụng tùy chọn**.
> 
> - [10-02 Lakehouse mở và ứng dụng đa engine]({{ site.baseurl }}{% multilang_post_url contents/chapter10/21-01-01-10_02_Open_Lakehouse_Applications %})
> - [Bản tiếng Anh của chương]({{ site.baseurl }}/contents/en/chapter10/)

## Video tham khảo

- **Xem trước:** [Big Data Explained](https://www.youtube.com/watch?v=HRfR4dJoKDc) — IBM Technology (tiếng Anh). Giới thiệu tổng quan về dữ liệu lớn và bối cảnh kiến trúc của chương.
- [What is Apache Iceberg?](https://www.youtube.com/watch?v=6tjSVXpHrE8) — IBM Technology (tiếng Anh). Giải thích Apache Iceberg — định dạng bảng mở trọng tâm trong bài tùy chọn về lakehouse.
