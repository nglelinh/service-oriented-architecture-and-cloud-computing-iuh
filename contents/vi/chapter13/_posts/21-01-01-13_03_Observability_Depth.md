---
layout: post
title: 13-03 Quan sát sâu — log, metric, trace, OpenTelemetry
chapter: '13'
order: 4
owner: Nguyen Le Linh
lang: vi
categories:
- chapter13
lesson_type: optional
---

Bài bắt buộc Chương 13 nhắc giám sát và nhật ký. Bài tùy chọn Chương 02 giải thích vì sao cần trace khi không có đồng hồ toàn cục. Bài 13-02 gắn OpenTelemetry với canary và artifact đã ký. Bài này là **lượt sâu**: mỗi tín hiệu *để trả lời câu gì*, OpenTelemetry chuyển dữ liệu ra sao, và nhóm IUH tránh bảng điều khiển không trả lời được “vì sao checkout chậm?”

## Mục tiêu học tập

- Tách **log, metric, trace** theo câu hỏi, rồi tương quan bằng một ngữ cảnh (`trace_id` / `service.version`).
- Mô tả đường OpenTelemetry: SDK → (Collector tùy chọn) → backend, gồm header W3C `traceparent`.
- Chọn một bộ **SLI/SLO** nhỏ và nói thứ bạn *không* cảnh báo.
- Gọi tên cardinality và sampling — hai cách quan sát vừa tính tiền vừa nói dối.

## 1. Ba tín hiệu, ba việc

| Tín hiệu | Câu hỏi | Hỏng nếu chỉ có cái này |
| --- | --- | --- |
| **Metric** | Dịch vụ có khỏe *trong tổng thể* phút này? | Thấy “p95 = 800 ms” mà không chỉ ra phụ thuộc chậm. |
| **Log** | Tiến trình này nói gì về sự kiện này? | Ngập dòng, không theo một người dùng qua bốn container. |
| **Trace** | Yêu cầu đi *đường nhân quả* nào? | Có đồ thị đẹp, không có tín hiệu năng lực (CPU, hàng đợi). |

Hợp đồng những năm 2020: **nhúng một lần**, xuất nhiều nơi. OpenTelemetry (OTel) vào CNCF năm 2019, ủ năm 2021, **tốt nghiệp 11 tháng 5 năm 2026**.

```text
request ──► span "edge"
               ├── span "api.http"
               │      ├── span "db.query"
               │      └── span "cache.get"
               └── span "worker.publish"
```

**Span** là một thao tác có thời gian. **Trace** là cây span chung `trace_id`. Log nên mang `trace_id`. Metric nên mang `service.name` và `service.version` để canary hiện thành *chuỗi thời gian mới*.

## 2. RED, USE, và metric thật sự cần

- **RED** (dịch vụ theo yêu cầu): **R**ate, **E**rrors, **D**uration.
- **USE** (tài nguyên): **U**tilization, **S**aturation, **E**rrors.

API Bảng tin cần RED trên cạnh HTTP và USE trên container cơ sở dữ liệu. Ngày đầu không cần 400 metric tự viết.

```python
import time
from collections import Counter

_requests = Counter()
_errors = Counter()
_latency_ms = []

def observe(route: str, status: int, started: float) -> None:
    _requests[route] += 1
    if status >= 500:
        _errors[route] += 1
    _latency_ms.append((route, (time.monotonic() - started) * 1000))
```

<div class="content-box insight-box">
<p><strong>SLI / SLO / SLA.</strong> SLI là phép đo. SLO là đích bạn chọn (ví dụ 99,5% GET Bảng tin dưới 300 ms trong một tuần). SLA là <em>hợp đồng</em> có hậu quả. Sinh viên viết SLO. Đừng bịa SLA cho một lab.</p>
</div>

## 3. Log rẻ khi viết, đắt khi dùng

Log lab tốt là **có cấu trúc** (JSON) và theo sự kiện. Log xấu là `print("here")`.

1. Ghi **đầu và cuối** lời gọi ngoài, không phải mọi dòng nghiệp vụ.
2. Không ghi mật khẩu, cookie phiên, MSSV thô. Băm hoặc bỏ.
3. Đưa log container ra **stdout/stderr** (Chương 08).
4. Log không có `trace_id` là bưu thiếp không địa chỉ.

