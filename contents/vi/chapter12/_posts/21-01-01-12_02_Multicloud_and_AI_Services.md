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

Trong các buổi thuyết trình chiến lược, khẩu hiệu *"Đa đám mây (Multi-cloud) để không bị khóa chặt vào một nhà cung cấp!"* nghe lúc nào cũng hấp dẫn. Nhưng khi bước vào phòng máy thực tế, mọi kỹ sư hạ tầng đều sớm đối mặt với hai bức tường vật lý khắc nghiệt: **trọng lực dữ liệu (data gravity)**—chi phí truyền dữ liệu ra khỏi đám mây (*egress*) đắt gấp nhiều lần lưu trữ—và độ phức tạp tăng theo cấp số nhân khi phải duy trì hai hệ thống IAM, mạng VPC và giám sát hoàn toàn khác nhau.

Bài học này bóc tách thực tế **đa đám mây** giai đoạn 2022–2026: khi nào doanh nghiệp thực sự cần chạy trên cả AWS, Azure và GCP (ví dụ thuê GPU/LLM chuyên biệt hoặc tuân thủ chủ quyền dữ liệu quốc gia), và làm sao thiết kế biên giới tích hợp thông minh mà không tự đốt ngân sách.

## Kiến thức cần nhớ: Bốn trụ cột quyết định kiến trúc Đa đám mây & Dịch vụ AI

1. **Trọng lực dữ liệu (Data Gravity) & Kinh tế học Egress**:
   Trên các đám mây lớn, đẩy dữ liệu **vào (ingress)** thường miễn phí (`$0/GB`), lưu trữ khá rẻ (`~$0,023/GB/tháng`), nhưng kéo dữ liệu **ra ngoài Internet hoặc sang đám mây khác (egress)** có thể tốn `$0,08–$0,09/GB`. Vì thế, kéo hàng chục TB dữ liệu qua lại giữa hai đám mây mỗi tuần là một cái bẫy tài chính nghiêm trọng.

2. **Tầng trừu tượng di động vs. Dịch vụ quản lý đặc thù (Managed Services)**:
   Các công nghệ chuẩn mở (Kubernetes, PostgreSQL, Apache Iceberg, OpenTofu) giúp bạn di chuyển khối lượng công việc tương đối dễ dàng giữa các hãng. Ngược lại, các dịch vụ quản lý sâu (AWS Bedrock, Azure OpenAI, GCP Vertex AI, BigQuery) mang lại tốc độ phát triển vượt trội nhưng tạo ra mức độ phụ thuộc (*lock-in*) nhất định—kiến trúc sư giỏi không cố xóa sạch lock-in mà chọn **khóa có chủ đích** ở nơi tạo ra giá trị lớn nhất.

3. **Dịch vụ AI quản lý (Managed AI & Inference Endpoints)**:
   Thay vì tự thuê cụm GPU H100 và gánh chi phí nhàn rỗi 24/7, doanh nghiệp thường gọi API mô hình nền tảng (Foundation Models) qua cổng quản lý có tích hợp sẵn kiểm soát hạn mức, lọc nội dung, mã hóa riêng tư và không dùng dữ liệu khách hàng để huấn luyện lại.

4. **Chủ quyền dữ liệu (Data Sovereignty) & Vị trí lưu trữ (Data Residency)**:
   Luật pháp tại EU (GDPR), Việt Nam (Nghị định 53/2022/NĐ-CP) hay ngành tài chính–y tế thường yêu cầu dữ liệu cá nhân của công dân phải được lưu trữ và xử lý trong phạm vi địa lý nhất định—buộc kiến trúc sư phải chọn đúng **Region** vật lý hỗ trợ cả lưu trữ lẫn điểm cuối suy luận AI.

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

### Giải thích: Tính tay cái bẫy Egress khi chạy Multi-Cloud (dữ liệu ở AWS, huấn luyện AI ở GCP)

Giả sử công ty bạn lưu **5 TB** (`5.120 GB`) kho ảnh y tế trên **AWS S3 (`ap-southeast-1`)**, nhưng đội AI muốn dùng cụm GPU/TPU bên **GCP** để huấn luyện lại mô hình **mỗi tuần 1 lần** (4 lần/tháng) bằng cách tải toàn bộ 5 TB qua Internet công cộng. Hãy tính hóa đơn truyền tải thuần túy trong 1 tháng:

| Phương án kiến trúc | Lưu lượng Egress qua Internet / tháng | Đơn giá Egress ước lượng | Chi phí Egress / tháng | Thời gian tải (đường truyền `1 Gbps` $\approx 125\text{ MB/s}$) |
| :--- | :---: | :---: | :---: | :---: |
| **A. Kéo 5 TB từ AWS sang GCP mỗi tuần** | $4 \times 5.120 = \mathbf{20.480\text{ GB}}$ | `$0,09 / GB` | $20.480 \times 0{,}09 = \mathbf{\$1.843{,}20}$ | $\frac{5.120\text{ GB}}{0{,}125\text{ GB/s}} \approx \mathbf{11{,}4\text{ giờ/lần}}$ |
| **B. Đặt Compute cùng Region AWS (hoặc đồng bộ tăng dần 1 lần + delta `200 GB`/tuần)** | Chỉ đẩy delta: $4 \times 200 = \mathbf{800\text{ GB}}$ (hoặc `$0` nếu cùng Region) | `$0,09 / GB` (hoặc `$0`) | **$\$72{,}00$** (hoặc **$\$0$** cùng Region) | **27 phút** (hoặc **vài phút** nội bộ `25 Gbps`) |

**Bài học rút ra từ con số `$1.843` vs `$0`**: Multi-cloud thực tế hiếm khi chạy *một ứng dụng* chia đôi lưu trữ và tính toán qua hai đám mây. Thay vào đó, doanh nghiệp áp dụng **Best-of-Breed theo miền độc lập** (ví dụ: ứng dụng chính chạy gọn trên AWS, còn kho phân tích + Vertex AI đồng bộ dữ liệu tăng dần một chiều sang GCP) để tránh trả thuế trọng lực dữ liệu lặp lại mỗi tuần.

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
