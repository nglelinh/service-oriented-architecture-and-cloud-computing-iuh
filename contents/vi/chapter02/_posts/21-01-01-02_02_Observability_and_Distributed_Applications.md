---
layout: post
title: 02-02 Quan sát hệ thống phân tán và ứng dụng hiện đại (2022–2026)
chapter: '02'
order: 3
owner: Nguyen Le Linh
lang: vi
categories:
- chapter02
lesson_type: optional
---

Phần lý thuyết bắt buộc định nghĩa hệ thống phân tán thông qua ba tính chất: tính tương tranh, không có đồng hồ toàn cục và lỗi độc lập. Bài đọc tùy chọn này trình bày cách các nhóm vận hành thực tế *quan sát* những tính chất đó trong giai đoạn 2022–2026, đồng thời lý giải vì sao trace, metric và kiểm thử hỗn loạn (chaos engineering) đã trở thành một phần thiết yếu của kiến trúc ứng dụng.

> Lý thuyết bắt buộc hiện ở bản tiếng Anh. Đây là bài ứng dụng tùy chọn bằng tiếng Việt.

## Mục tiêu học tập

- Giải thích vì sao chỉ đọc log cục bộ không đủ khi một yêu cầu đi qua nhiều dịch vụ độc lập.
- Mô tả vai trò của OpenTelemetry như một chuẩn đo đạc (telemetry) trung lập với nhà cung cấp (CNCF graduated, tháng 5/2026).
- Liên hệ tính chất lỗi độc lập với thực hành diễn tập sự cố (*chaos engineering*) và chỉ tiêu mức dịch vụ (*SLO*).

## 1. Từ "máy chủ còn hoạt động?" đến "luồng xử lý của người dùng có trọn vẹn?"

Một thao tác thanh toán có thể đi qua API gateway, dịch vụ giỏ hàng, lời gọi gRPC thanh toán, bộ nhớ đệm và hàng đợi cập nhật tồn kho. Trạng thái CPU của từng máy chủ bình thường không bảo đảm *chuỗi nhân quả* của yêu cầu diễn ra thành công. Khả năng quan sát (observability) hiện đại mô hình hóa mỗi yêu cầu như một **dấu vết phân tán (distributed trace)**: một cây gồm nhiều đoạn xử lý (span) gắn chung một `trace_id`.

OpenTelemetry (OTel) gia nhập CNCF năm 2019, đạt trạng thái incubating năm 2021 và chính thức **graduated ngày 11/5/2026**, là một trong những dự án có tốc độ phát triển nhanh nhất của CNCF. Trạng thái graduated khẳng định *định dạng truyền tải (wire format) và bộ SDK* đã đủ ổn định để chuẩn hóa, giúp tổ chức không bị ràng buộc vào agent độc quyền của một nhà cung cấp giám sát duy nhất.

```python
from opentelemetry import trace

tracer = trace.get_tracer("checkout.service")

def charge(order_id: str, amount_cents: int) -> str:
    with tracer.start_as_current_span("charge") as span:
        span.set_attribute("order.id", order_id)
        span.set_attribute("amount.cents", amount_cents)
        return _payments_client.capture(order_id, amount_cents)
```

Span không tự khắc phục xung đột đồng thời (race condition), nhưng giúp hiện rõ thứ tự nhân quả xuyên qua ranh giới các tiến trình — lời giải thực tiễn cho đặc trưng "không có đồng hồ toàn cục".

## 2. Ba tín hiệu quan sát, một khóa tương quan

| Tín hiệu | Câu hỏi trả lời | Hệ thống lưu trữ phổ biến |
| --- | --- | --- |
| Metric | Trạng thái *tổng thể* của hệ thống ra sao? | Prometheus hoặc dịch vụ metric quản lý |
| Log | *Phiên bản (instance) cụ thể này* đã ghi nhận gì? | Loki, CloudWatch, … |
| Trace | *Chặng xử lý nào* bị chậm hoặc phát sinh lỗi? | Jaeger, Tempo, nền tảng APM |

Đồng thuận phổ biến giai đoạn 2022–2026 là: **gắn mã đo đạc một lần theo chuẩn OTel, xuất dữ liệu sang nhiều hệ thống khác nhau**. Cơ chế lan truyền ngữ cảnh theo chuẩn W3C (`traceparent`) qua header HTTP/gRPC chính là hiện thực hóa thực tế của lớp định danh và giao tiếp trong hệ thống phân tán.

<div class="content-box insight-box">
<p><strong>Bài toán định danh ở hai tầng.</strong> Service discovery cho biết địa chỉ của một dịch vụ, còn <code>trace_id</code> định danh một <em>đơn vị công việc</em> xuyên suốt hệ thống. Hệ thống phân tán cần cả hai cơ chế này.</p>
</div>

## 3. Diễn tập lỗi độc lập có kiểm soát

