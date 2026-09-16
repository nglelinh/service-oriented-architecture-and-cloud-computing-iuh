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

Các bài bắt buộc đã phủ hypervisor, namespace/cgroup Linux, Docker và AWS Lambda. Ghi chú tùy chọn theo dõi điều đổi *quanh* các nguyên thủy đó trong 2022–2026: **snapshot Firecracker** (Lambda SnapStart), **hàm ảnh container**, và **WebAssembly** như cược cô lập/tốc độ mịn hơn — cùng lý do khảo sát CNCF cho thấy adoption serverless *tách đôi*, không tăng đều.

> Lý thuyết bắt buộc đang ở bản tiếng Anh. Đây là phần ứng dụng tùy chọn bằng tiếng Việt.

## Mục tiêu học tập

- Giải thích SnapStart như snapshot của microVM Firecracker đã khởi tạo, không phải mô hình lập trình mới.
- Đặt WASM/WASI cạnh container: isolate nhỏ hơn, câu chuyện syscall khác.
- Quyết định khi nào FaaS, Knative, hoặc Deployment luôn bật là biên SOA đúng.

## 1. SnapStart: đánh cold start mà không viết lại ứng dụng

AWS giới thiệu Lambda SnapStart cho **Java** (28/11/2022), rồi **Python và .NET** (GA 2024), và sau đó hàm **ảnh container** (AWS What’s New, 7/2026). Cơ chế (tài liệu AWS): khi publish, Lambda khởi tạo môi trường, lấy **snapshot microVM Firecracker** đã mã hóa của bộ nhớ và đĩa, rồi resume từ snapshot khi invoke/scale-out.

```python
import uuid
import socket

def handler(event, context):
    node_id = str(uuid.uuid4())  # sinh mỗi invoke, không phải mỗi init
    return {"id": node_id, "host": socket.gethostname()}
```

Đây vẫn là **ảo hóa** (bài bắt buộc): microVM, không phải container chỉ-namespace. Bài ứng dụng là *khi nào* tính duy nhất và kết nối được tạo.

## 2. WASM: cô lập không cần kernel khách

Dự án như Fermyon **Spin**, Wasmtime và WASI nhắm khởi động mili-giây và sandbox theo capability. Lời chào so với Docker: ship module, không userspace OS. Cái bắt: I/O WASI, gỡ lỗi và kỹ năng nhóm chưa đều bằng container Linux. Coi WASM là **isolate thứ ba** (VM / container / wasm), không phải “Docker đã chết.”

Khảo sát CNCF 2024 (công bố 2025) ghi dùng serverless **tách**: một số tổ chức mở rộng, số khác rút vì chi phí và phức tạp. Khớp lab sinh viên nơi Lambda + NAT + VPC + cold start vận hành chậm hơn một Deployment nhỏ.

## 3. Mẫu ứng dụng

| Mẫu | Cơ chế | Ví dụ những năm 2020 |
| --- | --- | --- |
| API đột biến | FaaS + SnapStart / provisioned concurrency | Webhook thanh toán |
| HTTP scale-to-zero | Knative / Cloud Run | Công cụ nội bộ |
| Isolate không sidecar | WASM ở edge hoặc Spin | Auth tại CDN |
| GPU luôn bật | Không FaaS-first | vLLM trên Kubernetes |

<div class="content-box insight-box">
<p><strong>Nhắc SOA.</strong> Serverless là lựa chọn <em>đóng gói và tính tiền</em>. Hợp đồng dịch vụ (timeout, idempotency, auth) vẫn như microservice đóng container.</p>
</div>

## 4. Mini kiến trúc: hướng sự kiện + lõi gRPC

Hình 2024–2026 sinh viên sẽ gặp:

1. API Gateway / Function cho HTTP bắc–nam *đột biến*.
2. Worker gRPC luôn bật cho lời gọi đông–tây *nói nhiều* (mesh hoặc ambient).
3. Object storage + hàng đợi cho payload không được sống trong thân sự kiện.

Đừng nhét generate LLM 30 giây vào timeout API Gateway mà không có job id bất đồng bộ.

## Thách thức

- Lỗi duy nhất SnapStart (chứng chỉ, RNG, pool kết nối).
- Khóa hệ sinh thái WASM (API host khác nhau).
- Chi phí: tính tiền theo ms vs. Deployment 1 replica rẻ hơn ở QPS ổn định.

## Bài tập

1. Liệt kê ba đối tượng lúc init phải xây lại sau restore SnapStart.
2. So sánh Lambda 128 MB vs. container 1 replica cho API CPU 20 RPS (định tính cũng được).
3. Phác thảo chỗ dùng WASM ở edge vs. Firecracker trong vùng.

## Tài liệu tham khảo

1. AWS, “Improving startup performance with Lambda SnapStart”.
2. AWS News Blog, SnapStart cho Python và .NET GA.
3. AWS What’s New, SnapStart cho hàm ảnh container (7/2026).
4. Fermyon Spin: [fermyon.com/spin](https://www.fermyon.com/spin).
5. CNCF *Cloud Native 2024 Annual Survey* (serverless tách).
6. Firecracker: [firecracker-microvm.github.io](https://firecracker-microvm.github.io/).
