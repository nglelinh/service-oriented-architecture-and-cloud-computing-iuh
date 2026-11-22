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

Phần bắt buộc của Chương 13 giới thiệu giám sát và ghi log; bài tùy chọn Chương 02 lý giải vai trò của trace trong hệ thống không có đồng hồ toàn cục; còn bài 13-02 gắn OpenTelemetry với phát hành canary và xác minh artifact. Bài học này đi sâu hơn vào kỹ thuật thực hành: mỗi loại tín hiệu *trả lời câu hỏi nào*, dữ liệu di chuyển qua kiến trúc OpenTelemetry ra sao, và cách xây dựng hệ thống giám sát trả lời trực tiếp câu hỏi "vì sao thao tác thanh toán bị chậm?".

## Mục tiêu học tập

- Phân biệt **log, metric, trace** theo câu hỏi cần giải đáp và liên kết chúng qua ngữ cảnh chung (`trace_id` và `service.version`).
- Mô tả luồng dữ liệu OpenTelemetry: SDK → Collector (tùy chọn) → hệ thống lưu trữ, bao gồm header W3C `traceparent`.
- Lựa chọn một bộ chỉ số **SLI/SLO** gọn nhẹ và xác định rõ những cảnh báo *không* nên bật.
- Nhận diện hai nguồn sai lệch và phát sinh chi phí phổ biến: bùng nổ cardinality và lấy mẫu (sampling) không phù hợp.

## 1. Ba loại tín hiệu và vai trò riêng biệt

| Tín hiệu | Câu hỏi chính | Hạn chế nếu chỉ dùng riêng tín hiệu này |
| --- | --- | --- |
| **Metric** | Trạng thái *tổng thể* của dịch vụ lúc này ra sao? | Biết "p95 = 800 ms" nhưng không chỉ ra được dịch vụ phụ thuộc nào gây chậm. |
| **Log** | Tiến trình cụ thể này đã ghi nhận sự kiện gì? | Khối lượng lớn, khó lần theo một yêu cầu đi qua nhiều container. |
| **Trace** | Yêu cầu đi theo *chuỗi nhân quả* nào? | Nhìn rõ luồng gọi nhưng thiếu cái nhìn tổng hợp về tải tài nguyên (CPU, hàng đợi). |

Nguyên tắc chuẩn hóa của thập niên 2020 là **gắn mã đo đạc một lần**, xuất dữ liệu tới nhiều hệ thống phân tích khác nhau. OpenTelemetry (OTel) gia nhập CNCF năm 2019, đạt trạng thái incubating năm 2021 và **chính thức graduated ngày 11 tháng 5 năm 2026**.

```text
request ──► span "edge"
               ├── span "api.http"
               │      ├── span "db.query"
               │      └── span "cache.get"
               └── span "worker.publish"
```

**Span** đại diện cho một thao tác có thời điểm bắt đầu và kết thúc. **Trace** là cây gồm các span liên kết nhau qua cùng một `trace_id`. Để tương quan hiệu quả, log cần đính kèm `trace_id`, còn metric cần gắn nhãn `service.name` và `service.version` để mỗi phiên bản canary hiện lên thành một *chuỗi thời gian tách biệt*.

## 2. Phương pháp RED, USE và các chỉ số thiết yếu

- **RED** (dành cho dịch vụ xử lý yêu cầu): **R**ate (tần suất yêu cầu), **E**rrors (tỷ lệ lỗi), **D**uration (thời gian phản hồi).
- **USE** (dành cho tài nguyên hệ thống): **U**tilization (mức sử dụng), **S**aturation (mức bão hòa/hàng đợi), **E**rrors (lỗi tài nguyên).

Với ứng dụng Bảng tin học phần, bạn chỉ cần thu thập nhóm RED tại tầng HTTP API và nhóm USE trên container cơ sở dữ liệu, thay vì tạo hàng trăm metric không cần thiết ngay từ đầu.

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
<p><strong>Phân biệt SLI, SLO và SLA.</strong> SLI là phép đo định lượng thực tế. SLO là mục tiêu nội bộ bạn đặt ra (ví dụ 99,5% yêu cầu GET Bảng tin phản hồi dưới 300 ms trong một tuần). SLA là <em>cam kết hợp đồng</em> kèm điều khoản bồi thường. Trong đồ án sinh viên, hãy định nghĩa rõ SLI và SLO, không cần đặt ra SLA giả định.</p>
</div>

