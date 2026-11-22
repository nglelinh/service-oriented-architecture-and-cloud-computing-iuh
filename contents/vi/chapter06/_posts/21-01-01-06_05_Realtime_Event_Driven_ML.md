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

Bộ công cụ bắt buộc (Spark SQL, Structured Streaming và phần tùy chọn MLlib) giải quyết bài toán *tính toán dữ liệu*. Bài tùy chọn này minh họa cách các hệ thống giai đoạn 2022–2026 kết nối bộ công cụ đó vào **kiến trúc hướng dịch vụ (SOA) hướng sự kiện**: tiếp nhận sự kiện từ Kafka/CloudEvents, ghi kết quả ra bảng lakehouse, và tách riêng nhánh **phục vụ trực tuyến (online serving)** gồm feature store và máy chủ mô hình (model server) để Spark không nằm trực tiếp trên đường xử lý yêu cầu người dùng.

> Lý thuyết bắt buộc hiện ở bản tiếng Anh. Đây là bài ứng dụng tùy chọn bằng tiếng Việt.

## Mục tiêu học tập

- Tách bạch tầng *xử lý luồng* (Spark/Flink) khỏi tầng *phục vụ yêu cầu trực tuyến* (gRPC/HTTP, KServe, vLLM).
- Mô tả luồng dữ liệu kiểu Kappa nhưng vẫn kết hợp Spark xử lý lô cho bài toán tính lại dữ liệu lịch sử (backfill).
- Trình bày cách xuất kết quả Structured Streaming thành bảng Iceberg/Delta để nhiều engine khác nhau cùng khai thác.

## 1. Ba nhịp thời gian trong một hệ thống sản phẩm

| Nhịp xử lý | Engine tiêu biểu | Mức cam kết độ trễ (SLA) |
| --- | --- | --- |
| Thời gian sự kiện (gian lận, lượt nhấn, IoT) | Spark Structured Streaming, Flink | tính bằng giây |
| Thời gian yêu cầu (chấm điểm *người dùng hiện tại*) | Feature store + API HTTP/gRPC của mô hình | tính bằng mili-giây |
| Thời gian nghiệp vụ (khóa sổ cuối tháng, GDPR) | Spark SQL xử lý lô | tính bằng giờ |

Một sai lầm kiến trúc phổ biến là đưa trực tiếp truy vấn Streaming làm API đồng bộ cho người dùng cuối. Cơ chế micro-batch của Spark **không** được thiết kế cho máy chủ suy luận yêu cầu p99 ở mức 50 ms. Mẫu thiết kế phù hợp là: tầng Streaming **cập nhật trạng thái**, còn tầng Serving **đọc** từ bộ nhớ đệm hoặc feature store.

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

Nhánh phục vụ trực tuyến (tách khỏi Spark): dịch vụ ứng dụng đọc đặc trưng gần nhất và gọi mô hình dự đoán.

```python
def score(user_id: str, model_client) -> float:
    feat = feature_store.get(user_id, columns=["amt_1h"])
    return model_client.predict(feat)
```

## 2. Ứng dụng học máy hướng sự kiện (2023–2026)

- **Hệ thống gợi ý cận thời gian thực (nearline):** Sự kiện phiên truy cập → tổng hợp bằng Streaming → ghi vào Redis/feature store → dịch vụ xếp hạng đọc khi phục vụ người dùng.
- **Đánh giá LLM ngoại tuyến (offline):** Dùng Spark SQL phân tích nhật ký trace (prompt, số token, độ trễ) lưu ở định dạng Parquet/Iceberg, trong khi việc suy luận LLM trực tuyến do vLLM/KServe đảm nhận.
- **Vai trò của MLlib hiện nay:** Vẫn phù hợp với các mô hình tuyến tính hoặc cây quyết định quy mô lớn trên Spark Connect (Spark 4.0 đã bổ sung ML trên Connect). Tuy vậy, nhiều nhóm hiện chọn huấn luyện bằng Ray hoặc PyTorch và chủ yếu dùng Spark cho khâu **xử lý dữ liệu**.

