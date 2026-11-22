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

**Lộ trình triển khai** tùy chọn này tập trung vào các kỹ năng kỹ thuật hệ thống *song hành* với mảng dữ liệu lớn (Hadoop, Spark, NoSQL). Thứ tự chủ đề được tham khảo từ cách tiếp cận ưu tiên thực hành triển khai của [Stanford CS 40 / infracourse.cloud](https://infracourse.cloud/) (Winter 2024) — gồm mạng, container, điều phối, hạ tầng dưới dạng mã (IaC), định danh, quan sát, CI/CD và quản trị chi phí. **Không có bài học, bài lab hay đồ án nào của IUH sao chép đề bài, tiêu chí chấm hoặc nội dung từ CS 40**; toàn bộ bài tập và tài liệu đều được biên soạn độc lập cho bối cảnh sinh viên IUH.

Hãy sử dụng trang tổng hợp này khi bạn muốn **tự thiết kế và đưa một dịch vụ đa tầng nhỏ vào vận hành thực tế**, thay vì chỉ dừng ở mức lý thuyết.

## Bạn sẽ thực hiện những gì

1. Đọc bốn bài lý thuyết tùy chọn chuyên sâu: mạng đám mây, quan sát hệ thống, CI/CD và FinOps.
2. Hoàn thành **hai đến ba bài lab dạng checklist** ưu tiên chạy trên máy cục bộ (Docker Compose, khái niệm OpenTofu/Terraform và nguyên tắc dùng free-tier an toàn).
3. Thực hiện [đồ án nhóm hai người]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_04_Capstone %}) (đồng bộ với tệp [`PROJECT.md`](https://github.com/nglelinh/service-oriented-architecture-and-cloud-computing-iuh/blob/main/PROJECT.md) trong kho mã nguồn).

Lộ trình này **không** yêu cầu phát sinh chi phí đám mây; toàn bộ nội dung có thể hoàn thành trên một máy tính cá nhân. Nếu có sử dụng đám mây công cộng, bạn bắt buộc phải hoàn thành Lab C và bài FinOps *trước* khi khởi tạo tài nguyên có tính phí đầu tiên.

## Thứ tự học tập gợi ý

| Bước | Tài liệu | Chương |
| --- | --- | --- |
| 1 | Chương 01 bắt buộc (NIST, mô hình dịch vụ và triển khai) | 01 |
| 2 | [01-07 Nhập môn mạng đám mây]({{ site.baseurl }}{% multilang_post_url contents/chapter01/21-01-01-01_07_Networking_Crash_Course %}) | 01 |
| 3 | Chương 08–09 bắt buộc | 08–09 |
| 4 | [Lab A — Ứng dụng đa tầng cục bộ với Docker Compose]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_01_Lab_Docker_Local %}) | 15 |
| 5 | Chương 13–14 bắt buộc | 13–14 |
| 6 | [13-03 Quan sát sâu]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_03_Observability_Depth %}) | 13 |
| 7 | [13-04 CI/CD cho ứng dụng đám mây]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_04_CICD_Cloud_Apps %}) | 13 |
| 8 | [Lab B — Khái niệm OpenTofu/Terraform]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_02_Lab_OpenTofu_Concepts %}) | 15 |
| 9 | [12-03 FinOps và đọc hóa đơn đám mây]({{ site.baseurl }}{% multilang_post_url contents/chapter12/21-01-01-12_03_FinOps_Billing_Literacy %}) | 12 |
| 10 | [Lab C — Nguyên tắc free-tier an toàn]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_03_Lab_Free_Tier_Notes %}) | 15 |
| 11 | [Đồ án Bảng tin học phần IUH]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_04_Capstone %}) | 15 |

Các phần bổ sung liên kết nằm *ngay trong* bài bắt buộc bản tiếng Anh: Chương 08 (chuyển từ thủ công sang dịch vụ quản lý), Chương 09 (cập nhật rolling và canary), Chương 12 (danh sách kiểm tra quy mô và chi phí), Chương 14 (so sánh CDK và Pulumi).