## 4. OpenTelemetry là đường ống, không phải sản phẩm

1. **API** — lời gọi trong mã (`start_as_current_span`).
2. **SDK** — lấy mẫu, thuộc tính tài nguyên (`service.name=courseboard-api`).
3. **Instrumentation** — bọc FastAPI, HTTPX, gRPC.
4. **Collector** (tùy chọn) — nhận OTLP, xuất Jaeger / Prometheus / Tempo / nhà cung cấp.
5. **Lan truyền ngữ cảnh** — W3C Trace Context (`traceparent`).

```python
from opentelemetry import trace
from opentelemetry.propagate import inject

tracer = trace.get_tracer("courseboard.api")

def call_worker(http_post, url: str, payload: dict) -> None:
    with tracer.start_as_current_span("worker.publish") as span:
        span.set_attribute("messaging.destination", "board.events")
        headers = {}
        inject(headers)
        http_post(url, json=payload, headers=headers)
```

Lab Lộ trình triển khai có thể **không** cần Collector: in `trace_id`, hoặc một container Jaeger all-in-one. Chuẩn đầu ra là *đường đi*, không phải một SaaS.

## 5. Sampling, cardinality, và lời nói dối

**Nổ cardinality.** Nhãn `user_id` biến một chuỗi thời gian thành hàng chục nghìn. Nhãn phải *ít giá trị*: `route`, `code`, `region`, `version`. Định danh duy nhất để trên *span và log*.

**Lấy mẫu.** Giữ 5% trace thì trượt mất lần thanh toán lỗi. Với lab IUH, **giữ 100%** đến khi có tải thật.

```text
nhãn tốt:  http.route=/api/posts  http.status=500  service.version=sha-8f3c
nhãn xấu:  user=sv123456  email=a@iuh.edu.vn  query="select ..."
```

## 6. Cảnh báo nhóm hai người có thể thức vì nó

| Đánh thức | Để bản tin tuần |
| --- | --- |
| Tỷ lệ lỗi $$2\times$$ nền 10 phút | Một 5xx đơn lẻ |
| Đốt SLO hết ngân sách lỗi trong vài giờ | p99 xấu chậm trên route admin không dùng |
| Đĩa > 90% volume cơ sở dữ liệu | Một dòng debug bạn không thích |

Gắn cảnh báo với **runbook** năm dòng: xem gì, rollback ra sao (Chương 09 / 13-04), ai giữ DNS nếu chứng chỉ hết hạn.

## 7. Quan sát tối thiểu cho Bảng tin

1. Metric RED trên API.
2. Log JSON có `trace_id` và `service.version`.
3. Một trace mỗi HTTP, con cho DB và HTTP ra.
4. `/healthz` kiểm cơ sở dữ liệu.
5. Ảnh panel khi **cố ý làm hỏng** (tắt DB).

<div class="content-box exercise-box">
<p><strong>Thành thật với lab.</strong> Dashboard xanh mà chưa từng tiêm lỗi không chứng minh bạn quan sát được. Hãy phá có chủ đích.</p>
</div>

## Thách thức

- Tự động nhúng span ồn và giấu truy vấn chậm.
- Đồng hồ: thời lượng span là địa phương (Chương 02).
- Quyền riêng tư: query string mang tên sinh viên.
- Khóa nhà cung cấp trên API agent.

## Bài tập

1. Với `POST /api/posts`, viết một SLI RED, một SLI USE cho Postgres, một SLO cho demo một tuần.
2. Trace: span API 1,2 s, span DB 20 ms. Bước tiếp theo là gì?
3. Nhãn an toàn hay không: `http.status`, `tenant_id` (3 tenant), `session_id`, `sql_text`.
4. Collector làm gì mà SDK một mình không làm? Bốn câu.

## Tiếp theo

- [13-02]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_02_Supply_Chain_Security_and_Observability %})
- [13-04 CI/CD]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_04_CICD_Cloud_Apps %})
- [Lab A]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_01_Lab_Docker_Local %})

## Tài liệu

1. Tài liệu OpenTelemetry và thông báo tốt nghiệp CNCF (11/5/2026).
2. W3C Trace Context.
3. SRE workbook (công khai); USE (Brendan Gregg); RED.
4. [infracourse.cloud](https://infracourse.cloud/) — cảm hứng; bài IUH là nguyên tác.
