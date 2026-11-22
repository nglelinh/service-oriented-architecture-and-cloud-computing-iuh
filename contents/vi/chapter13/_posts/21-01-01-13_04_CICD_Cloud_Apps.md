---
layout: post
title: 13-04 CI/CD cho ứng dụng đám mây
chapter: '13'
order: 5
owner: Nguyen Le Linh
lang: vi
categories:
- chapter13
lesson_type: optional
---

Phần bắt buộc của Chương 13 giới thiệu khái niệm pipeline và GitHub Actions. Bài tùy chọn này xây dựng **quy trình thực hành chuẩn cho nhóm hai sinh viên tại IUH**: các bước cần thiết trong pipeline, cách đưa artifact qua từng môi trường, và cách xác thực với đám mây mà không lưu mật khẩu trong kho mã nguồn. Ví dụ xuyên suốt của bài là ứng dụng **Bảng tin học phần IUH**.

## Mục tiêu học tập

- Phác thảo quy trình pipeline đầy đủ: **mã nguồn → kiểm tra → đóng gói → triển khai → quan sát**.
- Giải thích nguyên tắc **build một lần, thăng cấp (promote) qua nhiều môi trường** bằng cùng một mã băm digest ảnh container giữa `staging` và `prod`.
- Trình bày cơ chế xác thực **OIDC / thông tin đăng nhập ngắn hạn** giữa CI và đám mây thay cho khóa truy cập dài hạn.
- Viết quy tắc hoàn tác (rollback) rõ ràng: *ai* thực hiện và *tín hiệu nào* kích hoạt.

## 1. Phân biệt CI và CD

**CI (Continuous Integration)** trả lời câu hỏi: "Thay đổi này có *biên dịch* và *vượt qua kiểm thử* trên một máy sạch hay không?"  
**CD (Continuous Delivery / Deployment)** trả lời câu hỏi: "Làm sao *phát hành* artifact đã kiểm chứng lên môi trường đích — có bước phê duyệt thủ công hoặc tự động?"

```text
push / pull request
        │
        ▼
   [CI] lint + unit test + build ảnh
        │
        ▼
   artifact (digest ảnh, tarball, site tĩnh)
        │
        ▼
   [CD] staging → smoke → (duyệt) → prod
        │
        ▼
   quan sát (health + RED + một trace)
```

Với quy mô đồ án tại IUH, mô hình **Continuous Delivery** (tự động triển khai lên staging, yêu cầu một bước duyệt trước khi lên production) an toàn hơn so với tự động triển khai thẳng lên production.

## 2. Build một lần, thăng cấp theo digest

Một lỗi vận hành phổ biến là CI gắn thẻ `latest`: môi trường staging chạy bản `latest` build hôm thứ Hai, còn môi trường prod lại build lại bản `latest` vào thứ Sáu — dẫn đến hai môi trường thực chất chạy **hai phiên bản mã nhị phân khác nhau**.

Quy trình đúng gồm ba nguyên tắc:

1. CI tạo ra một artifact bất biến gắn với mã băm cụ thể: `ghcr.io/.../api@sha256:…`.
2. Cả `staging` và `prod` đều kéo **chính xác cùng một mã băm digest đó**.
3. Hoàn tác (rollback) đơn giản là trỏ môi trường về **digest ổn định trước đó**, thay vì cố gắng build lại một commit cũ.

{% raw %}
```yaml
jobs:
  verify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Unit tests
        run: python -m pytest
      - name: Build image
        run: docker build -t courseboard-api:${{ github.sha }} ./api
```
{% endraw %}

Ngay cả khi chưa dùng container registry từ xa, bạn vẫn có thể thực hành nguyên tắc này bằng cách ghi lại ID ảnh đã build vào tệp `artifacts/build.json`.

## 3. Phân tách môi trường theo chính sách