## 3. Ghi log có cấu trúc và tiết kiệm chi phí

Log hiệu quả cần ở dạng **có cấu trúc** (JSON) và gắn với sự kiện cụ thể, thay vì các dòng tự do kiểu `print("here")`. Bốn quy tắc thực hành:

1. Ghi nhận **điểm vào và điểm ra** của các lời gọi dịch vụ bên ngoài, tránh ghi log tràn lan ở mọi vòng lặp nghiệp vụ.
2. Tuyệt đối không ghi mật khẩu, cookie phiên đăng nhập hay mã số sinh viên dạng thô; cần băm (hash) hoặc lọc bỏ.
3. Xuất log của container ra luồng chuẩn **stdout/stderr** (theo nguyên lý Chương 08).
4. Luôn đính kèm `trace_id` để có thể liên kết log trực tiếp với trace tương ứng.

## 4. Kiến trúc đường ống OpenTelemetry

OpenTelemetry bao gồm năm thành phần chính:

1. **API:** Tập giao diện gọi trong mã ứng dụng (như `start_as_current_span`).
2. **SDK:** Xử lý lấy mẫu và gán thuộc tính tài nguyên (`service.name=courseboard-api`).
3. **Thư viện Instrumentation:** Tự động gắn đo đạc cho các framework như FastAPI, HTTPX, gRPC.
4. **Collector (tùy chọn):** Thành phần trung gian tiếp nhận dữ liệu OTLP và chuyển tiếp tới Jaeger, Prometheus, Tempo hoặc dịch vụ đám mây.
5. **Lan truyền ngữ cảnh:** Truyền thông tin trace qua header chuẩn W3C Trace Context (`traceparent`).

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

Trong bài thực hành Lộ trình triển khai, bạn chưa nhất thiết phải dựng riêng một Collector: có thể in trực tiếp `trace_id` ra log hoặc chạy một container Jaeger all-in-one. Mục tiêu chính là nắm vững *luồng đi của dữ liệu đo*, không phụ thuộc vào một nền tảng SaaS cụ thể.

## 5. Lấy mẫu (sampling) và bùng nổ cardinality

**Bùng nổ cardinality:** Đưa các giá trị có số lượng biến thể rất lớn (như `user_id` hoặc `session_id`) làm nhãn metric sẽ tạo ra hàng chục nghìn chuỗi thời gian, làm tăng mạnh chi phí và gây quá tải bộ nhớ. Nhãn metric phải có *tập giá trị hữu hạn* như `route`, `code`, `region`, `version`; còn các mã định danh chi tiết nên đặt ở thuộc tính của *span và log*.

**Lấy mẫu (sampling):** Nếu chỉ giữ 5% lượng trace một cách ngẫu nhiên, bạn rất dễ bỏ sót đúng giao dịch thanh toán bị lỗi. Ở quy mô bài lab IUH, nên **giữ lại 100%** dữ liệu trace cho đến khi hệ thống có lưu lượng lớn thực sự.

```text
nhãn tốt:  http.route=/api/posts  http.status=500  service.version=sha-8f3c
nhãn xấu:  user=sv123456  email=a@iuh.edu.vn  query="select ..."
```

## 6. Thiết lập cảnh báo hợp lý cho nhóm nhỏ

| Sự cố cần cảnh báo khẩn | Sự kiện chỉ cần tổng hợp báo cáo tuần |
| --- | --- |
| Tỷ lệ lỗi tăng $$2\times$$ mức nền trong 10 phút liên tiếp | Một phản hồi 5xx xuất hiện đơn lẻ |
| Tốc độ tiêu hao ngân sách lỗi nhanh, có nguy cơ cạn SLO trong vài giờ | Độ trễ p99 tăng nhẹ trên một đường dẫn quản trị ít dùng |
| Dung lượng đĩa của volume cơ sở dữ liệu vượt 90% | Một dòng log debug bất thường không ảnh hưởng người dùng |