Khảo sát CNCF 2024 cho thấy kiến trúc serverless và hướng sự kiện vẫn phổ biến, nhưng một số tổ chức chủ động thu hẹp phạm vi áp dụng khi chi phí hoặc độ phức tạp vận hành vượt quá lợi ích. Nguyên tắc phân định rõ ràng là: dùng sự kiện để truyền tải **dữ liệu đã phát sinh**, và dùng RPC cho các **truy vấn cần phản hồi ngay**.

## 3. Thống nhất ở tầng phát triển, tách biệt khi triển khai

Ưu điểm của "ngăn xếp thống nhất" (unified stack) là lập trình viên có thể dùng chung API DataFrame trong giai đoạn phát triển. Tuy nhiên, khi vận hành thực tế, các hệ thống thường **tách biệt đơn vị triển khai**:

1. Tác vụ streaming chạy liên tục, quản lý checkpoint chặt chẽ.
2. Tác vụ lô chạy theo lịch (Airflow hoặc Dagster), tái sử dụng cùng logic DataFrame.
3. Các bản sao dịch vụ phục vụ trực tuyến (không chứa Spark driver bên trong pod).

Sự tách biệt này giúp cơ chế đảm bảo ghi chính xác (exactly-once sink) của Structured Streaming không bị xung đột với quá trình tự động mở rộng (auto-scaling) của cụm dịch vụ HTTP.

<div class="content-box insight-box">
<p><strong>Watermark là một quy tắc nghiệp vụ.</strong> Ngưỡng watermark 10 phút phản ánh trực tiếp cam kết về mức độ trễ chấp nhận được của sự kiện từ thiết bị di động, không chỉ là một tham số kỹ thuật của Spark. Hãy tài liệu hóa ngưỡng này ngay cạnh SLO của dịch vụ.</p>
</div>

## Thách thức

- Mâu thuẫn giữa bản chất dữ liệu đến muộn (late data) và kỳ vọng "dữ liệu tức thời" của bảng điều khiển báo cáo.
- Rủi ro ghi kép (ghi đồng thời vào Kafka và cơ sở dữ liệu) khi không áp dụng mẫu transactional outbox.
- Sai lệch giữa huấn luyện và phục vụ (train/serve skew) nếu công thức tính đặc trưng ở nhánh Streaming khác với nhánh backfill lô.

## Video tham khảo

- **Xem trước:** [Event-Driven Architecture(EDA) vs. Request/Response (RR): System Design Explained](https://www.youtube.com/watch?v=7fkS-18KBlw) — Confluent, an IBM Company (tiếng Anh). Đối chiếu kiến trúc hướng sự kiện và yêu cầu–phản hồi, làm rõ lý do không đặt Spark trên đường phục vụ trực tiếp.
- [What is a Feature Store for Machine Learning?](https://www.youtube.com/watch?v=DESBDukN9gw) — Hopsworks (tiếng Anh). Giải thích vai trò của feature store trong nhánh phục vụ trực tuyến.
- [Introduction to Vertex AI Feature Store](https://www.youtube.com/watch?v=jXD8Sfx4hvQ) — Google Cloud Tech (tiếng Anh). Ví dụ dịch vụ feature store được quản lý trên đám mây.

## Bài tập

1. Đối với một hệ thống thanh toán, liệt kê ba chỉ số nên tính bằng Structured Streaming và ba chỉ số phù hợp với tác vụ Spark SQL chạy định kỳ ban đêm.
2. Giải thích vì sao việc xuất `outputMode("complete")` trực tiếp sang API phục vụ người dùng thường là một ranh giới thiết kế SOA không hợp lý.
3. Thiết kế quy trình backfill dùng Spark lô trên cùng bảng đặc trưng để xử lý lại đúng theo dấu thời gian sự kiện (event-time) cho dữ liệu phát lại từ Kafka chậm một tuần.

## Tài liệu tham khảo

1. Hướng dẫn Structured Streaming (tài liệu Spark 4.x hiện hành).
2. Apache Spark 4.0.0 — hỗ trợ ML trên Spark Connect.
3. CloudEvents: [cloudevents.io](https://cloudevents.io/).
4. CNCF *Cloud Native 2024 Annual Survey*.
5. Tài liệu phục vụ mô hình vLLM / KServe — mảnh ghép tầng phục vụ trực tuyến tương ứng với nhánh streaming này.
