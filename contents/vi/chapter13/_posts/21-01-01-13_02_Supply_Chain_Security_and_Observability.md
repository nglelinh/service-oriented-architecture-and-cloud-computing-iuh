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

Phần bắt buộc của chương đã trình bày IAM, mã hóa, chiến lược triển khai blue/green và canary, CI/CD và giám sát. Bài tùy chọn này bổ sung lớp **bảo mật chuỗi cung ứng phần mềm** giai đoạn 2022–2026 (SLSA, SBOM, Sigstore), đồng thời xem **OpenTelemetry** như một hợp đồng quan sát bắt buộc trong quá trình phát hành chứ không phải agent gắn thêm sau khi hệ thống đã đi vào hoạt động.

> Lý thuyết bắt buộc hiện ở bản tiếng Anh. Đây là bài ứng dụng tùy chọn bằng tiếng Việt.

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
