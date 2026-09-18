---
layout: post
title: 15-02 Lab B — Khái niệm OpenTofu / Terraform, tránh hóa đơn bất ngờ
chapter: '15'
order: 3
owner: Nguyen Le Linh
lang: vi
categories:
- chapter15
lesson_type: optional
---

**Hình thức:** lab checklist.  
**Khung giờ:** khoảng 2 giờ.  
**Apply đám mây:** tắt mặc định. Chuẩn đầu ra là **khai báo → plan → đọc đồ thị**, không phải “cấp VPC trên thẻ tín dụng cá nhân.”

Tài liệu nguyên tác IUH. Không phải handout CS 40.

## Mục đích

Viết một stack **khai báo** *sẽ* mô tả mạng Bảng tin từ [01-07]({{ site.baseurl }}{% multilang_post_url contents/chapter01/21-01-01-01_07_Networking_Crash_Course %}), rồi chứng minh bạn lý luận được **state** và **plan**. Dùng **OpenTofu** (`tofu`) *hoặc* Terraform — cùng quy trình. Chương 14-02 giải thích vì sao OpenTofu tồn tại.

## Chuẩn đầu ra

- HCL đặt tên mạng, hai subnet, và ba tài nguyên kiểu security group *hoặc* stack **Docker provider** nếu ở 100% local.
- File `plan` (hoặc bản text) giải thích được từng dòng.
- Một đoạn về state sau apply — và vì sao hai người không được apply từ hai laptop vào một state từ xa khi chưa có khóa.

## Lộ 1 — Chỉ local (khuyên dùng)

Dùng provider **Docker** hoặc viết HCL như tài liệu thiết kế. Ưu tiên Docker provider để `init` + `plan` là thật.

- [ ] Cài OpenTofu **hoặc** Terraform; ghi phiên bản vào `NOTES.md`.
- [ ] `tofu init` (hoặc `terraform init`) thành công trong `lab-b/` trống.
- [ ] Khai báo ít nhất: một mạng Docker, một volume có tên, một container (ảnh Lab A hoặc `nginx:1.27-alpine`).
- [ ] Nhãn `project=iuh-courseboard` và `pair=...`.
- [ ] `tofu plan -out=lab.tfplan` được lưu. Nói được tài nguyên tạo / đổi / xóa.
- [ ] Đọc plan: không có `destroy` bất ngờ.
- [ ] **Nếu** apply local, phải `destroy` trước khi gấp máy, dán tóm tắt destroy.
- [ ] `NOTES.md`: state là gì, nằm đâu, hỏng gì nếu cả hai apply cùng lúc.

Lộ 1 **không** cấu hình provider AWS/Azure/GCP.

## Lộ 2 — Chỉ plan trên đám mây (giảng viên mở)

Chỉ khi đã có tài khoản **và** xong ngân sách [Lab C]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_03_Lab_Free_Tier_Notes %}).

- [ ] Provider ghim ràng buộc phiên bản.
- [ ] Vùng do giảng viên chỉ định.
- [ ] Tối thiểu: một mạng, một subnet công, một subnet riêng, ba security group (cạnh / api / db). **Không GPU, tránh NAT, không cụm Kubernetes.**
- [ ] Cả cặp đọc `plan`.
- [ ] Mặc định **dừng sau plan**. `apply` chỉ khi giảng viên viết đồng ý.
- [ ] Sau apply: thẻ, budget còn xanh, **destroy** trong ngày.

## Câu khái niệm (cả hai lộ)

1. Khai báo khác ClickOps ở console thế nào.
2. Vì sao `plan` chưa phải biên bảo mật (14-02).
3. Một câu về CDK hoặc Pulumi từ [mục so sánh 14-01]({{ site.baseurl }}{% multilang_post_url contents/chapter14/21-01-01-14_01_Infrastructure_as_Code %}).
4. Thứ **không bao giờ** để trong state (mật khẩu, dữ liệu sinh viên chưa mã hóa).

## Không làm

- Không commit `*.tfstate` hay plan có bí mật.
- Không chép module “cả VPC AWS” mà không giải thích được.
- Không tạo reserved 1–3 năm (bài FinOps).

## Xong khi

Cả hai dẫn được người thứ ba qua output plan. Có minh chứng destroy nếu đã apply.
