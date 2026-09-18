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

Bài bắt buộc Chương 13 giới thiệu pipeline và nhắc GitHub Actions. Bài tùy chọn này là **mô hình làm việc IUH**: nhóm hai người đặt gì vào pipeline, artifact tới *môi trường có tên* ra sao, và làm sao không để mật khẩu đám mây trong repo. Ví dụ xuyên suốt: **Bảng tin học phần IUH**. Không sao chép đề trường khác.

## Mục tiêu học tập

- Vẽ pipeline **mã nguồn → kiểm → đóng gói → triển khai → quan sát**.
- Giải thích **build một lần, thăng hạng nhiều lần** (cùng digest ảnh ở `staging` và `prod`).
- Kể câu chuyện **OIDC / chứng chỉ ngắn hạn** khi CI nói với đám mây; khóa truy cập dài ngày là mùi lab.
- Viết luật rollback: *ai* bấm và *tín hiệu* nào kích hoạt.

## 1. CI và CD là hai việc

**CI** hỏi: “Thay đổi này *build* và *chạy* trên máy sạch chứ?”  
**CD** hỏi: “Ta *phát hành* artifact đã kiểm tới một môi trường — có duyệt hoặc tự động?”

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

Với bài IUH, **delivery** (prod cách một lần duyệt) an toàn hơn tự deploy.

## 2. Build một lần, thăng hạng digest

Hỏng điển hình: CI gắn thẻ `latest`, staging chạy `latest` thứ Hai, prod build lại `latest` thứ Sáu — đó là **bit khác nhau**.

1. CI tạo artifact bất biến: `ghcr.io/.../api@sha256:…`.
2. Staging và prod kéo **đúng digest đó**.
3. Rollback là trỏ môi trường về **digest trước**, không phải “build lại commit cũ rồi cầu may.”

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

Không có registry vẫn học được ý: lưu ID ảnh vào `artifacts/build.json`.

## 3. Môi trường là chính sách, không phải tên thư mục

| Môi trường | Dữ liệu | Ai deploy | Mạng |
| --- | --- | --- | --- |
| Local / compose | Tổng hợp | Cả cặp | Loopback + cầu do bạn định nghĩa |
| Staging | Tổng hợp / che | Pipeline sau CI xanh | Cùng *hình* prod, nhỏ hơn |
| Prod | Thật hoặc demo được duyệt | Duyệt + chứng chỉ ngắn hạn | Cạnh công + dữ liệu riêng (01-07) |

Cấu hình thuộc **biến môi trường hoặc kho bí mật**, không thuộc ảnh build lại.

<div class="content-box warning-box">
<p><strong>Bí mật trong Git là lab trượt.</strong> Nếu <code>.env</code> có khóa đám mây từng commit, xoay khóa, và chuyển sang secret của repo hoặc OIDC.</p>
</div>

## 4. CI xác thực với đám mây (mặc định 2024)

1. Workflow xin **token OIDC** từ GitHub.
2. Nhà cung cấp tin token cho *đúng* repo, nhánh, vai trò.
3. Vai trò sống vài phút và chỉ deploy tài khoản staging.

Azure federated credentials và GCP Workload Identity Federation cùng câu chuyện. Nếu lab không rời laptop, vẫn viết đoạn: *bạn sẽ dùng gì thay cho khóa 90 ngày*.

## 5. Bước deploy khớp Chương 09 và 13

- **Rolling** — mặc định Deployment; pipeline chờ `kubectl rollout status`.
- **Canary** — chuyển 5–10% lưu lượng, đọc SLI, tiếp tục hoặc hủy.
- **Blue/green** — lật Service hoặc tên DNS sau smoke.

Smoke không phải `curl` đến khi thấy HTML:

1. Gọi `/healthz` và một đường ghi/đọc.
2. Khẳng định 200 và dạng JSON.
3. Nên có `service.version` để biết **digest này** đang sống.

```python
import json
import urllib.request

def smoke(base: str, expected_sha: str) -> None:
    with urllib.request.urlopen(f"{base}/healthz", timeout=5) as res:
        body = json.loads(res.read().decode())
    if body.get("service.version") != expected_sha:
        raise SystemExit(f"sai digest: {body}")
```

## 6. Pipeline một tuần cho cặp đôi

1. **Pull request:** lint + unit test. Không deploy.
2. **Nhánh chính:** test + build ảnh.
3. **Duyệt / `workflow_dispatch`:** deploy staging.
4. Smoke + vài phút RED.
5. **Prod** chỉ khi có rollback viết sẵn.

GitHub Actions là mặc định vì site khóa học cũng deploy bằng nó.

## 7. “Xanh” được phép nghĩa là gì

Pipeline xanh mà chưa chạy trên runner sạch là sân khấu local. Hãy làm CI thất bại khi:

- Ảnh chạy `root` và Dockerfile không có `USER`.
- Test bị `|| true`.
- Compose mở `5432` ra `0.0.0.0`.

## Thách thức

- Runner tự host chưa vá (13-02).
- Ma trận build hết phút miễn phí.
- Hai môi trường cùng kéo `latest`.
- Người duyệt không đọc diff.

## Bài tập

1. Tách script “build rồi SSH rồi compose up” thành CI và CD. Artifact nào vượt ranh giới?
2. Runbook rollback bốn dòng, dùng từ vựng 13-03.
3. Ba quyền vai trò OIDC phải *từ chối*.
4. Smoke chỉ gọi `/` — vì sao chưa đủ cho ứng dụng đa tầng?

## Tiếp theo

- [14-01 Terraform]({{ site.baseurl }}{% multilang_post_url contents/chapter14/21-01-01-14_01_Infrastructure_as_Code %})
- [12-03 FinOps]({{ site.baseurl }}{% multilang_post_url contents/chapter12/21-01-01-12_03_FinOps_Billing_Literacy %})
- [Đồ án]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_04_Capstone %})

## Tài liệu

1. GitHub Actions: workflow, environment, OIDC.
2. SLSA — xem [13-02]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_02_Supply_Chain_Security_and_Observability %}).
3. [infracourse.cloud](https://infracourse.cloud/) — cảm hứng; pipeline IUH là nguyên tác.
