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

Bài bắt buộc đối chiếu client–server, P2P và hướng sự kiện, rồi giới thiệu YARN như hệ điều hành cụm. Bài tùy chọn này cho thấy các *mô hình tính toán* đó trên kiến trúc dịch vụ 2022–2026: **gRPC** cho RPC đồng bộ, **CloudEvents** cho hợp đồng bất đồng bộ, và **Kubernetes** như bộ lập lịch đã thay YARN ngoài các estate Hadoop.

> Lý thuyết bắt buộc đang ở bản tiếng Anh. Đây là phần ứng dụng tùy chọn bằng tiếng Việt.

## Mục tiêu học tập

- Đặt gRPC và CloudEvents lên cùng sơ đồ mô hình với client–server vs hướng sự kiện.
- Giải thích vì sao Spark Connect (gRPC) là thay đổi mô hình tính toán, không chỉ là tính năng Spark.
- So sánh ApplicationMaster của YARN với controller Kubernetes mà không viết lại nội bộ YARN.

## 1. SOA đồng bộ: gRPC như stub hiện đại

gRPC (CNCF graduated) là HTTP/2 + Protocol Buffers + stub sinh mã. Vẫn là **client–server**, nhưng có:

- Deadline và hủy (timeout hạng nhất).
- RPC streaming (client, server, hoặc hai chiều) làm mờ “hỏi/đáp” và “luồng sự kiện.”
- Kiểu nội dung mà service mesh định tuyến được (Gateway API **GRPCRoute** vào kênh Standard ở v1.1, tháng 5/2024).

```protobuf
syntax = "proto3";
service ResourceBroker {
  rpc Allocate (AllocateRequest) returns (AllocateReply);
  rpc WatchAllocations (WatchRequest) returns (stream AllocationEvent);
}
```

`Allocate` là client–server cổ điển. `WatchAllocations` là luồng sự kiện *trên* RPC. Sinh viên phải nói được mình đang ở mô hình nào trước khi chọn chính sách retry.

## 2. SOA bất đồng bộ: CloudEvents và broker

[CloudEvents](https://cloudevents.io/) (CNCF) là phong bì chung (`id`, `source`, `type`, `specversion`) để producer và consumer không invent JSON mới mỗi topic.

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

Đây là mô hình hướng sự kiện của bài bắt buộc với **tên chuẩn** cho tin nhắn. Khóa idempotency (`id`) là cách sống sót sau retry độc lập — cùng vấn đề tin cậy YARN giải cho tác vụ lô, nay ở lớp *dịch vụ*.

## 3. Ai là hệ điều hành cụm bây giờ?

| Quan tâm | Hadoop YARN (bài bắt buộc) | Kubernetes (mặc định những năm 2020) |
| --- | --- | --- |
| Đơn vị công việc | Application / container trên NM | Pod + controller |
| Bộ lập lịch | RM + hàng đợi | kube-scheduler + priority |
| Bộ não từng việc | ApplicationMaster | Operator / Job / Spark Driver |
| Đa framework | MR, Spark, Flink trên một cụm HDFS | Mọi container trên một API |

YARN không biến mất: nhiều nền tảng dữ liệu vẫn lập lịch Spark trên YARN. Nhưng khảo sát CNCF 2024 (công bố 2025) đặt **Kubernetes trên production ở 80%** người trả lời. Ý tưởng đã học — tách **thương lượng tài nguyên** khỏi **logic ứng dụng** — chuyển sang control plane khác.

**Spark Connect** (Apache Spark 3.4+, củng cố ở **Spark 4.0.0**, 23/5/2025) làm sự tách này thành nghĩa đen: tiến trình người dùng là client gRPC mỏng; cụm giữ Spark session.

## 4. Hướng dẫn quyết định ngắn

1. Cần câu trả lời ngay với hạn chót? **gRPC / HTTP** (client–server).
2. Cần phản ứng với sự kiện đã xảy ra? **CloudEvent + broker** (hướng sự kiện).
3. Cần đặt CPU/GPU/bộ nhớ vài phút đến vài giờ? **Bộ lập lịch** (YARN hoặc Kubernetes).

<div class="content-box insight-box">
<p><strong>Phản mẫu.</strong> Dùng bus tin nhắn như RPC đồng bộ (“chờ sự kiện trả lời”) mà không timeout là tái tạo client–server với khả năng gỡ lỗi tệ hơn. Cần RPC thì dùng RPC.</p>
</div>

## Thách thức

- Hiệu ứng nghiệp vụ exactly-once vẫn cần consumer idempotent; hầu hết cloud broker chỉ cho at-least-once.
- gRPC qua trình duyệt/Internet công cộng thường cần proxy (Connect/gRPC-Web, Gateway API).
- Chạy cả YARN và Kubernetes trong một tổ chức tạo hai hàng đợi, hai mô hình định danh.

## Bài tập

1. Phân loại năm lời gọi trong ứng dụng gọi xe (báo giá, luồng vị trí tài xế, chuyến xong, email hóa đơn, điểm gian lận) thành RPC vs sự kiện.
2. Viết proto khoảng 10 dòng cho `WatchAllocations` và liệt kê hai chế độ lỗi của stream mà unary RPC không có.
3. Một đoạn văn: ánh xạ ApplicationMaster của YARN sang `Job` Kubernetes + driver Spark Connect.

## Tài liệu tham khảo

1. Kubernetes SIG Network, “Gateway API v1.1” (9/5/2024): [kubernetes.io/blog/2024/05/09/gateway-api-v1-1](https://kubernetes.io/blog/2024/05/09/gateway-api-v1-1/).
2. Apache Spark, *Spark Release 4.0.0* (23/5/2025).
3. Apache Spark, *Spark Connect Overview*.
4. CloudEvents: [cloudevents.io](https://cloudevents.io/).
5. CNCF, *Cloud Native 2024 Annual Survey*.
6. Tài liệu gRPC: [grpc.io](https://grpc.io/).
