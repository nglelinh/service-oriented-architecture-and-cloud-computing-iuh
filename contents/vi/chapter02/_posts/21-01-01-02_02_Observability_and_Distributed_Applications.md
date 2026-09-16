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

Bài bắt buộc định nghĩa hệ thống phân tán qua tương tranh, không có đồng hồ toàn cục, và lỗi độc lập. Ghi chú tùy chọn này cho thấy nhóm production *quan sát* các tính chất đó thế nào trong 2022–2026, và vì sao trace, metric cùng thí nghiệm chaos nay thuộc về ứng dụng chứ không phải phụ lục.

> Lý thuyết bắt buộc đang ở bản tiếng Anh. Đây là phần ứng dụng tùy chọn bằng tiếng Việt.

## Mục tiêu học tập

- Giải thích vì sao chỉ xem log sẽ thất bại khi một yêu cầu tỏa ra nhiều dịch vụ.
- Mô tả OpenTelemetry như hợp đồng telemetry trung lập nhà cung cấp (CNCF graduated, tháng 5/2026).
- Nối lỗi độc lập với vận hành theo *chaos* và *SLO*.

## 1. Từ “máy còn sống?” sang “hành trình người dùng còn nguyên?”

Một lần thanh toán có thể chạm gateway, dịch vụ giỏ, stub gRPC thanh toán, cache và topic kho. CPU máy chủ xanh không có nghĩa *đường nhân quả* thành công. Quan sát hiện đại coi yêu cầu là **distributed trace**: cây span dùng chung `trace_id`.

OpenTelemetry (OTel) vào CNCF năm 2019, incubating 2021, và **graduated ngày 11/5/2026**. CNCF xếp đây là một trong các dự án vận tốc cao nhất. Graduation không phải khẩu hiệu: đó là tín hiệu *định dạng dây và SDK* đủ ổn để chuẩn hóa, thay vì khóa mọi dịch vụ vào một agent nhà cung cấp.

```python
from opentelemetry import trace

tracer = trace.get_tracer("checkout.service")

def charge(order_id: str, amount_cents: int) -> str:
    with tracer.start_as_current_span("charge") as span:
        span.set_attribute("order.id", order_id)
        span.set_attribute("amount.cents", amount_cents)
        return _payments_client.capture(order_id, amount_cents)
```

Span không sửa race. Nó làm race **nhìn thấy được** xuyên ranh giới tiến trình — câu trả lời thực tiễn cho “không có đồng hồ toàn cục.”

## 2. Ba tín hiệu, một khóa tương quan

| Tín hiệu | Trả lời | Backend điển hình |
| --- | --- | --- |
| Metric | Hệ thống khỏe *tổng thể*? | Prometheus / metric quản lý |
| Log | *Instance này* phát gì? | Loki, CloudWatch, … |
| Trace | *Đường nào* chậm hoặc gãy? | Jaeger, Tempo, APM |

Đồng thuận 2022–2026: **instrument một lần (OTel), xuất nhiều nơi**. Lan truyền ngữ cảnh (W3C `traceparent`) chính là lớp đặt tên/giao tiếp của bài bắt buộc, hiện thực hóa bằng header HTTP/gRPC.

<div class="content-box insight-box">
<p><strong>Đặt tên trở lại.</strong> Service discovery cho bạn một địa chỉ. <code>trace_id</code> cho bạn danh tính của một <em>đơn vị công việc</em>. Hệ thống phân tán cần cả hai.</p>
</div>

## 3. Lỗi độc lập, tập có chủ đích

Chaos Monkey của Netflix phổ biến việc *bơm* lỗi. Phiên bản những năm 2020 hẹp và khoa học hơn: game day theo **ngân sách lỗi** (Google SRE). Nếu SLO là 99,9% thanh toán thành công, bạn có ngân sách tháng cho các yêu cầu thất bại. Chaos kiểm tra retry, timeout và bulkhead có tiêu ngân sách chậm — hay một lần hết sạch.

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

## 4. Ứng dụng sinh viên sẽ gặp

- **Service mesh / sidecar hoặc ambient** xuất golden signal mà không cần mỗi nhóm viết lại exporter.
- Worker **hướng sự kiện** với CloudEvent: vẫn cần correlation id trên tin nhắn.
- Active-active **đa vùng**: trace là cách trung thực để thấy vùng nào phục vụ người dùng.

Khảo sát CNCF 2024 (công bố 4/2025) ghi **80%** dùng Kubernetes trên production. Chi phí và độ phức tạp quan sát tăng theo mật độ đó; OTel là nỗ lực giữ *hợp đồng* ổn khi backend đổi.

## Thách thức

- Nhãn cardinality cao (`user_id` trên mọi metric) có thể làm sập time-series DB.
- Trace 100% lưu lượng thì đắt; sampling theo đuôi cần collector thấy cả trace.
- Chaos không có kế hoạch rollback chỉ là sự cố.

## Bài tập

1. Vẽ sequence diagram ba dịch vụ. Đánh dấu chỗ phải sao chép header `traceparent`.
2. Giải thích bốn câu vì sao dashboard độ trễ trung bình có thể giấu lớp timeout 1%.
3. Đề xuất một thí nghiệm chaos cho “lỗi độc lập” của cache (không phải cả VM).

## Tài liệu tham khảo

1. CNCF, trang dự án *OpenTelemetry* (graduated 11/5/2026): [cncf.io/projects/opentelemetry](https://www.cncf.io/projects/opentelemetry/).
2. Blog CNCF, “OpenTelemetry has graduated… Now what?” (24/7/2026).
3. CNCF, *Cloud Native 2024 Annual Survey*: [cncf.io/reports/cncf-annual-survey-2024](https://www.cncf.io/reports/cncf-annual-survey-2024/).
4. W3C, *Trace Context* (`traceparent`): [w3.org/TR/trace-context](https://www.w3.org/TR/trace-context/).
5. Beyer et al., *Site Reliability Engineering* (Google) — chương SLO/ngân sách lỗi.
