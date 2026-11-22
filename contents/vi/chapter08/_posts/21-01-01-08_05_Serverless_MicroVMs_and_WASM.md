---
layout: post
title: 08-05 Serverless, microVM và WebAssembly (2022–2026)
chapter: '08'
order: 6
owner: Nguyen Le Linh
lang: vi
categories:
- chapter08
lesson_type: optional
---

Các bài bắt buộc đã trình bày về hypervisor, namespace/cgroup trong Linux, Docker và AWS Lambda. Bài tùy chọn này theo dõi các bước tiến xung quanh những nền tảng đó giai đoạn 2022–2026: **snapshot microVM Firecracker** (Lambda SnapStart), hỗ trợ **hàm đóng gói bằng ảnh container**, và **WebAssembly (WASM)** như một lớp cô lập gọn nhẹ hơn — đồng thời lý giải vì sao khảo sát CNCF ghi nhận xu hướng áp dụng serverless có sự *phân hóa* thay vì tăng trưởng đồng đều.

> Lý thuyết bắt buộc hiện ở bản tiếng Anh. Đây là bài ứng dụng tùy chọn bằng tiếng Việt.

## Mục tiêu học tập

- Giải thích cơ chế SnapStart như bản chụp trạng thái (snapshot) của một microVM Firecracker đã khởi tạo sẵn.
- Đặt WebAssembly/WASI cạnh container Linux để so sánh kích thước môi trường cô lập và giao diện lời gọi hệ thống.
- Lựa chọn hợp lý giữa FaaS, Knative và Kubernetes Deployment chạy thường trực cho từng ranh giới dịch vụ SOA.

## 1. SnapStart: giảm độ trễ khởi động nguội mà không phải viết lại ứng dụng

AWS ra mắt Lambda SnapStart cho **Java** (28/11/2022), mở rộng sang **Python và .NET** (phát hành rộng rãi năm 2024), và tiếp tục hỗ trợ hàm chạy từ **ảnh container** (AWS What’s New, tháng 7/2026). Theo tài liệu AWS, khi phiên bản hàm được xuất bản, Lambda khởi tạo môi trường thực thi, tạo **bản chụp (snapshot) đã mã hóa của bộ nhớ và đĩa trên microVM Firecracker**, rồi khôi phục nhanh từ bản chụp đó mỗi khi có lời gọi hoặc mở rộng tải.

```python
import uuid
import socket

def handler(event, context):
    node_id = str(uuid.uuid4())  # sinh mỗi invoke, không phải mỗi init
    return {"id": node_id, "host": socket.gethostname()}
```

Về bản chất, đây vẫn là **ảo hóa phần cứng** (microVM) chứ không chỉ là cô lập bằng namespace như container thuần. Vấn đề lập trình viên cần đặc biệt lưu ý là *thời điểm* sinh giá trị duy nhất, khóa mật mã hoặc khởi tạo kết nối mạng để tránh trùng lặp trạng thái khi khôi phục từ cùng một snapshot.

## 2. WebAssembly (WASM): cô lập nhẹ không cần nhân hệ điều hành khách

Các nền tảng như Fermyon **Spin**, Wasmtime và chuẩn WASI hướng tới thời gian khởi động ở mức mili-giây cùng cơ chế sandbox phân quyền theo năng lực (capability-based). So với Docker, ứng dụng chỉ cần đóng gói module biên dịch thay vì cả tầng userspace của hệ điều hành. Tuy nhiên, hệ sinh thái I/O của WASI, công cụ gỡ lỗi và mức độ phổ biến kỹ năng trong đội ngũ hiện chưa đồng đều như container Linux. Do đó, nên xem WASM là **lớp cô lập thứ ba** (bên cạnh VM và container), không phải sự thay thế tuyệt đối cho Docker.

Khảo sát CNCF 2024 (công bố năm 2025) cho thấy mức độ sử dụng serverless có sự **phân hóa**: một số tổ chức tiếp tục mở rộng, trong khi số khác thu hẹp vì bài toán chi phí và độ phức tạp vận hành. Điều này cũng phản ánh thực tế quen thuộc khi kết hợp Lambda với NAT Gateway, VPC và cold start đôi khi lại phức tạp và chậm hơn một Deployment nhỏ chạy ổn định.

## 3. Bảng đối chiếu mẫu ứng dụng

