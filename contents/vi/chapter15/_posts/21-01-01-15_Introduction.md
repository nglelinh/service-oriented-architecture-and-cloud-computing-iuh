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

Đọc lý thuyết về Docker, Kubernetes, OpenTofu hay FinOps giúp bạn hiểu *bản đồ*, nhưng chỉ khi tự tay viết một tệp `Dockerfile` đa tầng, nhìn thấy dòng `tofu plan` cảnh báo xóa tài nguyên, và giải cứu một container bị lỗi mạng ngay trên terminal của chính mình, kiến thức đó mới thực sự trở thành kỹ năng nghề nghiệp.

Chương 15 là **xưởng thực hành tổng hợp** thiết kế riêng cho sinh viên IUH: mọi bài lab đều được sắp xếp theo nguyên tắc **"cục bộ trước, đám mây sau — an toàn tài chính tuyệt đối"**, dẫn dắt bạn đi từng bước chắc chắn từ chiếc laptop cá nhân đến một đồ án Capstone hoàn chỉnh hai người.

## Bản đồ kiến thức nền: Bốn năng lực thực chiến bạn rèn luyện xuyên suốt Chương 15

1. **Tự chủ môi trường cục bộ (Local-First Cloud Engineering)**:
   Hơn $85\%$ kỹ năng Cloud-Native (đóng gói container, mạng nhiều dịch vụ, kiểm thử sức khỏe, viết IaC khai báo, ghi log cấu trúc và chạy CI cơ bản) có thể luyện tập nhanh hơn, lặp lại vô hạn lần và **hoàn toàn miễn phí (`$0`)** ngay trên máy cá nhân trước khi chạm tới đám mây thật.

2. **Tư duy "Cơ chế an toàn mặc định" (Guardrails by Default)**:
   Thay vì sửa sai sau khi đã lộ khóa bí mật hay phát sinh hóa đơn, mọi bài lab trong chương đều thiết lập rào chắn ngay từ bước 0: `.gitignore` chặn `.env` và `*.tfstate`, tắt `tofu apply` lên đám mây theo mặc định, và kiểm tra giới hạn Free Tier trên giấy trước khi bấm nút.

3. **Kỹ năng chẩn đoán hệ thống thực tế (Debugging Methodology)**:
   Khi một hệ thống nhiều container không chạy, sinh viên học cách điều tra có phương pháp thay vì đoán mò: kiểm tra trạng thái (`docker compose ps`), đọc log có cấu trúc (`logs`), kiểm tra DNS/cổng nội bộ và đối chiếu biến môi trường.

4. **Hợp tác nhóm kỹ thuật (Two-Person Engineering Workflow)**:
   Đồ án Capstone nhóm hai người mô phỏng đúng quy trình doanh nghiệp: chia nhánh Git rõ ràng, mở Pull Request có đồng đội duyệt, chạy CI tự động kiểm thử, và viết tài liệu bàn giao để người ngoài có thể chạy lại hệ thống của bạn trong 5 phút.

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

### Khung giải thích (nên đọc chậm): Lộ trình 4 chặng thực hành và ma trận Thời gian – Rủi ro chi phí

Trước khi bắt tay vào Lab 15-01, hãy nhìn toàn cảnh 4 chặng của Chương 15 như một dây chuyền tích lũy kỹ năng—mỗi bài lab phía trước tạo trực tiếp nền móng (và mã nguồn tái sử dụng được!) cho đồ án Capstone 15-04:

| Chặng | Bài thực hành | Sản phẩm kỹ thuật giữ lại cho Capstone | Thời lượng dự kiến | Chi phí tối đa (nếu tuân thủ quy tắc) |
| :---: | :--- | :--- | :---: | :---: |
| **1** | **15-01: Docker Local** | `Dockerfile` multi-stage + `compose.yaml` + `/healthz` + Volume DB | `2–3 giờ` | **`$0`** (100% cục bộ) |
| **2** | **15-02: OpenTofu Concepts** | Thư mục `infra/` với `main.tf`, `variables.tf`, quy trình `fmt/validate/plan` | `2 giờ` | **`$0`** (provider cục bộ) |
| **3** | **15-03: Free-Tier Notes** | Checklist bảo mật Root MFA, Budget Alert `$1`, bảng ước tính FinOps | `1–1,5 giờ` | **`$0`** (thực hành trên giấy/sandbox) |
| **4** | **15-04: Capstone (2 SV)** | Ứng dụng **Bảng tin học phần IUH** hoàn chỉnh + CI + tài liệu kiến trúc & FinOps | `1–2 tuần` | **`$0`** (chạy chuẩn cục bộ hoặc free-tier có kiểm soát) |

Nếu hoàn thành tuần tự 4 chặng này, bạn không chỉ qua môn xuất sắc mà còn sở hữu ngay một dự án kỹ thuật sạch sẽ, có thể tự tin đưa vào hồ sơ ứng tuyển (CV/Portfolio) cho vị trí Backend, DevOps hoặc Cloud Engineer.

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
