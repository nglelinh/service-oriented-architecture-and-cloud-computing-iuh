---
layout: post
title: 05 Giới thiệu — bài ứng dụng tùy chọn
chapter: '05'
order: 1
owner: Nguyen Le Linh
lang: vi
categories:
- chapter05
---

Nếu MapReduce cổ điển giống như một nhà máy mà sau mỗi công đoạn nhỏ đều phải đóng thùng hàng cất vào kho (ghi xuống ổ đĩa HDFS) rồi công đoạn sau lại mở thùng khiêng ra làm tiếp, thì **Apache Spark** là dây chuyền xử lý liên hoàn ngay trên bộ nhớ RAM. Chỉ riêng việc giữ dữ liệu trung gian trong RAM và lập kế hoạch toàn cục bằng đồ thị DAG đã giúp Spark tăng tốc các thuật toán lặp (như học máy hay phân tích đồ thị) lên gấp hàng chục lần so với Hadoop MapReduce.

Khi dữ liệu doanh nghiệp vượt từ vài trăm megabyte lên hàng trăm terabyte, câu hỏi kiến trúc không còn là *"viết câu SQL nào cho nhanh?"*, mà là **làm sao đưa tính toán tới gần dữ liệu thay vì kéo hàng trăm gigabyte qua mạng!** Chương 05 kết nối bài toán xử lý dữ liệu lớn (Big Data) với kiến trúc hướng dịch vụ và điện toán đám mây hiện đại thông qua **Spark Connect và Spark 4.0**.

Bài mở đầu ngắn này định vị vai trò của Chương 05 trong toàn khóa học: giúp bạn thấy cầu nối liền mạch giữa microservice nhẹ ở tầng ứng dụng và các cụm tính toán phân tán đàn hồi trên đám mây.

---

## Bản đồ kiến thức nền: Ba trụ cột kết nối Big Data với Điện toán Đám mây

1. **Đẩy tính toán về phía dữ liệu (Compute–Storage Co-design):** Đọc định dạng cột (Parquet) kèm cắt tỉa phân vùng (*partition pruning*) và đẩy điều kiện lọc (*predicate pushdown*) giúp giảm lưu lượng mạng hàng trăm lần.
2. **Tách rời lớp điều khiển khỏi lớp thực thi:** Spark Connect biến cụm xử lý dữ liệu lớn thành một **dịch vụ chuẩn gRPC** có thể gọi từ bất kỳ microservice hay sổ tay phân tích nào.
3. **Đàn hồi trên Kubernetes (Cloud-Native Elasticity):** Khởi tạo executor theo nhu cầu thực tế và giải phóng ngay khi job hoàn tất, thay vì duy trì cụm máy chủ cố định chạy không tải suốt 24/7.

---

## Bản đồ kiến thức nền: Ba khái niệm cốt lõi của Chương 05 — Nền tảng Apache Spark

1. **Kiến trúc Driver – Executor – Cluster Manager**:
   *Driver* là bộ não lập kế hoạch truy vấn và chia nhỏ công việc thành các task; các *Executor* là công nhân chạy song song trên các nút máy chủ để tính toán và giữ cache trong RAM; còn *Cluster Manager* (YARN hoặc Kubernetes) cấp phát CPU/RAM cho cụm.
2. **Đồ thị thực thi DAG & Tính toán lười (Lazy Evaluation)**:
   Khi bạn gọi `.filter()` hay `.select()`, Spark chưa hề chạy ngay mà chỉ ghi nhận vào một kế hoạch logic (DAG). Chỉ khi gặp một hành động (*Action*) như `.count()` hay `.write()`, bộ tối ưu Catalyst mới gộp toàn bộ chuỗi biến đổi thành ít bước nhất rồi mới thực thi.
3. **Biến đổi hẹp (Narrow) vs. Biến đổi rộng (Wide Transformation)**:
   Biến đổi hẹp (`map`, `filter`) chạy hoàn toàn cục bộ trên từng phân vùng nên cực nhanh; biến đổi rộng (`groupBy`, `join`) bắt buộc phải trao đổi dữ liệu qua mạng (Shuffle), tạo ranh giới cắt DAG thành các giai đoạn (*Stage*).

### Khung giải thích (nên đọc chậm): So sánh thời gian chạy một thuật toán lặp 10 vòng trên MapReduce vs. Spark

Giả sử bạn chạy một vòng lặp huấn luyện đơn giản **10 bước** trên tập dữ liệu **100 GB** (tốc độ đọc/ghi đĩa cục bộ hiệu dụng của cụm là `2 GB/s`, tốc độ đọc RAM hiệu dụng là `40 GB/s`):

