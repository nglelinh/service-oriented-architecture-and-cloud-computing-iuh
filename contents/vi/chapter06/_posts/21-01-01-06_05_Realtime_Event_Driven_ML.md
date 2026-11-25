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

Khi người dùng quẹt thẻ thanh toán ở một thành phố lạ, hệ thống chống gian lận chỉ có chưa đầy **50 mili-giây** để chấm điểm rủi ro: tổng số tiền người này đã chi trong 1 giờ qua là bao nhiêu? Số lần quẹt thẻ thất bại trong 5 phút gần nhất là mấy lần? Nếu dịch vụ API trực tiếp chạy một câu truy vấn Spark quét bảng giao dịch ngay lúc khách đang đứng trước máy POS, giao dịch sẽ timeout ngay lập tức. Bí quyết của các hệ thống học máy thời gian thực hiện đại là **tách đôi hai nhịp thời gian**: nhánh xử lý luồng (Structured Streaming) âm thầm tính sẵn đặc trưng liên tục theo dòng sự kiện, còn nhánh phục vụ trực tuyến (Online Serving) chỉ việc đọc kết quả sẵn có trong vài mili-giây.

Bộ công cụ bắt buộc (Spark SQL, Structured Streaming và phần tùy chọn MLlib) giải quyết bài toán *tính toán dữ liệu*. Bài tùy chọn này minh họa cách các hệ thống giai đoạn 2022–2026 kết nối bộ công cụ đó vào **kiến trúc hướng dịch vụ (SOA) hướng sự kiện**: tiếp nhận sự kiện từ Kafka/CloudEvents, ghi kết quả ra bảng lakehouse, và tách riêng nhánh **phục vụ trực tuyến (online serving)** gồm feature store và máy chủ mô hình (model server) để Spark không nằm trực tiếp trên đường xử lý yêu cầu người dùng.

> Lý thuyết bắt buộc hiện ở bản tiếng Anh. Đây là bài ứng dụng tùy chọn bằng tiếng Việt.

Một kẻ gian vừa quẹt thẻ tín dụng bị đánh cắp ở hai thành phố cách nhau 1.000 km chỉ trong vòng 3 phút. Nếu hệ thống chống gian lận của bạn chạy bằng job batch tổng hợp lúc nửa đêm, bạn sẽ phát hiện ra kẻ gian rất chính xác... sau khi tiền đã biến mất từ 12 tiếng trước! Để chặn đứng rủi ro ngay khoảnh khắc giao dịch diễn ra, kiến trúc hiện đại kết hợp **xử lý luồng thời gian thực (Structured Streaming)**, **hàng đợi sự kiện (Kafka)** và **kho đặc trưng trực tuyến (Online Feature Store)**.

Bài học này bóc tách các cơ chế cốt lõi của hệ thống ML hướng sự kiện: phân biệt **event time vs. processing time**, cách **watermark** xử lý sự kiện đến muộn mà không làm tràn RAM, và cách bảo vệ tính nhất quán đặc trưng giữa lúc huấn luyện (offline) và lúc phục vụ (online).

---

## Kiến thức cần nhớ: Bốn trụ cột của Kiến trúc ML Hướng Sự Kiện (Event-Driven ML)

1. **Event Time vs. Processing Time:** Sự kiện sinh ra trên điện thoại người dùng lúc `10:00:05` (*event time*) nhưng vì mất sóng 4G có thể đến máy chủ lúc `10:00:18` (*processing time*). Cửa sổ thời gian đúng nghiệp vụ luôn phải nhóm theo *event time*.
2. **Watermark & Quản lý trạng thái hữu hạn:** Nếu cứ giữ mở mãi mọi cửa sổ cũ chờ sự kiện đến trễ, bộ nhớ RAM của hệ thống streaming sẽ phình to đến khi sập! **Watermark** đặt một ngưỡng trễ tối đa (ví dụ 10 giây): sự kiện trễ trong ngưỡng được cập nhật bình thường; quá ngưỡng sẽ bị loại bỏ gọn gàng để giải phóng trạng thái.
3. **Feature Store (Offline–Online Parity):** Sai lệch giữa cách tính đặc trưng lúc train (SQL batch) và lúc serve (code microservice) tạo ra lỗi *training–serving skew* cực kỳ khó phát hiện. Feature Store định nghĩa một công thức duy nhất để đổ đồng bộ vào kho lịch sử (Parquet) lẫn kho độ trễ thấp (Redis).
4. **Ngữ nghĩa giao hàng (At-Least-Once vs. Exactly-Once) & Idempotency:** Trong mạng phân tán, gói tin Kafka có thể bị gửi lặp — dịch vụ tiêu thụ cần thiết kế khóa chống trùng (*idempotency key*) để không trừ tiền hay ghi điểm gian lận hai lần.

