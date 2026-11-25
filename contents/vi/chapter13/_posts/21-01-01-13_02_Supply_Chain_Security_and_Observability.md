---
layout: post
title: 13-02 Bảo mật chuỗi cung ứng, SLSA và quan sát production
chapter: '13'
order: 3
owner: Nguyen Le Linh
lang: vi
categories:
- chapter13
lesson_type: optional
---

Bạn đã viết mã nguồn cực kỳ cẩn thận, bật HTTPS, khóa chặt tường lửa và kiểm tra mọi câu SQL chống injection—nhưng sáng hôm sau hệ thống vẫn bị đánh cắp khóa API. Thủ phạm không hề tấn công vào ứng dụng của bạn, mà lén chèn mã độc vào **một thư viện mã nguồn mở tầng thứ 3** hoặc tráo đổi **container image gắn tag `:latest`** ngay trên kho lưu trữ trước khi cụm Kubernetes kéo về chạy!

Đó là bản chất của **tấn công chuỗi cung ứng phần mềm (Software Supply Chain Attack)**—mối đe dọa hàng đầu giai đoạn 2022–2026 (như vụ SolarWinds, Codecov, XZ Utils). Bài học này trang bị cho bạn hai lớp lá chắn gắn chặt với quy trình phát hành hiện đại: **xác minh mật mã chuỗi cung ứng** (SBOM, chữ ký số, digest bất biến) và **khả năng quan sát gắn liền với mỗi bản phát hành** để tự động quay lui khi chỉ số lỗi tăng vọt.

## Kiến thức cần nhớ: Bốn trụ cột của Bảo mật Chuỗi cung ứng & Phát hành Zero Trust

1. **Danh mục thành phần phần mềm (SBOM — Software Bill of Materials)**:
   Giống như bảng thành phần dinh dưỡng in trên hộp thực phẩm, **SBOM** (chuẩn SPDX hoặc CycloneDX) liệt kê đầy đủ mọi thư viện trực tiếp và bắc cầu cùng phiên bản chính xác nằm trong container image. Khi một lỗ hổng nghiêm trọng như Log4Shell nổ ra, doanh nghiệp có SBOM quét biết chính xác dịch vụ nào bị ảnh hưởng trong 30 giây thay vì mò mẫm cả tuần.

2. **Định danh bất biến bằng Digest (`sha256:...`) & Chữ ký số (Sigstore / Cosign)**:
   Một tag như `myapp:v1.2` hoặc `myapp:latest` là **nhãn có thể bị ghi đè** bất cứ lúc nào. Ngược lại, **digest SHA-256** (`myapp@sha256:9f86d0...`) ràng buộc mật mã với từng byte của image. Kết hợp **Cosign** (ký không cần giữ khóa bí mật dài hạn nhờ OIDC) và chứng thực nguồn gốc **SLSA Provenance**, cụm sản xuất kiểm tra được image này thực sự do đúng pipeline CI của nhóm xây dựng từ đúng commit Git.

3. **Cổng kiểm duyệt tại cụm (Admission Controller — Kyverno / OPA Gatekeeper)**:
   Trước khi Kubernetes cho phép khởi tạo bất kỳ Pod nào, **Admission Controller** chặn ở cửa `kube-apiserver`, xác minh chữ ký Cosign và chính sách bảo mật (cấm chạy `root`, cấm tag `:latest`). Mọi image không rõ nguồn gốc bị từ chối ngay lập tức—đúng tinh thần **Zero Trust**.

4. **Khả năng quan sát gắn với phát hành (Release-Aware Observability)**:
   Mỗi bản phát hành phải gắn nhãn `service.version` / `git.sha` vào mọi metric, log và trace. Nhờ đó, bộ điều khiển triển khai (Argo Rollouts / Flagger) so sánh trực tiếp tỷ lệ lỗi của bản Canary mới so với bản cũ và tự động **rollback** trong vài chục giây mà không chờ người trực tỉnh giấc.

## Mục tiêu học tập

- Phân biệt **SBOM** ("bên trong gói phần mềm chứa những thành phần nào") với **chứng thực nguồn gốc SLSA provenance** ("gói này được biên dịch bằng quy trình nào").
- Kết hợp bước xác minh chữ ký artifact với chiến lược phát hành canary trong cùng một pipeline.
- Gắn dữ liệu trace với phiên bản triển khai để phát hiện sớm bản canary gặp lỗi.

## 1. Vì sao quy trình triển khai cần thêm bước xác minh chuỗi cung ứng