Chaos Monkey của Netflix từng phổ biến ý tưởng chủ động *gây lỗi* để kiểm chứng khả năng chịu lỗi. Trong thập niên 2020, cách tiếp cận này được thu hẹp phạm vi và thực hiện bài bản hơn: tổ chức các buổi diễn tập (game day) gắn với **ngân sách lỗi (error budget)** theo mô hình Google SRE. Nếu SLO đặt mục tiêu 99,9% giao dịch thanh toán thành công, nhóm vận hành có một hạn mức lỗi nhất định mỗi tháng; kiểm thử hỗn loạn giúp xác minh cơ chế thử lại (retry), timeout và cô lập tài nguyên (bulkhead) hoạt động đúng dự kiến thay vì làm cạn ngân sách lỗi chỉ sau một sự cố.

```python
import time
import random

def call_with_budget(fn, timeout_s=0.2, attempts=3):
    for i in range(attempts):
        start = time.monotonic()
        try:
            return fn()
        except TimeoutError:
            if i == attempts - 1:
                raise
            time.sleep(min(timeout_s, 0.05) * (2 ** i) * random.random())
        if time.monotonic() - start > timeout_s:
            raise TimeoutError("hết hạn cục bộ")
```

## 4. Bối cảnh ứng dụng thực tế

- **Service mesh (sidecar hoặc ambient):** Tự động thu thập các chỉ số cơ bản (golden signals) mà không buộc từng nhóm ứng dụng viết lại bộ xuất dữ liệu.
- **Worker hướng sự kiện dùng CloudEvents:** Vẫn cần truyền kèm mã tương quan (correlation ID) trong thông điệp để nối liền trace bất đồng bộ.
- **Hệ thống đa vùng (multi-region active-active):** Trace giúp xác định chính xác vùng địa lý nào đã xử lý yêu cầu của người dùng.

Theo khảo sát CNCF 2024 (công bố tháng 4/2025), **80%** tổ chức tham gia đã vận hành Kubernetes trên môi trường sản xuất. Mật độ dịch vụ tăng kéo theo chi phí và độ phức tạp giám sát tăng mạnh; OpenTelemetry giữ vai trò lớp hợp đồng ổn định ngay cả khi hệ thống lưu trữ phía sau thay đổi.

## Thách thức

- Gắn nhãn có độ phân biệt quá cao (high cardinality, ví dụ đưa `user_id` vào mọi metric) có thể làm quá tải cơ sở dữ liệu chuỗi thời gian.
- Lưu trữ trace cho 100% lưu lượng rất tốn kém; kỹ thuật lấy mẫu theo đuôi (tail-based sampling) lại đòi hỏi bộ thu thập phải tập hợp được toàn bộ trace trước khi quyết định giữ hay bỏ.
- Thực hiện kiểm thử hỗn loạn mà không có phương án hoàn tác (rollback) rõ ràng sẽ tự gây ra sự cố thực sự.

## Video tham khảo

- **Xem trước:** [OpenTelemetry: OpenTelemetry in Five Minutes \| Project Lightning Talk](https://www.youtube.com/watch?v=l8xiNOCIdLY) — CNCF [Cloud Native Computing Foundation] (tiếng Anh). Tóm tắt OpenTelemetry trong năm phút — nền tảng tương quan tín hiệu được trình bày ở đầu bài.
- [Observability and Its Pillars Explained \| Logs, Metrics & Traces Simplified](https://www.youtube.com/watch?v=rJfZyA831fI) — OpenObserve (tiếng Anh). Phân biệt vai trò của log, metric và trace, tương ứng với bảng so sánh ba tín hiệu.
- [Understanding Chaos Engineering](https://www.youtube.com/watch?v=mfEMXKSFtaQ) — Microsoft Developer (tiếng Anh). Trình bày cách diễn tập chủ động với lỗi độc lập trong hệ thống phân tán.

## Bài tập

1. Vẽ biểu đồ tuần tự (sequence diagram) gồm ba dịch vụ và đánh dấu rõ các điểm cần sao chép header `traceparent`.
2. Giải thích trong khoảng bốn câu vì sao biểu đồ độ trễ trung bình có thể che khuất nhóm 1% yêu cầu bị timeout.
3. Đề xuất một kịch bản kiểm thử hỗn loạn mô phỏng "lỗi độc lập" ở tầng bộ nhớ đệm (cache) mà không làm tắt toàn bộ máy ảo.

## Tài liệu tham khảo

1. CNCF, trang dự án *OpenTelemetry* (graduated 11/5/2026): [cncf.io/projects/opentelemetry](https://www.cncf.io/projects/opentelemetry/).
2. Blog CNCF, “OpenTelemetry has graduated… Now what?” (24/7/2026).
3. CNCF, *Cloud Native 2024 Annual Survey*: [cncf.io/reports/cncf-annual-survey-2024](https://www.cncf.io/reports/cncf-annual-survey-2024/).
4. W3C, *Trace Context* (`traceparent`): [w3.org/TR/trace-context](https://www.w3.org/TR/trace-context/).
5. Beyer et al., *Site Reliability Engineering* (Google) — chương về SLO và ngân sách lỗi.