| Mẫu bài toán | Cơ chế phù hợp | Ví dụ thực tế |
| --- | --- | --- |
| API có lưu lượng đột biến | FaaS + SnapStart hoặc provisioned concurrency | Tiếp nhận webhook thanh toán |
| Dịch vụ HTTP cần `scale-to-zero` | Knative / Cloud Run | Công cụ quản trị nội bộ |
| Môi trường cô lập nhẹ không sidecar | WASM tại biên (edge) hoặc Spin | Xác thực tại tầng CDN |
| Tải GPU chạy liên tục | Không ưu tiên FaaS | Phục vụ vLLM trên Kubernetes |

<div class="content-box insight-box">
<p><strong>Góc nhìn SOA.</strong> Serverless là một lựa chọn về <em>cách đóng gói và tính phí</em>. Các hợp đồng dịch vụ cốt lõi như timeout, tính lũy đẳng (idempotency) và xác thực vẫn phải được thiết kế nghiêm ngặt tương tự microservice đóng gói trong container.</p>
</div>

## 4. Kiến trúc kết hợp: hướng sự kiện ở biên và gRPC ở lõi

Một mẫu kiến trúc phổ biến giai đoạn 2024–2026 thường kết hợp ba lớp:

1. API Gateway và hàm FaaS xử lý lưu lượng HTTP bắc–nam *biến động mạnh*.
2. Các worker gRPC chạy thường trực cho giao tiếp đông–tây *tần suất cao* (kết hợp service mesh hoặc ambient mode).
3. Lưu trữ đối tượng (object storage) và hàng đợi cho các dữ liệu kích thước lớn thay vì nhồi trực tiếp vào nội dung sự kiện.

Không nên đặt trực tiếp tác vụ sinh nội dung LLM kéo dài 30 giây bên trong giới hạn timeout của API Gateway mà không tách thành cơ chế xử lý bất đồng bộ kèm mã tác vụ (`job_id`).

## Thách thức

- Lỗi mất tính duy nhất khi khôi phục snapshot trong SnapStart (chứng chỉ hết hạn, bộ sinh số ngẫu nhiên RNG, hoặc kết nối cũ trong pool).
- Rủi ro phụ thuộc nền tảng trong hệ sinh thái WASM do giao diện host API giữa các nhà cung cấp còn khác nhau.
- Bài toán chi phí: tính phí theo mili-giây có thể đắt hơn một Deployment 1 replica chạy liên tục khi lưu lượng QPS ổn định.

## Video tham khảo

- **Xem trước:** [How AWS’s Firecracker virtual machines work \| Amazon Science](https://www.youtube.com/watch?v=BIRv2FnHJAg) — Amazon Science (tiếng Anh). Trình bày kiến trúc microVM Firecracker đứng sau AWS Lambda.
- [Introducing AWS Lambda SnapStart \| Amazon Web Services](https://www.youtube.com/watch?v=AgxvrZLI1mc) — Amazon Web Services (tiếng Anh). Giới thiệu cơ chế tạo và khôi phục snapshot của Lambda SnapStart.
- [Introduction to WebAssembly and WASI](https://www.youtube.com/watch?v=Of07IsplHPY) — Root Access (tiếng Anh). Giải thích mô hình cô lập của WebAssembly và WASI.

## Bài tập

1. Liệt kê ba loại đối tượng được khởi tạo lúc init cần phải làm mới lại sau khi khôi phục từ snapshot SnapStart.
2. So sánh (có thể định tính) giữa chạy AWS Lambda cấu hình 128 MB và chạy một container 1 replica thường trực cho một API tốn CPU ở mức ổn định 20 yêu cầu/giây (RPS).
3. Phác thảo kịch bản nên dùng WASM tại tầng biên (edge) so với dùng microVM Firecracker trong vùng đám mây (region).

## Tài liệu tham khảo

1. AWS, “Improving startup performance with Lambda SnapStart”.
2. AWS News Blog, SnapStart cho Python và .NET GA.
3. AWS What’s New, SnapStart cho hàm ảnh container (7/2026).
4. Fermyon Spin: [fermyon.com/spin](https://www.fermyon.com/spin).
5. CNCF *Cloud Native 2024 Annual Survey* (xu hướng phân hóa serverless).
6. Firecracker: [firecracker-microvm.github.io](https://firecracker-microvm.github.io/).