---

## Kiến thức cần nhớ: Bốn mảnh ghép của hệ thống dữ liệu và học máy thời gian thực

1. **Ba thang thời gian (Event-time, Request-time, Batch-time)**:
   Phân biệt rạch ròi giữa *tính toán cận thời gian thực theo sự kiện* (tính bằng giây, do Spark/Flink đảm nhận), *phục vụ yêu cầu đồng bộ* (tính bằng mili-giây, do Redis/Feature Store + Model Server đảm nhận) và *tổng hợp lô định kỳ* (tính bằng giờ, cho khóa sổ/backfill).
2. **Thời gian sự kiện (Event-time) & Ngưỡng Watermark**:
   Dữ liệu từ điện thoại di động có thể mất sóng rồi gửi bù trễ vài phút. *Event-time* là lúc sự kiện thực sự xảy ra, còn *Watermark* quy định hệ thống sẵn sàng chờ dữ liệu đến muộn tối đa bao lâu trước khi chốt cửa sổ tổng hợp và giải phóng RAM trạng thái.
3. **Feature Store & Bài toán chống lệch Huấn luyện–Phục vụ (Train/Serve Skew)**:
   Feature Store lưu hai mặt: kho ngoại tuyến (Iceberg/Parquet để huấn luyện mô hình) và kho trực tuyến (Redis/DynamoDB để đọc dưới 5 ms). Dùng chung một định nghĩa tính đặc trưng cho cả hai nhánh giúp tránh sai lệch công thức giữa lúc huấn luyện và lúc chạy thật.
4. **Tách biệt đơn vị triển khai (Unified API, Decoupled Deployment)**:
   Dù cùng viết bằng DataFrame API, tác vụ Streaming (cần checkpoint ổn định), tác vụ Batch (chạy theo lịch) và dịch vụ Serving (tự co giãn theo RPS) phải triển khai tách rời nhau để tránh xung đột vòng đời.

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

### Giải thích: Chạy tay cơ chế Watermark 10 phút với các sự kiện đến muộn

Xét đúng đoạn mã Structured Streaming ở Mục 1 với cửa sổ 1 giờ `[10:00, 11:00)` và ngưỡng **Watermark = 10 phút**. Quy tắc tính Watermark của Spark là:
$$\text{Watermark} = \max(\text{event\_ts đã thấy}) - 10\text{ phút}.$$

Hãy theo dõi từng sự kiện đi vào Kafka cho cùng một người dùng `U1`:

| Thứ tự đến | Thời điểm sự kiện (`event_ts`) | Số tiền | `max(event_ts)` hiện tại | Watermark hiện tại | Hệ thống xử lý thế nào? |
| :---: | :---: | :---: | :---: | :---: | :--- |
| 1 | `10:45` | 200k | `10:45` | `10:35` | **Chấp nhận** vào cửa sổ `[10:00, 11:00)` → tổng = `200k` |
| 2 | `11:05` | 150k | `11:05` | `10:55` | Vào cửa sổ `[11:00, 12:00)`. Cửa sổ `[10:00, 11:00)` vẫn mở vì `10:55 < 11:00` |
| 3 (trễ 12 phút) | `10:58` | 50k | `11:05` | `10:55` | **Vẫn chấp nhận!** Vì `10:58 ≥ 10:55` → tổng `[10:00, 11:00)` = `250k` |
| 4 | `11:12` | 100k | `11:12` | `11:02` | Vì Watermark `11:02 > 11:00`, cửa sổ `[10:00, 11:00)` được **chốt và ghi ra Iceberg** (`250k`) |
| 5 (trễ quá hạn) | `10:50` | 300k | `11:12` | `11:02` | **Bị loại bỏ (dropped)** vì `10:50 < 11:02` — cửa sổ đã đóng để giải phóng bộ nhớ trạng thái! |

