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

Phần bắt buộc của chương đối chiếu các dịch vụ nền tảng (EC2/VM, S3/Blob và hạn mức free tier) giữa AWS, Azure và GCP. Bài tùy chọn này phân tích cách các danh mục dịch vụ đó *được vận dụng thực tế* giai đoạn 2022–2026: **đa đám mây theo ràng buộc nghiệp vụ**, quản trị chi phí FinOps xuyên hóa đơn, và nhóm **dịch vụ AI được quản lý** (Amazon Bedrock, Azure OpenAI, Google Vertex AI) như một hạng mục mới trong bảng so sánh.

> Lý thuyết bắt buộc hiện ở bản tiếng Anh. Đây là bài ứng dụng tùy chọn bằng tiếng Việt.

## Mục tiêu học tập

- Phân tách *giao diện di động* (API tương thích S3, Kubernetes, OpenTelemetry, Iceberg) khỏi các dịch vụ *gây ràng buộc chặt* (API AI độc quyền).
- So sánh các nền tảng LLM được quản lý dựa trên cùng hệ tiêu chí áp dụng cho tính toán và lưu trữ.
- Thiết kế lớp chuyển đổi (adapter) mỏng để phần lớn hệ thống SOA không bị phụ thuộc trực tiếp vào SDK của một nhà cung cấp duy nhất.

## 1. Đa đám mây nhìn từ số liệu thực tế

Theo báo cáo Flexera 2025, mô hình lai đã trở thành phổ biến (**70%** tổ chức), mỗi đơn vị dùng trung bình **2,4** nhà cung cấp đám mây công cộng; quản lý chi tiêu là thách thức lớn nhất (**84%**) và **59%** tổ chức đã có đội ngũ FinOps. AWS và Azure duy trì quy mô bám sát nhau, trong khi GCP giữ vị trí thứ ba với thế mạnh rõ ở mảng dữ liệu và học máy. Đây là bối cảnh thực tế cho bảng đối chiếu ba nhà cung cấp lớn trong phần lý thuyết.

**Đa đám mây hợp lý** (đầu tư tính di động ở những tầng mang lại giá trị):

- Dùng chuẩn chung như Kubernetes, Gateway API và OpenTelemetry trên mọi đám mây.
- Lưu dữ liệu bằng định dạng bảng mở trên object storage.
- Áp dụng liên kết định danh (OIDC) thay vì phát hành khóa truy cập cố định dài hạn.

**Đa đám mây tốn kém** (nhân đôi hạ tầng một cách máy móc):

- Duy trì đồng thời ba kho dữ liệu, ba service mesh và ba hệ thống CI chỉ để "dự phòng".
- Gọi gRPC tần suất cao xuyên qua hai nhà cung cấp đám mây khác nhau (tốn phí egress và tăng độ trễ RTT).

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

Nhờ lớp giao diện `CompletionClient`, mã nghiệp vụ tách biệt khỏi nhà cung cấp; bài toán FinOps và yêu cầu lưu trữ dữ liệu theo vùng sẽ quyết định adapter nào được nạp khi chạy.

## 2. Dịch vụ AI quản lý trong bảng so sánh ba nhà cung cấp

| Tiêu chí | AWS | Azure | GCP |
| --- | --- | --- | --- |
| Nền tảng mô hình quản lý | Bedrock (đa mô hình) | Azure OpenAI / AI Foundry | Vertex AI / Gemini |
| Hạ tầng GPU (IaaS) | EC2 / Trainium, Inferentia | Dòng N-series / ND | A3/A4, TPU |
| Trọng lực dữ liệu (data gravity) | S3 + IAM | ADLS + Entra | GCS + IAM |
| Yếu tố gây ràng buộc phổ biến | Tổ chức tài khoản + đồ thị IAM | Entra ID + Microsoft 365 | BigQuery + hệ phân tích |

Báo cáo Flexera 2025 ghi nhận khoảng **72%** tổ chức tham gia đã sử dụng AI tạo sinh, kéo theo áp lực chi phí mới lên hóa đơn đám mây. Có thể nhìn API mô hình quản lý như một dịch vụ **PaaS thiên về SaaS**: bạn không phải tự bảo trì GPU, nhưng vẫn chịu trách nhiệm kiểm soát tấn công chèn lệnh (prompt injection), nhật ký sử dụng và quy định lưu trữ dữ liệu theo vùng.

