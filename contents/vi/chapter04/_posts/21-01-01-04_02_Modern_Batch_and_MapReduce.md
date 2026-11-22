---
layout: post
title: 04-02 Xử lý lô hiện đại sau MapReduce cổ điển (2022–2026)
chapter: '04'
order: 3
owner: Nguyen Le Linh
lang: vi
categories:
- chapter04
lesson_type: optional
---

Mô hình tư duy của MapReduce (map, shuffle, reduce và chạy lại khi gặp lỗi) vẫn là nền tảng của **xử lý lô (batch processing)**. Điểm thay đổi trong giai đoạn 2022–2026 nằm ở *vị trí thực hiện bước shuffle* và *engine thực thi*: Spark, Flink ở chế độ batch, BigQuery/Dataflow cùng các định dạng bảng **lakehouse mở** giúp mỗi tác vụ lô chỉ cần đọc trực tiếp từ cùng một kho tệp chung.

> Lý thuyết bắt buộc hiện ở bản tiếng Anh. Đây là bài ứng dụng tùy chọn bằng tiếng Việt.

## Mục tiêu học tập

- Nhận diện các nguyên lý kế thừa từ MapReduce trong dịch vụ xử lý lô đám mây và tác vụ nén dữ liệu (compaction) của lakehouse.
- So sánh cách viết lớp `Mapper` thủ công với truy vấn SQL/DataFrame hiện đại (vốn vẫn thực hiện shuffle bên dưới).
- Giải thích vì sao xử lý lô vẫn giữ vai trò thiết yếu ngay cả khi xử lý luồng (streaming) trở nên phổ biến (kiến trúc Lambda/Kappa, tác vụ viết lại tệp Iceberg).

## 1. Mô hình không biến mất — nó được nâng cấp lớp biên dịch

Một truy vấn Spark SQL hoặc BigQuery như `SELECT country, COUNT(*) FROM events GROUP BY country` về bản chất vẫn vận hành theo mẫu MapReduce: tổng hợp cục bộ trên từng phân vùng, thực hiện shuffle theo khóa `country`, rồi tổng hợp kết quả cuối cùng. Bài toán Word Count trong phần lý thuyết bắt buộc có cùng cấu trúc đồ thị thực thi (DAG), chỉ khác ở cú pháp biểu đạt ngắn gọn hơn.

```python
from collections import Counter

def map_line(line: str):
    return [(word.lower(), 1) for word in line.split()]

def reduce_counts(pairs):
    c = Counter()
    for k, v in pairs:
        c[k] += v
    return c
```

Thay đổi *thực chất* nằm ở cơ chế **chịu lỗi**: thay vì khởi chạy lại lớp Mapper Java trên một khối tách (split) của HDFS, hệ thống hiện đại tính lại phân vùng bị mất dựa trên đồ thị nguồn gốc (lineage) hoặc trạng thái snapshot của bảng. Lưu trữ đối tượng (object storage) kết hợp với **Apache Iceberg** hoặc **Delta Lake** cung cấp cơ chế ghi nguyên tử (atomic commit), đảm bảo một tác vụ lô gặp sự cố giữa chừng không để lại thư mục dữ liệu ghi dở như mô hình Hive trước đây.

## 2. Các mẫu ứng dụng xử lý lô thuần đám mây

| Mẫu bài toán | Hệ sinh thái Hadoop thập niên 2010 | Thực tế giai đoạn 2022–2026 |
| --- | --- | --- |
| ETL chạy định kỳ ban đêm | MapReduce + Oozie | Tác vụ Spark/Flink trên Kubernetes hoặc EMR/Dataproc |
| Truy vấn SQL quy mô lớn | Hive trên MapReduce | BigQuery, Snowflake, Spark SQL trên Iceberg |
| Pipeline di động giữa các engine | Phụ thuộc từng nhà cung cấp | Apache Beam với nhiều runner (Dataflow, Flink, Spark) |
| Nén và sắp xếp tệp | `fsimage` + MapReduce | Iceberg rewrite / Delta `OPTIMIZE` |

**Apache Beam** duy trì triết lý *di động* của MapReduce: viết một pipeline duy nhất và chạy trên nhiều runner khác nhau, trong đó Google Cloud Dataflow là dịch vụ quản lý phổ biến. Nhiều tổ chức như Netflix (nơi khởi nguồn Iceberg), Apple hay các ngân hàng vẫn duy trì các **tác vụ lô kéo dài nhiều giờ** cho bài toán khóa sổ kế toán, tính lại đặc trưng lịch sử (backfill) và xóa dữ liệu theo yêu cầu GDPR — những bài toán vốn thao tác trên tập dữ liệu hữu hạn (bounded). Xử lý luồng không thay thế hoàn toàn xử lý lô mà bổ sung thêm một nhánh thời gian thực song song.

## 3. Xử lý lô trong lakehouse: MapReduce trên định dạng bảng mở

Các định dạng bảng mở biến kho lưu trữ đối tượng thành tầng dữ liệu tin cậy cho các bước tổng hợp:

- **Cô lập theo snapshot:** Tiến trình đọc luôn nhìn thấy một phiên bản dữ liệu nhất quán đã được commit.
- **Phân vùng ẩn (hidden partitioning):** Không còn phải ghi cứng cấu trúc thư mục kiểu `dt=2026-09-16` trong mã xử lý.
- **Nén tệp (compaction):** Hàng loạt tệp nhỏ sinh ra từ luồng nạp dữ liệu thời gian thực được *gom và viết lại theo lô* — một dạng gộp ở pha reduce kinh điển.

Khảo sát cộng đồng Iceberg năm 2025 (công bố đầu năm 2026) ghi nhận Iceberg là định dạng bảng mở được nhắc đến nhiều nhất trong *mẫu khảo sát đó*, với Spark và Trino là hai engine truy vấn phổ biến. Cần xem đây là *dữ liệu phản ánh xu hướng cộng đồng* chứ không phải số liệu thị phần toàn cầu.

<div class="content-box insight-box">
<p><strong>Shuffle vẫn là chi phí lớn nhất.</strong> Dù chạy trên container YARN hay executor của Kubernetes, các phép <code>GROUP BY</code> rộng và phép nối (join) lớn vẫn chiếm phần lớn chi phí tính toán và mạng. Kỹ năng thực tế quan trọng là giảm lượng dữ liệu phải shuffle và kiểm soát số lượng tệp nhỏ, chứ không phải bỏ qua mô hình tư duy MapReduce.</p>
</div>

## 4. Tình huống ngắn: xóa dữ liệu GDPR như một tác vụ reduce lô

1. **Map:** Đọc danh sách mã người dùng cần xóa (tombstone, kích thước nhỏ) và các phân vùng sự kiện (kích thước lớn).
2. **Shuffle:** Gom nhóm theo `user_id` (hoặc broadcast danh sách cần xóa nếu đủ nhỏ để nằm gọn trong bộ nhớ).
3. **Reduce:** Ghi ra một snapshot Iceberg mới đã loại bỏ các bản ghi tương ứng.
4. **Chịu lỗi:** Nếu tiến trình ghi gặp sự cố, các tiến trình đọc vẫn tiếp tục đọc an toàn từ snapshot trước đó.

Đây chính là cơ chế chạy lại an toàn của MapReduce trong phần lý thuyết, được áp dụng vào bài toán tuân thủ pháp lý hiện đại.

## Thách thức

- Gia tăng đột biến số lượng tệp nhỏ nếu luồng nạp dữ liệu thời gian thực không chạy kèm tác vụ nén lô định kỳ.
- Các nhóm chỉ viết SQL thuần thường khó phát hiện tình trạng lệch phân bố dữ liệu (data skew) cho đến khi chi phí tính toán tăng vọt.
- Cam kết ghi "đúng một lần" (exactly-once) trên thực tế thường được xây dựng từ cơ chế "ít nhất một lần" (at-least-once) kết hợp phép gộp có tính lũy đẳng (idempotent merge).

## Video tham khảo

- **Xem trước:** [What is Apache Spark?](https://www.youtube.com/watch?v=p8FGC49N-zM) — Databricks (tiếng Anh). Giới thiệu Spark — engine vẫn thực thi luồng map/shuffle/reduce dưới lớp tối ưu hóa hiện đại.
- [What is Apache Iceberg?](https://www.youtube.com/watch?v=6tjSVXpHrE8) — IBM Technology (tiếng Anh). Giải thích Apache Iceberg — định dạng bảng mở phục vụ xử lý lô trên object storage.
- [Big Data Architecture Patterns \| Lambda vs Kappa Architecture](https://www.youtube.com/watch?v=waDJcSCXz_Y) — Software Architecture Academy (tiếng Anh). So sánh kiến trúc Lambda và Kappa, làm rõ vai trò song hành của nhánh lô và nhánh luồng.

## Bài tập

1. Viết lại bài toán Word Count bằng Spark SQL và chỉ rõ bước nào phát sinh ranh giới shuffle.
2. Đánh giá xem một phép nối (join) dữ liệu 2 TB chạy ban đêm nên dùng broadcast map-side join hay shuffled reduce-side join (nêu rõ giả định về dung lượng bộ nhớ).
3. Thiết kế chính sách thử lại (retry) cho một pipeline Apache Beam ghi vào bảng Iceberg và xác định những bước nào có thể chạy lại an toàn.

## Tài liệu tham khảo

1. Apache Spark 4.0.0 (23/5/2025).
2. Tài liệu Apache Iceberg (snapshot, rewrite): [iceberg.apache.org](https://iceberg.apache.org/).
3. Data Lakehouse Hub, khảo sát hệ sinh thái Iceberg 2025 (2/2026).
4. Hướng dẫn lập trình Apache Beam: [beam.apache.org](https://beam.apache.org/).
5. Dean & Ghemawat, “MapReduce” (OSDI 2004).