| Môi trường | Dữ liệu | Quyền triển khai | Mạng |
| --- | --- | --- | --- |
| Local / Compose | Dữ liệu giả lập | Cả hai thành viên | Loopback và mạng bridge cục bộ |
| Staging | Dữ liệu giả lập hoặc đã ẩn danh | Tự động qua pipeline sau khi CI đạt | Cùng *cấu trúc* với prod nhưng quy mô nhỏ hơn |
| Prod | Dữ liệu thật hoặc demo chính thức | Phê duyệt thủ công + thông tin xác thực ngắn hạn | Cổng vào công khai + tầng dữ liệu riêng tư (bài 01-07) |

Cấu hình khác nhau giữa các môi trường phải được truyền qua **biến môi trường hoặc kho quản lý bí mật (secret manager)**, tuyệt đối không build lại ảnh container riêng cho từng môi trường.

<div class="content-box warning-box">
<p><strong>Không đưa bí mật vào kho Git.</strong> Nếu tệp <code>.env</code> chứa khóa đám mây vô tình bị commit, hãy thu hồi và tạo lại khóa mới ngay lập tức, sau đó chuyển sang dùng kho secret của repository hoặc xác thực OIDC.</p>
</div>

## 4. Xác thực từ CI tới đám mây bằng OIDC

Thay vì lưu khóa truy cập cố định dài ngày (như khóa 90 ngày), tiêu chuẩn hiện nay dùng OIDC ngắn hạn:

1. Workflow trên GitHub Actions yêu cầu một **token OIDC** tạm thời.
2. Nhà cung cấp đám mây xác minh token này khớp *đúng* kho mã nguồn, nhánh và vai trò (role) được phép.
3. Vai trò tạm thời chỉ có hiệu lực trong vài phút và giới hạn quyền triển khai trong phạm vi tài khoản staging.

Azure Federated Credentials và GCP Workload Identity Federation đều vận hành theo cùng nguyên lý này. Nếu bài lab chỉ chạy cục bộ trên máy cá nhân, bạn vẫn nên trình bày rõ trong báo cáo phương án thay thế khóa truy cập dài hạn khi đưa lên đám mây.

## 5. Chiến lược triển khai và kiểm thử nhanh (smoke test)

Các chiến lược triển khai ở Chương 09 và Chương 13 được tích hợp trực tiếp vào bước CD:

- **Rolling update:** Chiến lược mặc định của Kubernetes Deployment; pipeline chờ `kubectl rollout status` hoàn tất.
- **Canary:** Chuyển trước 5–10% lưu lượng sang phiên bản mới, đối chiếu chỉ số SLI rồi quyết định mở rộng tiếp hoặc hủy bỏ.
- **Blue/green:** Chuyển định tuyến Service hoặc bản ghi DNS sang môi trường mới sau khi vượt qua kiểm thử nhanh.

Kiểm thử nhanh (**smoke test**) sau khi triển khai không chỉ là gọi `curl` kiểm tra có trả về HTML hay không, mà cần:

1. Gọi `/healthz` và thực hiện một luồng đọc/ghi cơ bản.
2. Xác nhận mã trạng thái HTTP 200 và cấu trúc JSON hợp lệ.
3. Đối chiếu trường `service.version` để chắc chắn **đúng mã băm digest mới** đang phục vụ.

```python
import json
import urllib.request

def smoke(base: str, expected_sha: str) -> None:
    with urllib.request.urlopen(f"{base}/healthz", timeout=5) as res:
        body = json.loads(res.read().decode())
    if body.get("service.version") != expected_sha:
        raise SystemExit(f"sai digest: {body}")
```

## 6. Quy trình pipeline gợi ý trong một tuần cho nhóm hai người