Các sự cố chuỗi cung ứng lớn như SolarWinds (2020) và lỗ hổng `xz` (2024) buộc ngành phần mềm chuyển từ niềm tin mặc định "tệp nhị phân do CI tạo ra là an toàn" sang **xác minh có chứng thực mật mã**. Khung **SLSA v1.0** của OpenSSF (ổn định tháng 10/2023) định nghĩa các cấp độ Build (L1–L3): tạo chứng thực nguồn gốc (provenance), cô lập môi trường build và gia cố bảo mật nền tảng build. Cơ chế GitHub Artifact Attestations (GA năm 2024) và provenance của npm (từ năm 2023) giúp người dùng kiểm chứng trực tiếp các gói công khai.

Song song đó, **SBOM** (theo chuẩn SPDX hoặc CycloneDX) cung cấp danh mục thành phần để rà soát nhanh lỗ hổng CVE. CISA và chuẩn NIST SSDF (SP 800-218), cùng Sắc lệnh hành pháp EO 14028 của Hoa Kỳ, thúc đẩy yêu cầu SBOM và chứng thực phát triển an toàn cho phần mềm khối chính phủ; **Đạo luật Cyber Resilience (CRA)** của EU tiếp tục áp dụng yêu cầu tương tự đối với phần mềm thương mại.

```text
mã nguồn (đã review) → CI cô lập → provenance đã ký + SBOM → xác minh lúc deploy → canary
```

```yaml
steps:
  - verify_cosign: image: ghcr.io/iuh/checkout@sha256:...
  - require_slsa: level: 2
  - require_sbom: format: cyclonedx
  - canary:
      percent: 5
      abort_if: "otel.error_rate > 2x baseline"
```

Chiến lược blue/green hay canary ở phần lý thuyết vẫn chưa đủ an toàn nếu tệp triển khai mới được biên dịch thủ công trên máy cá nhân không kiểm soát.

## 2. Khả năng quan sát gắn liền với quy trình triển khai

OpenTelemetry chính thức đạt trạng thái graduated tại CNCF vào ngày **11/5/2026**. Mẫu thực hành chuẩn khi phát hành gồm ba bước:

1. Mọi dịch vụ gắn thuộc tính `service.version` (mã Git SHA hoặc mã băm digest của ảnh container) vào tất cả trace và metric.
2. Bước phân tích canary đối chiếu trực tiếp tỷ lệ lỗi và độ trễ của *phiên bản mới* với mã băm trước đó.
3. Log ứng dụng được liên kết xuyên suốt qua `trace_id` (chuẩn W3C Trace Context).

Đây chính là bước hiện thực hóa yêu cầu giám sát và ghi log của phần lý thuyết bằng bộ SDK chuẩn hóa, không phụ thuộc nhà cung cấp.

```python
from opentelemetry import trace

tracer = trace.get_tracer("deploy.canary")

def handle(req):
    with tracer.start_as_current_span("checkout") as span:
        span.set_attribute("service.version", "sha-8f3c")
        return route(req)
```

## 3. Áp dụng Zero Trust trong quy trình phát hành

Sự kết hợp giữa định danh khối lượng công việc (workload identity / SPIFFE qua service mesh ở Chương 09), nguyên tắc IAM đặc quyền tối thiểu và **xác thực OIDC ngắn hạn** từ CI tới đám mây (loại bỏ khóa truy cập cố định dài hạn) đã trở thành tiêu chuẩn mặc định từ năm 2024. Kiểm soát chuỗi cung ứng ngăn chặn việc triển khai *mã độc giả mạo*, còn Zero Trust ngăn chặn *khóa bị lộ* giả danh dịch vụ hợp lệ.

<div class="content-box insight-box">
<p><strong>Phạm vi trách nhiệm chia sẻ ngày càng mở rộng.</strong> Nhà cung cấp đám mây đảm nhận mã hóa ổ đĩa và bảo mật hạ tầng, nhưng bạn vẫn chịu trách nhiệm bảo vệ tệp khóa thư viện (lockfile), môi trường build và bước xác minh chứng thực (attestation) trước khi triển khai vào cụm.</p>
</div>

### Giải thích: Trace vụ tráo tag `:latest` bị Admission Controller + Cosign chặn đứng trong 5 bước

Hãy theo dõi từng bước điều gì xảy ra khi kẻ tấn công chiếm được quyền đẩy image lên registry và ghi đè nhãn `registry.iuh.edu/board:latest` bằng một ảnh chứa mã độc đào tiền ảo!

#### Bước 1: So sánh hai ảnh trên Registry

| Thuộc tính | Ảnh hợp pháp do CI build từ commit `a1b2c3d` | Ảnh giả mạo do kẻ tấn công đẩy đè `:latest` |
| :--- | :--- | :--- |
| Tag hiển thị | `board:latest` (bị ghi đè) | `board:latest` |
| **Digest SHA-256 thực tế** | `sha256:4a7e...91c0` | `sha256:ff12...00e8` (khác hoàn toàn!) |
| **Chữ ký Cosign (OIDC từ GitHub Actions)** | Hợp lệ (khớp repo `iuh/board`, nhánh `main`) | **Không có chữ ký** (hoặc ký bằng khóa lạ) |
| **Tệp SBOM đính kèm** | 42 gói sạch, 0 CVE Critical | Thiếu SBOM hoặc chứa binary lạ `xmrig` |

