---
layout: post
title: 15 Lộ trình triển khai — từ mạng tới đồ án
chapter: '15'
order: 1
owner: Nguyen Le Linh
lang: vi
categories:
- chapter15
lesson_type: optional
---

**Lộ trình triển khai** tùy chọn này là đường IUH đi qua các kỹ năng *bên cạnh* xương sống dữ liệu lớn (Hadoop, Spark, NoSQL). Nó lấy cảm hứng từ cung triển khai trước của [Stanford CS 40 / infracourse.cloud](https://infracourse.cloud/) (Winter 2024). **Không bài, lab, hay đồ án IUH nào sao chép đề, rubrics, hoặc câu chữ CS 40.** Chúng tôi tái sử dụng *thứ tự chủ đề*—mạng, container, điều phối, IaC, định danh, quan sát, CI/CD, chi phí—và viết tài liệu campus nguyên tác.

Dùng hub này khi bạn muốn **đưa một dịch vụ đa tầng nhỏ lên chạy**, không chỉ giải thích MapReduce.

## Bạn sẽ làm gì

1. Đọc bốn bài lý thuyết tùy chọn (mạng, quan sát sâu, CI/CD, FinOps).
2. Làm **hai hoặc ba checklist lab** ưu tiên máy local (Docker; khái niệm OpenTofu/Terraform; vệ sinh free-tier).
3. **Nhóm 2 người**, làm [đồ án]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_04_Capstone %}) (cùng nội dung trong [`PROJECT.md`](https://github.com/nglelinh/service-oriented-architecture-and-cloud-computing-iuh/blob/main/PROJECT.md) của repo).

Không bắt buộc tốn tiền đám mây. Cả lộ trình có thể ở trên một laptop. Nếu chạm đám mây công cộng, Lab C và bài FinOps phải *trước* lời gọi API tính phí đầu tiên.

## Thứ tự đọc gợi ý

| Bước | Tài liệu | Chương |
| --- | --- | --- |
| 1 | Chương 01 bắt buộc (NIST, mô hình) | 01 |
| 2 | [01-07 Nhập môn mạng]({{ site.baseurl }}{% multilang_post_url contents/chapter01/21-01-01-01_07_Networking_Crash_Course %}) | 01 |
| 3 | Chương 08–09 bắt buộc | 08–09 |
| 4 | [Lab A — Docker đa tầng local]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_01_Lab_Docker_Local %}) | 15 |
| 5 | Chương 13–14 bắt buộc | 13–14 |
| 6 | [13-03 Quan sát sâu]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_03_Observability_Depth %}) | 13 |
| 7 | [13-04 CI/CD]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_04_CICD_Cloud_Apps %}) | 13 |
| 8 | [Lab B — OpenTofu/Terraform]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_02_Lab_OpenTofu_Concepts %}) | 15 |
| 9 | [12-03 FinOps]({{ site.baseurl }}{% multilang_post_url contents/chapter12/21-01-01-12_03_FinOps_Billing_Literacy %}) | 12 |
| 10 | [Lab C — free-tier]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_03_Lab_Free_Tier_Notes %}) | 15 |
| 11 | [Đồ án Bảng tin]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_04_Capstone %}) | 15 |

Bổ sung P1 nằm *trong* bài bắt buộc tiếng Anh: Chương 08 (thủ công → quản lý), 09 (rolling / canary), 12 (checklist quy mô + chi phí), 14 (so CDK / Pulumi).

## Ánh xạ chủ đề CS 40 → IUH

| Khối kiểu CS 40 (infracourse.cloud) | Chỗ IUH đặt |
| --- | --- |
| Nền tảng | Chương 01 bắt buộc |
| Mạng, DNS, TLS | [01-07]({{ site.baseurl }}{% multilang_post_url contents/chapter01/21-01-01-01_07_Networking_Crash_Course %}) + Lab A/C |
| Lưu trữ / CSDL | Chương 07 và 10 |
| Container / điều phối | Chương 08–09 + Lab A |
| Hạ tầng dưới dạng mã | Chương 14 + Lab B + so sánh CDK/Pulumi trong 14-01 |
| IAM / bảo mật | Chương 13 + 13-02 |
| Quan sát | 02-02, 13-02, **[13-03]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_03_Observability_Depth %})** |
| Serverless / phục vụ ML | 08-04, 08-05, 01-06, 12-02 |
| CI/CD | 13-01 + **[13-04]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_04_CICD_Cloud_Apps %})** |
| Chi phí / đạo đức | 01-06, **[12-03]({{ site.baseurl }}{% multilang_post_url contents/chapter12/21-01-01-12_03_FinOps_Billing_Literacy %})**, Lab C |
| Đồ án triển khai | **[15-04]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_04_Capstone %})** (cặp đôi) |

CS 40 là khóa *triển khai*. IUH vẫn là khóa *nền tảng + dữ liệu* với lộ trình này như xương tùy chọn. Có thể hoàn thành phần khái niệm cốt lõi mà không cần Chương 15; không thể nhận “chúng tôi đã triển khai dịch vụ” chỉ bằng slide lý thuyết.

## Quy tắc an toàn IUH (mọi lab)

- Ưu tiên **localhost**. Đám mây tùy chọn và do giảng viên cửa.
- **Cảnh báo ngân sách trước apply.** Destroy thuộc buổi demo.
- Không bí mật trong Git. Không `0.0.0.0/0` trên SSH hoặc cổng CSDL.
- Không chép đề trường khác vào báo cáo. Trích ý, viết thiết kế của mình.

## Tài liệu

1. [infracourse.cloud](https://infracourse.cloud/) — CS 40 Winter 2024, Stanford.
2. [Bản đồ chủ đề trong README](https://github.com/nglelinh/service-oriented-architecture-and-cloud-computing-iuh#inspiration-stanford-cs-40--infracoursecloud).
3. Các chương 01, 08–09, 12–14 như liên kết trên.