1. **Khi mở Pull Request:** Chạy lint và kiểm thử đơn vị (unit test); không thực hiện triển khai.
2. **Khi hợp nhất vào nhánh chính:** Chạy lại kiểm thử và đóng gói ảnh container.
3. **Kích hoạt triển khai (`workflow_dispatch` hoặc phê duyệt):** Triển khai lên môi trường staging.
4. Chạy smoke test và theo dõi bộ chỉ số RED trong vài phút.
5. **Triển khai lên Production:** Chỉ thực hiện khi đã có quy trình rollback chuẩn bị sẵn.

GitHub Actions được chọn làm ví dụ mặc định vì bản thân trang web khóa học cũng sử dụng nền tảng này để triển khai.

## 7. Tiêu chí để một pipeline được coi là đạt chuẩn

Một quy trình CI chỉ thực sự đáng tin khi chạy trên môi trường sạch (clean runner) và chủ động báo lỗi nếu phát hiện rủi ro cấu hình, chẳng hạn:

- Ảnh container chạy bằng quyền `root` và Dockerfile thiếu chỉ thị `USER`.
- Lệnh chạy kiểm thử bị che lỗi bằng `|| true`.
- Tệp Docker Compose mở cổng cơ sở dữ liệu `5432` ra toàn bộ giao diện `0.0.0.0`.

## Thách thức

- CI runner tự vận hành chậm cập nhật bản vá bảo mật (xem bài 13-02).
- Ma trận build quá lớn làm cạn hạn mức phút chạy miễn phí.
- Hai môi trường cùng kéo thẻ `latest` gây sai lệch phiên bản.
- Bước phê duyệt triển khai trở thành hình thức khi người duyệt không đọc kỹ thay đổi (diff).

## Video tham khảo

- **Xem trước:** [CI/CD Explained in 5 Minutes](https://www.youtube.com/watch?v=ddDJxFnv-qs) — TechWorld with Nana (tiếng Anh). Giải thích phân biệt CI và CD trong năm phút.
- [How to use GitHub Actions \| GitHub for Beginners](https://www.youtube.com/watch?v=BQrohJ3PT7I) — GitHub (tiếng Anh). Hướng dẫn nhập môn GitHub Actions — công cụ CI được dùng trong bài.
- [Authenticate GitHub Actions with AWS Using OIDC — No Secrets Needed](https://www.youtube.com/watch?v=Sdzd4N6L5Hg) — Integrations Ninjas (tiếng Anh). Minh họa cách cấu hình GitHub Actions xác thực với AWS qua OIDC mà không cần lưu khóa bí mật dài hạn.

## Bài tập

1. Tách kịch bản thủ công "build rồi SSH lên máy chủ rồi chạy `compose up`" thành hai giai đoạn CI và CD rõ ràng; xác định artifact nào được truyền qua ranh giới hai giai đoạn.
2. Viết một quy trình hoàn tác (rollback runbook) dài bốn dòng có sử dụng hệ thuật ngữ quan sát từ bài 13-03.
3. Liệt kê ba quyền mà vai trò OIDC dùng cho CI cần *từ chối* rõ ràng theo nguyên tắc đặc quyền tối thiểu.
4. Giải thích vì sao một smoke test chỉ kiểm tra đường dẫn `/` là chưa đủ đối với ứng dụng đa tầng.

## Đọc tiếp

- [14-01 Terraform]({{ site.baseurl }}{% multilang_post_url contents/chapter14/21-01-01-14_01_Infrastructure_as_Code %})
- [12-03 FinOps]({{ site.baseurl }}{% multilang_post_url contents/chapter12/21-01-01-12_03_FinOps_Billing_Literacy %})
- [Đồ án]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_04_Capstone %})

## Tài liệu tham khảo

1. Tài liệu GitHub Actions: workflow, environment và OIDC.
2. Khung SLSA — xem [13-02]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_02_Supply_Chain_Security_and_Observability %}).
3. [infracourse.cloud](https://infracourse.cloud/) — nguồn tham khảo cách tiếp cận; quy trình pipeline cho IUH được biên soạn độc lập.