<div class="content-box warning-box">
<p><strong>Phí truyền dữ liệu ra ngoài (egress) là chi phí ẩn lớn của đa đám mây.</strong> Huấn luyện mô hình trên một nhà cung cấp và phục vụ suy luận trên nhà cung cấp khác có thể phát sinh chi phí truyền tải vượt cả tiền thuê GPU. Hãy tính toán luồng dữ liệu trước khi vẽ sơ đồ kiến trúc.</p>
</div>

## 3. Chủ quyền dữ liệu và lựa chọn vùng triển khai

Với các hệ thống quy mô trường đại học như IUH, những câu hỏi từng ít được chú ý trước năm 2018 nay là yêu cầu đầu vào bắt buộc khi thiết kế:

- Dữ liệu prompt của sinh viên có được phép truyền ra ngoài lãnh thổ quốc gia không?
- Đối tác có yêu cầu triển khai trên hạ tầng đáp ứng chủ quyền dữ liệu EU không?
- Kho lưu trữ embedding có cần mã hóa bằng khóa do tổ chức tự quản lý (customer-managed key) không?

Trong nhiều trường hợp, lời giải phù hợp là kết hợp **một** nhà cung cấp đám mây lớn với hạ tầng tại chỗ hoặc đám mây trong nước cho phần dữ liệu chịu điều tiết, thay vì dàn trải trên ba đám mây công cộng cùng lúc.

## Thách thức

- Khó so sánh trực tiếp cấu trúc hóa đơn giữa các đơn vị tính khác nhau (lượt gọi, số token, giây GPU).
- Mô hình phân quyền IAM giữa các nhà cung cấp khác nhau sâu sắc — đây thường mới là yếu tố gây phụ thuộc mạnh nhất.
- Vòng đời mô hình AI thay đổi nhanh: mã định danh mô hình đang dùng hôm nay có thể sớm bị ngừng hỗ trợ (deprecated).

## Video tham khảo

- **Xem trước:** [What is Multicloud? How Do You Manage It?](https://www.youtube.com/watch?v=AjtdZ3gFRjU) — IBM Technology (tiếng Anh). Phân tích đa đám mây dưới góc độ vận hành thực tế.
- [Introducing Amazon Bedrock \| Amazon Web Services](https://www.youtube.com/watch?v=_vdK5PgcNvc) — Amazon Web Services (tiếng Anh). Giới thiệu Amazon Bedrock — dịch vụ AI tạo sinh được quản lý của AWS.
- [What is Vertex AI](https://www.youtube.com/watch?v=Rl9Gz1Yf6wM) — Google Cloud (tiếng Anh). Giới thiệu nền tảng Vertex AI của Google trong cùng bảng so sánh.

## Bài tập

1. Bổ sung vào bảng ánh xạ dịch vụ của phần bắt buộc thêm *một* hàng cho dịch vụ AI quản lý và *một* hàng cho tầng quan sát (OpenTelemetry Collector).
2. Dựa trên bảng giá công khai, ước lượng chi phí egress khi sao chép 5 TB dữ liệu embedding mỗi tháng giữa hai vùng địa lý khác nhau.
3. Viết một bản ghi quyết định kiến trúc (ADR) ngắn lập luận cho lựa chọn: "Chúng tôi chấp nhận phụ thuộc vào Azure OpenAI trong 18 tháng vì…" (hoặc lập luận ngược lại).

## Tài liệu tham khảo

1. Flexera *2025 State of the Cloud Report* (thông cáo báo chí và bài tổng hợp xu hướng).
2. Tài liệu sản phẩm AWS Bedrock, Azure OpenAI và Google Vertex AI (đối chiếu hạn mức, chính sách xử lý dữ liệu và vùng khả dụng).
3. Khảo sát CNCF 2024 — vai trò của Kubernetes như lớp tính toán di động.
4. Bảng ánh xạ nhà cung cấp trong phần lý thuyết bắt buộc — cơ sở mà bài đọc này mở rộng.
