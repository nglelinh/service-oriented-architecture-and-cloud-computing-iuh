---
layout: post
title: 06-05 Spark thời gian thực, hướng sự kiện và cạnh phục vụ
chapter: '06'
order: 5
owner: Nguyen Le Linh
lang: vi
categories:
- chapter06
lesson_type: optional
---

Stack bắt buộc (Spark SQL, Structured Streaming, MLlib tùy chọn) đủ để *tính*. Bài tùy chọn này cho thấy sản phẩm 2022–2026 nối stack đó vào **SOA hướng sự kiện**: Kafka/CloudEvents vào, bảng lakehouse ra, và đường **online** riêng (feature store + model server) để Spark không nằm trên đường yêu cầu người dùng.

> Lý thuyết bắt buộc đang ở bản tiếng Anh. Đây là phần ứng dụng tùy chọn bằng tiếng Việt.

## Mục tiêu học tập

- Tách *xử lý luồng* (Spark/Flink) khỏi *phục vụ yêu cầu* (gRPC/HTTP, KServe, vLLM).
- Mô tả đường kiểu Kappa vẫn dùng Spark lô cho backfill.
- Phác thảo đầu ra Structured Streaming như bảng Iceberg/Delta cho nhiều engine đọc.

## 1. Hai đồng hồ trong một sản phẩm

| Đồng hồ | Engine điển hình | SLA |
| --- | --- | --- |
| Thời gian sự kiện (gian lận, click, IoT) | Spark Structured Streaming, Flink | giây |
| Thời gian yêu cầu (chấm *user này*) | Feature store + HTTP/gRPC mô hình | mili-giây |
| Thời gian nghiệp vụ (cuối tháng, GDPR) | Spark SQL lô | giờ |

Thất bại thường gặp: phơi query Streaming như API đồng bộ. Micro-batch Spark **không** phải model server p99 50 ms. Mẫu ứng dụng: Streaming **cập nhật trạng thái**; Serving **đọc** cache hoặc feature store.

```python
from pyspark.sql import functions as F

features = (
    kafka_df
    .select(F.col("user_id"), F.col("event_ts").cast("timestamp").alias("ts"), "amount")
    .withWatermark("ts", "10 minutes")
    .groupBy("user_id", F.window("ts", "1 hour"))
    .agg(F.sum("amount").alias("amt_1h"))
)

(
    features.writeStream
    .format("iceberg")
    .outputMode("append")
    .option("checkpointLocation", "s3://lab/chk/feat")
    .toTable("prod.features_user_1h")
)
```

Đường online (không phải Spark): dịch vụ tải đặc trưng gần đây và gọi mô hình.

```python
def score(user_id: str, model_client) -> float:
    feat = feature_store.get(user_id, columns=["amt_1h"])
    return model_client.predict(feat)
```

## 2. Ứng dụng ML hướng sự kiện (2023–2026)

- **Gợi ý nearline**: sự kiện phiên → tổng hợp Streaming → Redis/feature store → dịch vụ xếp hạng.
- **Đánh giá LLM offline**: Spark SQL trên trace (prompt, token, độ trễ) lưu Parquet/Iceberg; LLM được phục vụ bởi vLLM/KServe.
- **MLlib năm 2026**: vẫn hữu ích cho job tuyến tính/cây lớn trên Spark Connect (Spark 4.0 thêm ML-on-Connect). Nhiều nhóm nay huấn luyện bằng Ray/PyTorch và chỉ dùng Spark cho **dữ liệu**.

Khảo sát CNCF 2024 ghi serverless và eventing vẫn trong toolbox nhưng nhóm bỏ khi chi phí/phức tạp thắng. Dùng sự kiện cho **sự thật**; dùng RPC cho **câu hỏi**.

## 3. Engine thống nhất, triển khai tách

“Stack thống nhất” của bài bắt buộc là thắng lợi *phát triển*. Production thường **tách** jar:

1. Job streaming (luôn bật, có checkpoint).
2. Job lô (Airflow/dagster, cùng mã DataFrame).
3. Replica phục vụ (không có Spark driver trong pod).

Sự tách đó giữ ngữ nghĩa sink exactly-once của Structured Streaming khỏi va vào fleet HTTP tự scale.

<div class="content-box insight-box">
<p><strong>Watermark là quy tắc nghiệp vụ.</strong> Watermark 10 phút là tuyên bố về sự kiện di động đến muộn, không phải trivia Spark. Viết nó cạnh SLO.</p>
</div>

## Thách thức

- Dữ liệu muộn vs. marketing “độ tươi” của dashboard.
- Ghi kép (Kafka + bảng) không có outbox giao dịch.
- Lệch train/serve khi đặc trưng Streaming khác backfill lô.

## Bài tập

1. Với ứng dụng thanh toán, liệt kê ba metric thuộc Streaming và ba metric thuộc job Spark SQL đêm.
2. Giải thích vì sao `outputMode("complete")` ra API phục vụ thường là biên SOA sai.
3. Thiết kế backfill: cùng bảng đặc trưng, Spark lô, đúng event-time cho một tuần replay Kafka muộn.

## Tài liệu tham khảo

1. Hướng dẫn Structured Streaming (tài liệu 4.x hiện hành).
2. Apache Spark 4.0.0 — ML trên Spark Connect.
3. CloudEvents: [cloudevents.io](https://cloudevents.io/).
4. CNCF *Cloud Native 2024 Annual Survey*.
5. Tài liệu phục vụ vLLM / KServe — đối ứng thời gian yêu cầu của đường streaming này.