## Ánh xạ chủ đề từ CS 40 sang chương trình IUH

| Nhóm chủ đề kiểu CS 40 (infracourse.cloud) | Vị trí tương ứng trong khóa học IUH |
| --- | --- |
| Nền tảng đám mây | Chương 01 bắt buộc |
| Mạng, DNS và TLS | [01-07]({{ site.baseurl }}{% multilang_post_url contents/chapter01/21-01-01-01_07_Networking_Crash_Course %}) + Lab A và Lab C |
| Lưu trữ và cơ sở dữ liệu | Chương 07 và Chương 10 |
| Container và điều phối | Chương 08–09 + Lab A |
| Hạ tầng dưới dạng mã (IaC) | Chương 14 + Lab B + phần so sánh CDK/Pulumi trong 14-01 |
| IAM và bảo mật | Chương 13 + bài 13-02 |
| Khả năng quan sát | 02-02, 13-02 và **[13-03]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_03_Observability_Depth %})** |
| Serverless và phục vụ ML | 08-04, 08-05, 01-06 và 12-02 |
| CI/CD | 13-01 và **[13-04]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_04_CICD_Cloud_Apps %})** |
| Chi phí và đạo đức vận hành | 01-06, **[12-03]({{ site.baseurl }}{% multilang_post_url contents/chapter12/21-01-01-12_03_FinOps_Billing_Literacy %})** và Lab C |
| Đồ án triển khai | **[15-04]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_04_Capstone %})** (nhóm 2 sinh viên) |

Nếu CS 40 là môn học chuyên biệt về *triển khai*, thì môn học tại IUH vẫn giữ trọng tâm *nền tảng đám mây và dữ liệu* kết hợp nhánh thực hành tùy chọn này. Bạn vẫn có thể nắm phần lý thuyết cốt lõi mà không cần Chương 15, nhưng để thực sự tự tin triển khai một dịch vụ hoàn chỉnh thì không thể chỉ dựa vào slide bài giảng.

## Quy tắc an toàn áp dụng cho mọi bài lab

- Ưu tiên chạy trên **localhost**; việc dùng đám mây công cộng là tùy chọn và cần có sự cho phép của giảng viên.
- **Thiết lập cảnh báo ngân sách trước khi chạy `apply`**, và đưa bước dọn dẹp (`destroy`) vào cuối buổi trình diễn.
- Không lưu khóa bí mật trong Git; không mở `0.0.0.0/0` cho cổng SSH hoặc cổng cơ sở dữ liệu.
- Không sao chép đề bài hay nội dung từ trường khác vào báo cáo; chỉ tham khảo ý tưởng và tự trình bày thiết kế của nhóm.

## Video tham khảo

- **Xem trước:** [What is Docker Compose? \| Docker Concepts](https://www.youtube.com/watch?v=f3xb7kt_dH4) — Docker (tiếng Anh). Giới thiệu Docker Compose — bước khởi đầu cho ứng dụng đa tầng trên máy cục bộ.
- [Introduction to HashiCorp Terraform with Armon Dadgar](https://www.youtube.com/watch?v=h970ZBgKINg) — HashiCorp, an IBM Company (tiếng Anh). Trình bày quy trình `plan` và `apply` của hạ tầng dưới dạng mã đặt sau bài lab cục bộ.
- [How to use GitHub Actions \| GitHub for Beginners](https://www.youtube.com/watch?v=BQrohJ3PT7I) — GitHub (tiếng Anh). Nhập môn GitHub Actions cho phần tự động hóa CI/CD.

## Tài liệu tham khảo

1. [infracourse.cloud](https://infracourse.cloud/) — CS 40 Winter 2024, Đại học Stanford.
2. [Bản đồ chủ đề trong README](https://github.com/nglelinh/service-oriented-architecture-and-cloud-computing-iuh#inspiration-stanford-cs-40--infracoursecloud).
3. Các chương 01, 08–09 và 12–14 theo các liên kết đã dẫn ở trên.