#### Bước 2: Trace 2 kịch bản khi `kubectl apply` chạy vào cụm Production

- **Kịch bản thiếu bảo vệ (dùng `image: board:latest`, không có Admission Controller)**:
  1. `kubelet` thấy `imagePullPolicy: Always` → kéo `board:latest` (`sha256:ff12...00e8`).
  2. Pod khởi chạy thành công với mã độc bên trong, bắt đầu quét mạng nội bộ và đốt 100% CPU cụm.

- **Kịch bản Zero Trust (ghim digest + Kyverno kiểm chữ ký Cosign)**:
  1. Manifest Deployment khai báo cứng `image: registry.iuh.edu/board@sha256:4a7e...91c0` → kẻ tấn công sửa tag `:latest` **hoàn toàn vô tác dụng** vì digest cũ không hề thay đổi!
  2. Ngay cả khi kẻ tấn công sửa được file manifest sang `sha256:ff12...00e8`, yêu cầu `CREATE Pod` gửi tới `kube-apiserver` bị webhook **Kyverno** chặn lại ở bước Admission.
  3. Kyverno gọi xác minh chữ ký Cosign trên nhật ký minh bạch Rekor → **Thất bại (`no matching signatures`)**.
  4. `kube-apiserver` trả mã HTTP `403 Forbidden`, **từ chối tạo Pod** và bắn ngay cảnh báo bảo mật lên kênh vận hành.

Chỉ cần hai thói quen—**ghim digest SHA-256** và **ký kiểm chứng tại cổng Admission**—toàn bộ kịch bản tráo ảnh phổ biến nhất bị vô hiệu hóa hoàn toàn.

## Thách thức

- Đạt mức cô lập SLSA L3 gặp nhiều khó khăn trên các CI runner tự vận hành.
- Cảnh báo nhiễu từ SBOM (hàng nghìn CVE tiềm ẩn) nếu thiếu thông tin đánh giá khả năng khai thác thực tế (VEX / reachability analysis).
- Thiết lập cơ chế lấy mẫu trace sao cho bước phân tích canary vẫn bắt được các lỗi xuất hiện với tần suất thấp.

## Video tham khảo

- **Xem trước:** [SLSA: Industry-Driven Guidelines for Software Supply Chain Security \| OpenSSF Project Spotlight](https://www.youtube.com/watch?v=gdYlSuH5Srs) — OpenSSF (tiếng Anh). Giới thiệu khung SLSA và các cấp độ bảo đảm chuỗi cung ứng trong pipeline.
- [Sigstore: A Wax Seal of Security for the Digital Era \| OpenSSF Project Spotlight](https://www.youtube.com/watch?v=m5eTw4x33kU) — OpenSSF (tiếng Anh). Giải thích cơ chế ký số không cần quản lý khóa dài hạn của Sigstore.
- [OpenTelemetry: OpenTelemetry in Five Minutes \| Project Lightning Talk](https://www.youtube.com/watch?v=l8xiNOCIdLY) — CNCF [Cloud Native Computing Foundation] (tiếng Anh). Tóm tắt OpenTelemetry — nền tảng quan sát tích hợp sẵn trong quy trình phát hành.

## Bài tập

1. Đối với một ảnh container từ đồ án sinh viên, phân biệt những thông tin thuộc về SBOM CycloneDX và những thông tin thuộc về chứng thực nguồn gốc SLSA provenance.
2. Thiết kế quy tắc tự động hủy bản canary dựa trên metric OpenTelemetry, kết hợp cả tỷ lệ lỗi và độ trễ thay vì chỉ kiểm tra mã HTTP 500.
3. Giải thích vì sao việc lưu trực tiếp `AWS_SECRET_ACCESS_KEY` cố định trong biến môi trường GitHub Actions không còn đáp ứng chuẩn quản lý định danh CI hiện đại.

## Tài liệu tham khảo

1. Đặc tả SLSA v1.0: [slsa.dev/spec/v1.0](https://slsa.dev/spec/v1.0/).
2. NIST SP 800-218, *Secure Software Development Framework (SSDF)*.
3. Tài nguyên SBOM của CISA; Sắc lệnh EO 14028 của Hoa Kỳ.
4. CNCF OpenTelemetry graduation: [cncf.io/projects/opentelemetry](https://www.cncf.io/projects/opentelemetry/).
5. Tài liệu Sigstore và GitHub Artifact Attestations (GA 2024).