Từ bảng chạy tay này, bạn thấy ngay hai sự thật: Watermark nhỏ (ví dụ 1 phút) giúp giải phóng RAM nhanh và chốt dữ liệu sớm nhưng dễ mất giao dịch đến muộn; Watermark lớn (ví dụ 60 phút) giữ được nhiều dữ liệu muộn hơn nhưng làm chậm thời điểm phát bản ghi trong chế độ `append` và tốn bộ nhớ trạng thái.

## Thách thức

- Mâu thuẫn giữa bản chất dữ liệu đến muộn (late data) và kỳ vọng "dữ liệu tức thời" của bảng điều khiển báo cáo.
- Rủi ro ghi kép (ghi đồng thời vào Kafka và cơ sở dữ liệu) khi không áp dụng mẫu transactional outbox.
- Sai lệch giữa huấn luyện và phục vụ (train/serve skew) nếu công thức tính đặc trưng ở nhánh Streaming khác với nhánh backfill lô.

---

### Ví dụ thực tế làm từng bước: Truy vết từng sự kiện đi qua Watermark 10 giây

Giả sử hệ thống chống gian lận đếm số giao dịch trong từng cửa sổ 1 phút (ví dụ `[10:00, 10:01)`) với **ngưỡng trễ Watermark = 10 giây**. Công thức cập nhật: $$	ext{Watermark} = \max(	ext{Event Time đã thấy}) - 10	ext{s}$$. Truy vết 4 sự kiện đến Kafka lần lượt:

| Thứ tự đến | Event Time của giao dịch | Max Event Time đã thấy | Watermark hiện tại | Quyết định của bộ xử lý |
| :--- | :--- | :--- | :--- | :--- |
| 1 | `10:00:05` | `10:00:05` | `09:59:55` | **Chấp nhận** (vào cửa sổ `[10:00, 10:01)`) |
| 2 | `10:00:20` | `10:00:20` | `10:00:10` | **Chấp nhận** (đẩy Watermark lên `10:00:10`) |
| 3 | `10:00:14` (trễ 6s) | `10:00:20` | `10:00:10` | **Chấp nhận!** (`10:00:14 >= 10:00:10`) |
| 4 | `10:00:02` (trễ 18s) | `10:00:20` | `10:00:10` | **Loại bỏ (Dropped)!** (`10:00:02 < 10:00:10`) |

Bảng 4 dòng này cho thấy bản chất của Watermark: đó là **sự đánh đổi tường minh giữa độ chính xác đầy đủ và độ trễ/bộ nhớ** — tăng ngưỡng trễ từ 10s lên 60s sẽ giữ được sự kiện #4 nhưng buộc hệ thống phải giữ trạng thái lâu gấp 6 lần!

### Giải thích: Bốn kỷ luật thiết kế hệ thống ML thời gian thực

1. **Phân tách luồng sự kiện khỏi luồng suy luận (Inference):** Kafka đóng vai trò bộ đệm giảm chấn (shock absorber); nếu dịch vụ chấm điểm ML chậm lại 2 giây, sự kiện xếp hàng an toàn trên Kafka thay vì mất trắng.
2. **Đo độ trễ ở cả tầng p99 lẫn độ tươi của đặc trưng (Feature Freshness):** Mô hình chạy trong 5 ms nhưng đọc đặc trưng từ bảng batch cũ 6 tiếng vẫn sẽ chấm sai giao dịch gian lận mới tinh.
3. **Giám sát tỷ lệ sự kiện bị loại bởi Watermark:** Nếu tỷ lệ drop vượt 1–2%, mạng di động người dùng đang trễ hơn dự tính — cần nới Watermark hoặc xử lý bù ở nhánh batch (*Lambda/Kappa architecture*).
4. **Dùng cùng một mã nguồn tính đặc trưng cho cả train và serve:** Loại bỏ hoàn toàn hiện tượng mô hình đạt AUC 0,98 trên sổ tay nhưng tụt xuống 0,72 ngoài thực tế.

**Kiểm tra nhanh.** Trong bảng truy vết 4 sự kiện ở trên, nếu sự kiện thứ 5 có Event Time là `10:01:05` đến hệ thống, Watermark mới sẽ nhảy lên mốc mấy giờ — và cửa sổ `[10:00, 10:01)` lúc đó đã được đóng lại chưa?

---

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
