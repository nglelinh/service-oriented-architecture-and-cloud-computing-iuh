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

Hãy thử hỏi bất kỳ đội phần mềm nào từng triển khai thủ công bằng cách SSH vào máy chủ, gõ `git pull` rồi `docker compose up -d`: sớm muộn gì cũng sẽ có một tối thứ Sáu ai đó quên chạy lệnh di trú cơ sở dữ liệu, đẩy nhầm biến môi trường của máy phát triển lên máy thật, hoặc lỡ commit khóa bí mật AWS (`AWS_SECRET_ACCESS_KEY`) thẳng lên kho GitHub công khai!

**CI/CD (Continuous Integration / Continuous Delivery)** biến việc phát hành phần mềm từ một nghi lễ căng thẳng đầy rủi ro thành một dây chuyền tự động, lặp lại được và kiểm chứng chặt chẽ ở từng cổng. Bài học này trình bày chuẩn mực xây dựng pipeline đám mây hiện đại cho nhóm nhỏ: *build một lần, thăng cấp theo digest, xác thực không mật khẩu bằng OIDC, và kiểm thử khói (smoke test) tự động*.

## Bản đồ kiến thức nền: Bốn nguyên tắc vàng của đường ống CI/CD đám mây hiện đại

1. **Tách bạch CI (Tích hợp liên tục) và CD (Chuyển giao / Triển khai liên tục)**:
   **CI** chạy trên mọi Pull Request để xác minh chất lượng (lint, unit test, quét lỗ hổng, đóng gói artifact). **CD** tiếp nhận artifact đã đạt chuẩn CI để đưa dần qua các môi trường (`Dev → Staging → Production`) kèm chính sách phê duyệt và kiểm thử sau triển khai.

2. **"Build một lần, thăng cấp theo Digest" (Build Once, Promote Many)**:
   Sai lầm phổ biến nhất là chạy `docker build` lại ở bước Staging rồi lại `docker build` lần nữa ở bước Production—khiến bản chạy thật có thể kéo phải một gói thư viện mới vừa phát hành 10 phút trước và khác hoàn toàn bản đã kiểm thử! Chuẩn vàng là chỉ build **đúng 1 lần**, ghi nhận `sha256` digest bất biến, và chỉ thay đổi **cấu hình môi trường** (biến môi trường/Secret) khi thăng cấp đúng digest đó sang Staging và Production.

3. **Xác thực không khóa tĩnh bằng OIDC (Workload Identity Federation)**:
   Lưu `AWS_ACCESS_KEY_ID` hay JSON Service Account vĩnh viễn trong GitHub Secrets giống như dán chìa khóa nhà dưới thảm chùi chân—lộ một lần là mất kiểm soát cho đến khi ai đó phát hiện. Với **OIDC (OpenID Connect)**, GitHub Actions cấp một mã thông báo JWT ngắn hạn cho đúng một lượt chạy workflow; đám mây xác minh chữ ký JWT đó và đổi lấy **phiên truy cập tạm thời chỉ sống 15 phút**, không có bất kỳ khóa dài hạn nào để bị đánh cắp!

4. **Phân tách môi trường & Kiểm thử khói tự động (Smoke Test + Rollback)**:
   Sau khi triển khai bản mới, pipeline tự động gọi endpoint `/healthz` và một luồng đọc/ghi cốt lõi (smoke test). Nếu không trả về `200 OK` trong thời hạn quy định, CD lập tức giữ nguyên hoặc quay lui về digest ổn định trước đó.

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

### Khung giải thích (nên đọc chậm): Trace từng bước luồng xác thực OIDC không mật khẩu và thăng cấp Digest từ PR lên Production

Hãy theo dõi trọn vẹn một lượt phát hành của nhóm 2 sinh viên từ lúc bấm **Merge Pull Request** trên GitHub cho tới khi người dùng thấy tính năng mới trên Production—không hề lưu một khóa mật khẩu đám mây dài hạn nào!

#### Giai đoạn 1: CI đóng gói & gắn định danh bất biến (phút `0:00 → 2:10`)
1. Commit `e4f9a12` được merge vào nhánh `main`.
2. Job `ci-test` chạy `pytest` / `cargo test` và linter (`ruff` / `clippy`) → **Pass**.
3. Job `ci-build` đóng gói container image, đẩy lên Registry và xuất ra duy nhất một **Digest bất biến**:
   ```text
   IMAGE_DIGEST = ghcr.io/iuh-team/board@sha256:8c3d7b...e910
   ```
   *(Lưu ý: từ giây phút này trở đi, **không bao giờ** chạy `docker build` lại nữa!)*

#### Giai đoạn 2: Đổi danh tính OIDC trong 4 bước (không có `SECRET_KEY` dài hạn!)

| Bước | Bên gửi → Bên nhận | Nội dung trao đổi thực tế | Thời hạn sống |
| :---: | :--- | :--- | :---: |
| 1 | GitHub Actions Runner → GitHub OIDC Provider | Yêu cầu cấp JWT cho workflow hiện tại | — |
| 2 | GitHub OIDC Provider → Runner | Trả về JWT đã ký có claim: `repo="iuh-team/board"`, `ref="refs/heads/main"`, `environment="staging"` | `5 phút` |
| 3 | Runner → Cloud STS (`AssumeRoleWithWebIdentity`) | Gửi JWT để xin quyền triển khai vào Role `deploy-staging-role` | — |
| 4 | Cloud STS → Runner | Kiểm tra chữ ký + khớp chính xác `repo` & `environment` → cấp cặp token tạm thời | **`15 phút` rồi tự hủy!** |

#### Giai đoạn 3: Thăng cấp đúng Digest & Kiểm thử khói (phút `2:15 → 4:00`)
1. Job `deploy-staging` triển khai đúng `sha256:8c3d7b...e910` lên môi trường Staging.
2. Smoke test gọi `GET https://staging.board.iuh/healthz` → nhận `200 OK` trong `45 ms`, kiểm tra truy vấn DB thành công.
3. Thành viên thứ 2 trong nhóm bấm nút **Approve** trên cổng bảo vệ môi trường `production` (GitHub Environment Protection Rule).
4. Job `deploy-prod` xin token OIDC mới cho Role `deploy-prod-role` và đẩy **chính xác `sha256:8c3d7b...e910`** lên Production, rồi chạy smoke test lần cuối.

Nhờ thiết kế này: (1) lộ kho mã nguồn cũng không lộ khóa đám mây, (2) nhánh `feature/*` không thể giả mạo quyền deploy `production`, và (3) mã chạy trên Production giống từng byte với mã vừa vượt qua Staging.

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
