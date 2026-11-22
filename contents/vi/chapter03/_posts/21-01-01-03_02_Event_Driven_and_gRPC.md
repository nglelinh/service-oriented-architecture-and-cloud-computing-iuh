---
layout: post
title: 03-02 Mô hình hướng sự kiện, gRPC và bộ lập lịch cụm (2022–2026)
chapter: '03'
order: 3
owner: Nguyen Le Linh
lang: vi
categories:
- chapter03
lesson_type: optional
---

Phần lý thuyết bắt buộc so sánh các mô hình client–server, ngang hàng (P2P) và hướng sự kiện, đồng thời giới thiệu YARN với vai trò bộ quản lý tài nguyên cụm. Bài tùy chọn này minh họa cách các *mô hình tính toán* đó xuất hiện trong kiến trúc dịch vụ giai đoạn 2022–2026: **gRPC** cho gọi thủ tục từ xa đồng bộ, **CloudEvents** cho giao tiếp bất đồng bộ chuẩn hóa, và **Kubernetes** với vai trò bộ lập lịch cụm phổ biến bên ngoài hệ sinh thái Hadoop truyền thống.

> Lý thuyết bắt buộc hiện ở bản tiếng Anh. Đây là bài ứng dụng tùy chọn bằng tiếng Việt.

## Mục tiêu học tập

- Đặt gRPC và CloudEvents vào đúng vị trí trên phổ mô hình client–server và hướng sự kiện.
- Giải thích vì sao Spark Connect (dựa trên gRPC) là một thay đổi ở cấp mô hình tính toán chứ không chỉ là tính năng bổ sung của Spark.
- So sánh vai trò giữa ApplicationMaster của YARN và controller trong Kubernetes.

## 1. Giao tiếp dịch vụ đồng bộ: gRPC

gRPC (dự án CNCF đã đạt trạng thái graduated) kết hợp HTTP/2, Protocol Buffers và cơ chế sinh mã stub tự động. Về bản chất, gRPC vẫn thuộc mô hình **client–server**, nhưng bổ sung các cơ chế quan trọng cho hệ thống phân tán:

- Hỗ trợ hạn chót (deadline) và hủy yêu cầu (cancellation) ngay ở tầng giao thức.
- Hỗ trợ RPC dạng luồng (client streaming, server streaming hoặc hai chiều), thu hẹp khoảng cách giữa mô hình hỏi–đáp và luồng sự kiện.
- Định tuyến trực tiếp ở tầng service mesh và cổng vào (tài nguyên **GRPCRoute** của Kubernetes Gateway API đã vào kênh Standard từ v1.1, tháng 5/2024).

```protobuf
syntax = "proto3";
service ResourceBroker {
  rpc Allocate (AllocateRequest) returns (AllocateReply);
  rpc WatchAllocations (WatchRequest) returns (stream AllocationEvent);
}
```

Trong định nghĩa trên, `Allocate` là lời gọi client–server truyền thống, còn `WatchAllocations` truyền luồng sự kiện *trên nền* RPC. Người thiết kế cần xác định rõ mình đang dùng mô hình nào trước khi xây dựng chính sách thử lại (retry).

## 2. Giao tiếp dịch vụ bất đồng bộ: CloudEvents và message broker