| Chi phí I/O cho 10 vòng lặp | Hadoop MapReduce (ghi/đọc đĩa mỗi vòng) | Apache Spark (giữ trong RAM sau vòng 1) |
| :--- | :---: | :---: |
| Vòng 1 (đọc dữ liệu gốc từ đĩa) | $$100 / 2 = \mathbf{50\text{ giây}}$$ | $$100 / 2 = \mathbf{50\text{ giây}}$$ |
| Ghi/đọc dữ liệu trung gian ở 9 vòng còn lại | $$9 \times (50\text{ ghi} + 50\text{ đọc}) = \mathbf{900\text{ giây}}$$ | $$9 \times (100 / 40) = \mathbf{22{,}5\text{ giây}}$$ |
| **Tổng thời gian I/O ước tính** | **950 giây (~15,8 phút)** | **72,5 giây (~1,2 phút)** — nhanh hơn **$$13\times$$**! |

Khi kết hợp tốc độ trong bộ nhớ này với kiến trúc **Spark Connect** (tách client mỏng qua gRPC) ở bài tùy chọn bên dưới, Spark trở thành một động cơ dữ liệu dễ dàng cắm vào mọi kiến trúc dịch vụ đám mây hiện đại.

> **Lưu ý lộ trình:** Phần lý thuyết bắt buộc của **Chương 05 (Nền tảng Apache Spark)** hiện được trình bày bằng tiếng Anh. Lộ trình tiếng Việt ở chương này bổ sung **một bài ứng dụng tùy chọn**.
>
> - [05-02 Spark Connect, Spark 4.0 và ứng dụng cloud-native]({{ site.baseurl }}{% multilang_post_url contents/chapter05/21-01-01-05_02_Spark_Connect_and_Cloud_Native %})
> - [Bản tiếng Anh của chương]({{ site.baseurl }}/contents/en/chapter05/)

---

### Ví dụ thực tế cầm tay chỉ việc: Sức mạnh của Predicate Pushdown trên 100 GB dữ liệu Parquet

Giả sử kho dữ liệu giao dịch lưu **100 GB** tệp Parquet trên S3 (gồm 20 cột, trải đều 100 ngày, mỗi ngày 1 GB), và một dịch vụ chỉ cần tính tổng doanh thu của **1 ngày duy nhất** trên **2 cột** (`date`, `amount`):
- **Cách ngây thơ (kéo toàn bộ bảng về dịch vụ):** Phải truyền qua mạng $$100	ext{ GB}$$; ở băng thông $$1	ext{ Gbps}$$ ($$125	ext{ MB/s}$$) mất **800 giây (~13,3 phút)**!
- **Cách cloud-native (Partition Pruning + Column Pruning trên Parquet):**
  - Cắt tỉa phân vùng theo ngày giữ lại $$1 / 100 \implies 1	ext{ GB}$$.
  - Chỉ đọc 2/20 cột ($$10\%$$) $$\implies \mathbf{100	ext{ MB}}$$ (giảm **1.000 lần** lượng I/O, hoàn thành trong chưa đầy **1 giây**!).

### Khung giải thích: Ba câu hỏi cần tự hỏi trước khi bước vào bài chuyên sâu Chương 05 (nên đọc chậm)

1. **Bài toán này thực sự cần phân tán chưa?** Dưới vài chục GB, DuckDB/Polars trên một máy chủ đơn thường nhanh và rẻ hơn nhiều so với dựng cả cụm Spark.
2. **Dịch vụ của bạn giao tiếp với tầng dữ liệu lớn qua giao diện nào?** Tránh nhúng trực tiếp thư viện nặng vào API phục vụ người dùng — hãy tách qua giao thức RPC mỏng như Spark Connect hoặc hàng đợi sự kiện.
3. **Định dạng lưu trữ đã tối ưu cho truy vấn phân tích chưa?** Chuyển từ CSV/JSON sang Parquet/Iceberg thường mang lại hiệu quả tốc độ lớn hơn cả việc nhân đôi số máy chủ!

**Kiểm tra nhanh.** Trong ví dụ 100 GB ở trên, nếu dữ liệu được lưu dưới dạng một tệp **CSV** nén duy nhất thay vì Parquet phân vùng theo ngày, hệ thống có thể bỏ qua 99 ngày còn lại và 18 cột thừa không?

---

## Video tham khảo

- **Xem trước:** [What is Apache Spark?](https://www.youtube.com/watch?v=p8FGC49N-zM) — Databricks (tiếng Anh). Giới thiệu tổng quan về Apache Spark trong khoảng hai phút.
- [Spark Connect: Flexible, Local Access to Apache Spark at Scale](https://www.youtube.com/watch?v=Poz63cOyp-Q) — Databricks (tiếng Anh). Trình bày kiến trúc tách client/server của Spark Connect được thảo luận trong bài tùy chọn.
- [Học Spark bằng tiếng việt, miễn phí cho người mới bắt đầu](https://www.youtube.com/watch?v=Gr1chpBWmCU) — Mai DE (tiếng Việt). Video nhập môn Spark bằng tiếng Việt, hữu ích trước khi đọc phần Spark Connect.
