---
layout: post
title: 12-02 Kiến trúc đa đám mây và dịch vụ AI quản lý (2022–2026)
chapter: '12'
order: 3
owner: Nguyen Le Linh
lang: vi
categories:
- chapter12
lesson_type: optional
---

Bài bắt buộc ánh xạ EC2/VM, S3/Blob và free tier giữa AWS, Azure và GCP. Ghi chú tùy chọn là cách các catalog đó *được dùng* trong 2022–2026: **đa đám mây vì ràng buộc**, FinOps xuyên hóa đơn, và **AI quản lý** (Bedrock, Azure OpenAI, Vertex AI) như hàng mới trong bảng so sánh.

> Lý thuyết bắt buộc đang ở bản tiếng Anh. Đây là phần ứng dụng tùy chọn bằng tiếng Việt.

## Mục tiêu học tập

- Tách *giao diện di động* (API S3, Kubernetes, OTel, Iceberg) khỏi dịch vụ *dính* (API AI độc quyền).
- So sánh nền tảng LLM quản lý trên cùng trục đã dùng cho tính toán/lưu trữ.
- Thiết kế adapter mỏng để phần còn lại của SOA không import SDK một nhà cung cấp khắp nơi.

## 1. Đa đám mây trong dữ liệu, không trên slide

Flexera 2025: hybrid là bình thường (**70%**), khoảng **2,4** đám mây công cộng, chi tiêu là nỗi đau số 1 (**84%**), nhóm FinOps **59%**. AWS và Azure sát nút; GCP mạnh thứ ba, thường vì dữ liệu/ML. Đó là bối cảnh bảng “Big 3” bắt buộc đang nằm trong.

**Đa đám mây tốt** (di động chỗ đáng trả):

- Kubernetes + Gateway API + OTel trên mọi đám mây.
- Định dạng bảng mở trên object storage.
- Liên kết định danh (OIDC) thay khóa sống lâu.

**Đa đám mây đắt** (nhân đôi mọi thứ):

- Ba kho, ba service mesh, ba hệ CI “cho bền.”
- gRPC nói nhiều xuyên đám mây (egress + RTT).

```python
class CompletionClient:
    def complete(self, prompt: str, max_tokens: int) -> str:
        raise NotImplementedError

class BedrockAdapter(CompletionClient):
    def complete(self, prompt: str, max_tokens: int) -> str:
        return _bedrock_invoke(prompt, max_tokens)

class VertexAdapter(CompletionClient):
    def complete(self, prompt: str, max_tokens: int) -> str:
        return _vertex_invoke(prompt, max_tokens)
```

Ứng dụng phụ thuộc `CompletionClient`. FinOps và cư trú quyết định adapter nào được inject.

## 2. AI quản lý như một hàng dịch vụ đám mây

| Trục | AWS | Azure | GCP |
| --- | --- | --- | --- |
| Mô hình nền tảng quản lý | Bedrock (đa mô hình) | Azure OpenAI / AI Foundry | Vertex AI / Gemini |
| GPU IaaS | EC2 / Trainium, Inferentia | N-series / ND | A3/A4, TPU |
| Trọng lực mặt phẳng dữ liệu | S3 + IAM | ADLS + Entra | GCS + IAM |
| Điểm dính điển hình | Account + đồ thị IAM | Entra ID + M365 | BigQuery + phân tích |

Flexera 2025 ghi khoảng **72%** người trả lời dùng AI tạo sinh và áp lực AI lên hóa đơn đám mây tăng. Coi API mô hình quản lý như **PaaS kiểu SaaS**: bạn không vá GPU, nhưng *có* sở hữu injection prompt, log và cư trú dữ liệu.

<div class="content-box warning-box">
<p><strong>Egress là thuế đa đám mây thầm lặng.</strong> Huấn luyện một đám mây và phục vụ đám mây khác có thể đắt hơn giờ GPU. Vẽ byte trước khi vẽ hộp.</p>
</div>

## 3. Chủ quyền và “vùng nào” như kiến trúc

Với hệ thống kiểu IUH, câu hỏi từng tùy chọn năm 2018 nay là đầu vào thiết kế:

- Prompt sinh viên có được rời khỏi nước?
- Đối tác có yêu cầu chào hàng chủ quyền EU?
- Có cần khóa do khách quản lý trên kho embedding?

Đôi khi câu trả lời là **một** hyperscaler + on-prem hoặc đám mây quốc gia cho lát bị điều tiết — không phải ba đám mây công cộng.

## Thách thức

- Hóa đơn khó so (RUM, token, giây-GPU).
- Mô hình IAM khác nhau (khóa thật).
- Vận tốc tính năng AI: adapter hôm nay là model id bị deprecated ngày mai.

## Bài tập

1. Mở rộng bảng ánh xạ dịch vụ bắt buộc thêm *một* hàng AI và *một* hàng quan sát (collector OTel).
2. Ước lượng egress cho 5 TB embedding copy hàng tháng giữa hai vùng (dùng trang giá công khai).
3. Viết ADR: “Chúng tôi chấp nhận khóa Azure OpenAI 18 tháng vì …” (hoặc ngược lại).

## Tài liệu tham khảo

1. Flexera *2025 State of the Cloud Report* (báo chí và blog xu hướng).
2. Tài liệu sản phẩm AWS Bedrock, Azure OpenAI, Google Vertex AI (so hạn mức, chính sách dùng dữ liệu, vùng).
3. Khảo sát CNCF 2024 — Kubernetes như lớp tính toán di động.
4. Bảng ánh xạ nhà cung cấp bài bắt buộc — đường cơ sở ghi chú này mở rộng.