[CloudEvents](https://cloudevents.io/) (CNCF) định nghĩa một định dạng phong bì sự kiện chuẩn (`id`, `source`, `type`, `specversion`) để các dịch vụ phát (producer) và nhận (consumer) không phải tự tạo cấu trúc JSON riêng cho từng topic.

```python
event = {
    "specversion": "1.0",
    "id": "8a3c-2026-lab",
    "source": "urn:iuh:orders",
    "type": "com.iuh.order.created.v1",
    "datacontenttype": "application/json",
    "data": {"order_id": "A-19", "items": 3},
}
```

Đây chính là mô hình hướng sự kiện của phần lý thuyết, kết hợp với **quy ước đặt tên và định dạng chuẩn**. Trường khóa chống trùng lặp (`id`, dùng cho idempotency) giúp dịch vụ xử lý an toàn khi thông điệp bị gửi lại nhiều lần — bài toán đảm bảo độ tin cậy tương tự YARN ở xử lý lô, nay được giải quyết ở tầng *dịch vụ*.

## 3. Bộ lập lịch cụm hiện đại

| Khía cạnh | Hadoop YARN (lý thuyết bắt buộc) | Kubernetes (phổ biến thập niên 2020) |
| --- | --- | --- |
| Đơn vị thực thi | Application / container trên NodeManager | Pod và controller |
| Bộ lập lịch | ResourceManager và hàng đợi | `kube-scheduler` và lớp ưu tiên |
| Điều phối từng ứng dụng | ApplicationMaster | Operator / Job / Spark Driver |
| Hỗ trợ đa framework | MapReduce, Spark, Flink trên cùng cụm HDFS | Mọi ứng dụng container hóa trên một API thống nhất |

YARN vẫn được duy trì trong nhiều nền tảng dữ liệu lớn hiện có. Tuy nhiên, theo khảo sát CNCF 2024 (công bố năm 2025), **80%** tổ chức đã chạy Kubernetes trên môi trường sản xuất. Nguyên lý cốt lõi đã học — tách bạch **điều phối tài nguyên** khỏi **logic ứng dụng** — vẫn giữ nguyên và được chuyển sang control plane của Kubernetes.

**Spark Connect** (ra mắt từ Apache Spark 3.4 và củng cố ở **Spark 4.0.0**, ngày 23/5/2025) hiện thực hóa triệt để sự tách bạch này: tiến trình người dùng chỉ là một client gRPC gọn nhẹ, còn toàn bộ Spark session chạy tập trung trên cụm.

## 4. Nguyên tắc lựa chọn nhanh

1. Cần phản hồi ngay kèm hạn chót rõ ràng? Chọn **gRPC / HTTP** (client–server).
2. Cần phản ứng bất đồng bộ trước các sự kiện đã xảy ra? Chọn **CloudEvents + broker** (hướng sự kiện).
3. Cần cấp phát CPU/GPU/bộ nhớ cho tác vụ kéo dài từ vài phút đến nhiều giờ? Dùng **bộ lập lịch cụm** (YARN hoặc Kubernetes).

<div class="content-box insight-box">
<p><strong>Mô hình cần tránh (anti-pattern).</strong> Sử dụng hàng đợi thông điệp như cơ chế RPC đồng bộ ("gửi tin rồi ngồi chờ sự kiện phản hồi") mà không thiết lập timeout thực chất chỉ là tái tạo mô hình client–server nhưng khó gỡ lỗi hơn. Khi bài toán cần RPC đồng bộ, hãy dùng thẳng RPC.</p>
</div>

## Thách thức

- Đảm bảo hiệu ứng nghiệp vụ "xử lý đúng một lần" (exactly-once) vẫn đòi hỏi consumer có tính lũy đẳng (idempotent), vì đa số cloud broker chỉ cam kết gửi ít nhất một lần (at-least-once).
- Dùng gRPC trực tiếp từ trình duyệt qua Internet công cộng thường cần lớp chuyển đổi trung gian (như Connect, gRPC-Web hoặc Gateway API).
- Vận hành song song cả YARN và Kubernetes trong cùng tổ chức dẫn đến hai hệ thống hàng đợi và hai mô hình quản lý định danh khác nhau.

## Video tham khảo

- **Xem trước:** [gRPC in 5 minutes \| Eric Anderson & Ivy Zhuang, Google](https://www.youtube.com/watch?v=njC24ts24Pg) — gRPC (tiếng Anh). Tóm tắt gRPC trong năm phút, phục vụ mục SOA đồng bộ.
- [What is the Cloud Events Specification & Why You Should Care!](https://www.youtube.com/watch?v=Sjw0PFwTvek) — James Eastham (tiếng Anh). Giải thích đặc tả CloudEvents cho mục hợp đồng sự kiện bất đồng bộ.
- [What is Kubernetes?](https://www.youtube.com/watch?v=cC46cg5FFAM) — Google Cloud Tech (tiếng Anh). Giới thiệu Kubernetes — bộ lập lịch cụm phổ biến thay thế YARN ngoài hệ sinh thái Hadoop.

## Bài tập

1. Phân loại năm tương tác trong một ứng dụng gọi xe (tính giá cước, luồng cập nhật vị trí tài xế, hoàn tất chuyến đi, gửi email hóa đơn, chấm điểm gian lận) thành lời gọi RPC đồng bộ hoặc sự kiện bất đồng bộ.
2. Viết đoạn định nghĩa `.proto` khoảng 10 dòng cho `WatchAllocations` và nêu hai dạng lỗi đặc thù của RPC streaming không xuất hiện ở unary RPC.
3. Viết một đoạn ngắn so sánh vai trò giữa ApplicationMaster của YARN với sự kết hợp của Kubernetes `Job` và driver Spark Connect.

## Tài liệu tham khảo

1. Kubernetes SIG Network, “Gateway API v1.1” (9/5/2024): [kubernetes.io/blog/2024/05/09/gateway-api-v1-1](https://kubernetes.io/blog/2024/05/09/gateway-api-v1-1/).
2. Apache Spark, *Spark Release 4.0.0* (23/5/2025).
3. Apache Spark, *Spark Connect Overview*.
4. CloudEvents: [cloudevents.io](https://cloudevents.io/).
5. CNCF, *Cloud Native 2024 Annual Survey*.
6. Tài liệu gRPC: [grpc.io](https://grpc.io/).