Mỗi cảnh báo khẩn nên đi kèm một **quy trình xử lý nhanh (runbook)** ngắn gọn khoảng năm dòng: cần kiểm tra chỉ số nào trước, cách hoàn tác phiên bản (Chương 09 / 13-04) và ai là người giữ quyền cập nhật DNS nếu chứng chỉ TLS hết hạn.

## 7. Mức quan sát tối thiểu cho ứng dụng Bảng tin học phần

1. Thu thập bộ chỉ số RED trên tầng API.
2. Ghi log JSON có kèm `trace_id` và `service.version`.
3. Tạo một trace cho mỗi yêu cầu HTTP, gồm các span con cho truy vấn cơ sở dữ liệu và lời gọi HTTP ra ngoài.
4. Cung cấp endpoint `/healthz` kiểm tra kết nối thực tế tới cơ sở dữ liệu.
5. Chụp lại bằng chứng bảng giám sát khi **chủ động gây lỗi** (chẳng hạn tạm dừng container cơ sở dữ liệu).

<div class="content-box exercise-box">
<p><strong>Kiểm chứng thực tế trong bài lab.</strong> Một bảng điều khiển luôn hiển thị màu xanh khi chưa từng thử nghiệm lỗi không chứng minh được hệ thống quan sát hoạt động đúng. Hãy chủ động tiêm lỗi để kiểm tra tín hiệu phản hồi.</p>
</div>

## Thách thức

- Cơ chế tự động instrumentation có thể sinh ra quá nhiều span vụn, làm che khuất truy vấn thực sự chậm.
- Sai lệch đồng hồ: mốc thời gian của mỗi span dựa trên đồng hồ cục bộ của từng nút (xem Chương 02).
- Rủi ro quyền riêng tư nếu chuỗi truy vấn URL (query string) chứa tên hoặc mã số sinh viên được ghi nguyên văn vào log.
- Ràng buộc nhà cung cấp nếu mã nguồn gọi trực tiếp API của một agent độc quyền thay vì OpenTelemetry.

## Video tham khảo

- **Xem trước:** [OpenTelemetry: OpenTelemetry in Five Minutes \| Project Lightning Talk](https://www.youtube.com/watch?v=l8xiNOCIdLY) — CNCF [Cloud Native Computing Foundation] (tiếng Anh). Trình bày kiến trúc đường ống OpenTelemetry trong năm phút.
- [Observability and Its Pillars Explained \| Logs, Metrics & Traces Simplified](https://www.youtube.com/watch?v=rJfZyA831fI) — OpenObserve (tiếng Anh). Phân biệt vai trò của log, metric và trace, khớp với phần mở đầu bài học.
- [OpenTelemetry: Simplifying Hybrid Cloud Monitoring](https://www.youtube.com/watch?v=hLvwoow3XTk) — IBM Technology (tiếng Anh). Minh họa cách triển khai OpenTelemetry Collector và backend trong môi trường đám mây lai.

## Bài tập

1. Đối với endpoint `POST /api/posts`, xác định một SLI theo RED, một SLI theo USE cho PostgreSQL và một SLO phù hợp cho tuần chạy thử nghiệm.
2. Một trace ghi nhận tổng thời gian span API là 1,2 giây, trong khi span truy vấn cơ sở dữ liệu chỉ mất 20 ms. Bạn sẽ kiểm tra những nguyên nhân nào tiếp theo?
3. Đánh giá các nhãn metric sau có an toàn về mặt cardinality hay không: `http.status`, `tenant_id` (hệ thống có 3 tenant), `session_id`, `sql_text`.
4. Trình bày trong khoảng bốn câu những vai trò mà OpenTelemetry Collector đảm nhận mà riêng SDK trong ứng dụng không làm được.

## Đọc tiếp

- [13-02]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_02_Supply_Chain_Security_and_Observability %})
- [13-04 CI/CD]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_04_CICD_Cloud_Apps %})
- [Lab A]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_01_Lab_Docker_Local %})

## Tài liệu tham khảo

1. Tài liệu OpenTelemetry và thông báo trạng thái graduated của CNCF (11/5/2026).
2. Đặc tả W3C Trace Context.
3. *The Site Reliability Workbook* (công khai); phương pháp USE (Brendan Gregg) và phương pháp RED.
4. [infracourse.cloud](https://infracourse.cloud/) — nguồn tham khảo cách tiếp cận; nội dung bài học IUH được biên soạn độc lập.
