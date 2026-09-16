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

Bài bắt buộc phủ IAM, mã hóa, blue/green/canary, CI/CD và giám sát. Ghi chú tùy chọn thêm lớp **chuỗi cung ứng phần mềm** 2022–2026 (SLSA, SBOM, Sigstore) và coi **OpenTelemetry** là hợp đồng quan sát production — không phải agent nhà cung cấp gắn sau khi go-live.

> Lý thuyết bắt buộc đang ở bản tiếng Anh. Đây là phần ứng dụng tùy chọn bằng tiếng Việt.

## Mục tiêu học tập

- Phân biệt SBOM (“bên trong có gì”) với provenance SLSA (“xây thế nào”).
- Đặt canary + artifact đã ký trên cùng pipeline.
- Nối trace với phiên bản triển khai để canary xấu lộ rõ.

## 1. Vì sao bảo mật triển khai thêm giai đoạn pipeline thứ tư

SolarWinds (2020) và các sự cố hệ sinh thái sau đó (kể cả làn sóng xz 2024) chuyển lòng tin từ “binary đến từ CI” sang **chứng thực**. OpenSSF **SLSA v1.0** (spec ổn định đầu, 10/2023) định nghĩa track Build (L1–L3): provenance, build cô lập, và nền tảng cứng. Artifact attestation GitHub (GA 2024) và provenance npm (từ 2023) làm điều này thấy được trên gói công khai.

**SBOM** (SPDX hoặc CycloneDX) trả lời câu hỏi tồn kho/CVE. CISA và NIST SSDF (SP 800-218), thúc bởi EO 14028, đẩy SBOM và chứng thực phát triển an toàn cho phần mềm chính phủ. **Đạo luật Cyber Resilience** của EU giữ cùng áp lực lên phần mềm thương mại đến cuối những năm 2020.

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

Blue/green của bài bắt buộc cần nhưng chưa đủ nếu artifact *xanh* được xây trên laptop.

## 2. Quan sát là một phần của deploy

OpenTelemetry tốt nghiệp CNCF ngày **11/5/2026**. Mẫu ứng dụng:

1. Mọi dịch vụ phát trace/metric với `service.version` = Git SHA hoặc digest ảnh.
2. Phân tích canary so tỷ lệ lỗi và độ trễ *phiên bản đó* với digest trước.
3. Log gắn qua `trace_id` (W3C Trace Context).

Đây là kết quả “giám sát và ghi log” bắt buộc với SDK di động.

```python
from opentelemetry import trace

tracer = trace.get_tracer("deploy.canary")

def handle(req):
    with tracer.start_as_current_span("checkout") as span:
        span.set_attribute("service.version", "sha-8f3c")
        return route(req)
```

## 3. Zero trust, vẫn trong một đoạn

Định danh (workload identity / SPIFFE qua mesh, Chương 09), IAM đặc quyền tối thiểu (bắt buộc), và **OIDC sống ngắn** tới đám mây (không khóa truy cập sống lâu trong CI) là mặc định 2024. Kiểm soát chuỗi cung ứng chặn deploy *độc hại đã ký*; zero trust chặn *khóa bị đánh cắp* trông như dịch vụ bình thường.

<div class="content-box insight-box">
<p><strong>Trách nhiệm chia sẻ mở rộng.</strong> Đám mây mã hóa đĩa. Bạn vẫn sở hữu lockfile, builder, và bộ xác minh attestation trong cụm.</p>
</div>

## Thách thức

- Cô lập SLSA L3 khó trên runner tự host.
- Nhiễu SBOM (hàng nghìn CVE) không có VEX / reachability.
- Sampling trace sao cho phân tích canary vẫn thấy lỗi hiếm.

## Bài tập

1. Với ảnh dự án sinh viên, liệt kê thứ gì vào SBOM CycloneDX vs. tuyên bố provenance SLSA.
2. Thiết kế quy tắc hủy canary dùng metric OTel, không chỉ HTTP 500 (gồm độ trễ).
3. Giải thích vì sao lưu `AWS_SECRET_ACCESS_KEY` trong biến GitHub Actions trượt câu chuyện định danh CI 2024.

## Tài liệu tham khảo

1. Đặc tả SLSA v1.0: [slsa.dev/spec/v1.0](https://slsa.dev/spec/v1.0/).
2. NIST SP 800-218, *SSDF*.
3. Tài nguyên SBOM của CISA; EO 14028 Hoa Kỳ (bối cảnh chứng thực liên bang).
4. CNCF OpenTelemetry graduation: [cncf.io/projects/opentelemetry](https://www.cncf.io/projects/opentelemetry/).
5. Tài liệu Sigstore / artifact attestation GitHub (GA 2024).
