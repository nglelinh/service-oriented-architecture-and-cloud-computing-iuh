---
layout: post
title: 04 Giới thiệu — bài ứng dụng tùy chọn
chapter: '04'
order: 1
owner: Nguyen Le Linh
lang: vi
categories:
- chapter04
---

Giả sử bạn cần đếm số lần xuất hiện của từng mã sản phẩm trong **10 terabyte** nhật ký bán hàng. Một chiếc máy tính đơn lẻ đọc ổ đĩa ở tốc độ 500 MB/s sẽ mất gần 6 tiếng chỉ riêng cho việc *đọc xong dữ liệu*, chưa kể hết RAM khi lập bảng đếm. Năm 2004, Google công bố bài báo **MapReduce** với một ý tưởng làm thay đổi cả ngành dữ liệu lớn: thay vì kéo 10 TB dữ liệu về một máy chủ khổng lồ, hãy **đưa đoạn mã tính toán nhỏ bé xuống hàng trăm máy chủ phổ thông** nơi dữ liệu đang nằm sẵn, và để khung phần mềm tự lo việc chịu lỗi khi một máy bất kỳ bị hỏng giữa chừng.

## Bản đồ kiến thức nền: Ba nguyên lý sống mãi của Chương 04

1. **Chia để trị trên cụm phân tán (Map & Reduce)**:
   Tách bài toán lớn thành hai hàm thuần khiết: **Map** biến đổi độc lập từng mảnh dữ liệu thành các cặp `(khóa, giá trị)`, còn **Reduce** tổng hợp tất cả giá trị có cùng một khóa.
2. **Ranh giới Shuffle — Nút cổ chai mạng của dữ liệu lớn**:
   Giữa pha Map và pha Reduce là pha **Shuffle**: sắp xếp và vận chuyển dữ liệu qua mạng để các khóa giống nhau hội tụ về cùng một máy Reducer. Tối ưu hóa xử lý lô về bản chất là nghệ thuật giảm thiểu lượng dữ liệu phải đi qua pha Shuffle.
3. **Chịu lỗi tự động bằng tái thực thi xác định**:
   Khi cụm có 1.000 máy, chuyện một ổ cứng chết giữa tác vụ là điều bình thường. Vì hàm Map/Reduce không có trạng thái ẩn, hệ thống chỉ cần chạy lại đúng mảnh bị lỗi trên một máy khác mà không phải bỏ toàn bộ công việc từ đầu.

### Khung giải thích (nên đọc chậm): Chạy tay bài toán Word Count qua đúng 3 pha Map → Shuffle → Reduce

Giả sử ta có 2 mảnh dữ liệu nhỏ nằm trên 2 máy chủ khác nhau:
- **Máy 1:** `"cloud scale cloud"`
- **Máy 2:** `"scale data cloud"`

| Pha xử lý | Máy 1 thực hiện | Máy 2 thực hiện | Bản chất kỹ thuật |
| :--- | :--- | :--- | :--- |
| **1. Map** | `("cloud", 1), ("scale", 1), ("cloud", 1)` | `("scale", 1), ("data", 1), ("cloud", 1)` | Chạy song song hoàn toàn, không cần giao tiếp mạng |
| **2. Shuffle** | Gom toàn bộ khóa `"cloud"` về Reducer A | Gom khóa `"scale"` và `"data"` về Reducer B | Trao đổi qua mạng theo hàm băm `hash(key) % R` |
| **3. Reduce** | `"cloud": [1, 1, 1]` $$\rightarrow$$ `("cloud", 3)` | `"scale": [1, 1]` $$\rightarrow$$ `2`; `"data": [1]` $$\rightarrow$$ `1` | Tổng hợp độc lập theo từng khóa và ghi ra tệp kết quả |

Dù sau này bạn viết `SELECT word, COUNT(*) FROM docs GROUP BY word` trên Spark SQL hay BigQuery, bộ lập kế hoạch truy vấn vẫn đang dựng lại chính xác 3 bước trên ở tầng dưới!

> **Lưu ý lộ trình:** Phần lý thuyết bắt buộc của **Chương 04 (Hadoop MapReduce)** hiện được trình bày bằng tiếng Anh. Lộ trình tiếng Việt ở chương này bổ sung **một bài ứng dụng tùy chọn**.
>
> - [04-02 Xử lý lô hiện đại sau MapReduce cổ điển]({{ site.baseurl }}{% multilang_post_url contents/chapter04/21-01-01-04_02_Modern_Batch_and_MapReduce %})
> - [Bản tiếng Anh của chương]({{ site.baseurl }}/contents/en/chapter04/)

## Video tham khảo

- **Xem trước:** [MapReduce - Computerphile](https://www.youtube.com/watch?v=cvhKoniK5Uo) — Computerphile (tiếng Anh). Giải thích trực quan mô hình MapReduce cổ điển trước khi chuyển sang các công cụ xử lý lô hiện đại.
- [What is Apache Spark?](https://www.youtube.com/watch?v=p8FGC49N-zM) — Databricks (tiếng Anh). Giới thiệu ngắn gọn về Apache Spark — một trong những engine kế thừa xử lý lô sau MapReduce.
- [What is Apache Iceberg?](https://www.youtube.com/watch?v=6tjSVXpHrE8) — IBM Technology (tiếng Anh). Giới thiệu Apache Iceberg — định dạng bảng mở dùng cho các tác vụ lô trong kiến trúc lakehouse.
