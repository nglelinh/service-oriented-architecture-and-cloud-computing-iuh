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

Mô hình lập trình MapReduce (map, shuffle, reduce, chạy lại khi lỗi) vẫn là mô hình tư duy cho **xử lý lô**. Điều đổi trong 2022–2026 là *shuffle sống ở đâu* và *engine nào* chạy: Spark, Flink batch, BigQuery/Dataflow, và bảng **lakehouse mở** khiến job lô chỉ là thêm một reader của cùng tệp.

> Lý thuyết bắt buộc đang ở bản tiếng Anh. Đây là phần ứng dụng tùy chọn bằng tiếng Việt.

## Mục tiêu học tập

- Nhận ra hậu duệ MapReduce trong dịch vụ lô đám mây và job nén lakehouse.
- Đối chiếu “viết lớp Mapper” với lô SQL/DataFrame vẫn shuffle.
- Giải thích vì sao lô không chết khi streaming phổ biến (Lambda/Kappa, rewrite Iceberg).

## 1. Mô hình không biến mất — nó có trình biên dịch

SQL Spark hoặc BigQuery 2025 như `SELECT country, COUNT(*) FROM events GROUP BY country` vẫn là MapReduce: tổng hợp cục bộ, shuffle theo `country`, tổng hợp cuối. Word Count của bài bắt buộc là cùng DAG với cú pháp dễ hơn.

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

Điều *thật sự* đổi: **chịu lỗi** thường là “tính lại partition mất từ lineage hoặc snapshot bảng” hơn là “chạy lại Mapper Java trên một split HDFS.” Object storage cộng **Apache Iceberg / Delta Lake** cho commit nguyên tử để job lô lỗi không để lại thư mục Hive viết dở.

## 2. Ứng dụng lô native-đám mây

| Mẫu | Hadoop những năm 2010 | Ví dụ 2022–2026 |
| --- | --- | --- |
| ETL đêm | MR + Oozie | Job Spark/Flink trên Kubernetes hoặc EMR/Dataproc |
| SQL lớn | Hive trên MR | BigQuery, Snowflake, Spark SQL trên Iceberg |
| Pipeline di động | Khóa nhà cung cấp | Runner Apache Beam (Dataflow, Flink, Spark) |
| Nén/sắp tệp | `fsimage` + MR | Iceberg rewrite / Delta OPTIMIZE |

**Apache Beam** giữ ý tưởng *di động* của MapReduce: một pipeline, nhiều runner. Google Cloud Dataflow là runner quản lý. Netflix (nguồn gốc Iceberg), Apple và nhiều ngân hàng vẫn chạy **lô nhiều giờ** vì khóa sổ, backfill đặc trưng và xóa GDPR vốn bị chặn (bounded). Streaming không xóa các job đó; nó thêm đồng hồ thứ hai.

## 3. Lô lakehouse: MapReduce trên bảng mở

Định dạng bảng mở biến object storage thành thứ reducer có thể tin:

- **Cô lập snapshot** — reader thấy phiên bản đã commit.
- **Phân vùng ẩn** — không tự viết `dt=2026-09-16` trong mọi Mapper.
- **Nén** — tệp nhỏ từ ingest streaming được *viết lại theo lô* (merge phía reduce cổ điển).

Khảo sát cộng đồng Iceberg 2025 (kết quả 2026) ghi Iceberg là định dạng mở được nêu nhiều nhất trong *mẫu đó*, với Spark và Trino là engine phổ biến. Coi đó là *bằng chứng cộng đồng*, không phải thị phần toàn cầu.

<div class="content-box insight-box">
<p><strong>Shuffle là thuế.</strong> Dù trả bằng container YARN hay executor Kubernetes, <code>GROUP BY</code> rộng và join vẫn thống trị chi phí. Kỹ năng ứng dụng là giảm khóa shuffle và số tệp — không phải bỏ từ vựng MapReduce.</p>
</div>

## 4. Mini case: xóa GDPR như reduce lô

1. **Map**: đọc tombstone user-id (nhỏ) và partition sự kiện (lớn).
2. **Shuffle**: co-group theo `user_id` (hoặc broadcast tập tombstone nếu vừa bộ nhớ).
3. **Reduce**: ghi snapshot Iceberg mới không còn các hàng đó.
4. **Chịu lỗi**: writer chết thì snapshot trước vẫn đọc được.

Đó là câu chuyện chạy lại của bài bắt buộc, áp vào job tuân thủ những năm 2020.

## Thách thức

- Bùng nổ tệp nhỏ khi ingest streaming không bao giờ chạy nén lô.
- Nhóm “chỉ SQL” không giải thích được lệch dữ liệu cho đến khi hóa đơn tới.
- Sink “exactly-once” thực ra là “at-least-once + merge idempotent.”

## Bài tập

1. Viết lại Word Count bằng Spark SQL và đánh dấu biên shuffle.
2. Ước lượng join đêm 2 TB nên dùng broadcast map-side hay shuffled reduce-side (nêu giả định bộ nhớ).
3. Thiết kế chính sách retry cho pipeline Beam ghi Iceberg: cái gì chạy lại được an toàn?

## Tài liệu tham khảo

1. Apache Spark 4.0.0 (23/5/2025).
2. Tài liệu Apache Iceberg (snapshot, rewrite): [iceberg.apache.org](https://iceberg.apache.org/).
3. Data Lakehouse Hub, khảo sát hệ sinh thái Iceberg 2025 (2/2026).
4. Hướng dẫn lập trình Apache Beam: [beam.apache.org](https://beam.apache.org/).
5. Dean & Ghemawat, “MapReduce” (OSDI 2004).
