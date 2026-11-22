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

**Hình thức:** Bài thực hành dạng checklist.  
**Thời lượng dự kiến:** Khoảng 2 giờ.  
**Chạy `apply` lên đám mây:** Tắt theo mặc định. Mục tiêu của bài là nắm vững quy trình **khai báo cấu hình → chạy `plan` → đọc đồ thị thay đổi**, không phải cấp phát VPC trên thẻ tín dụng cá nhân.

Tài liệu được biên soạn độc lập cho IUH, không sử dụng tài liệu bài tập của CS 40.

## Mục đích

Viết một cấu hình hạ tầng **khai báo (declarative)** mô tả kiến trúc mạng của ứng dụng Bảng tin học phần từ bài [01-07]({{ site.baseurl }}{% multilang_post_url contents/chapter01/21-01-01-01_07_Networking_Crash_Course %}), sau đó chứng minh bạn hiểu rõ cơ chế **state** và **plan**. Bạn có thể sử dụng **OpenTofu** (`tofu`) *hoặc* Terraform vì quy trình làm việc tương đương (xem bài 14-02 để hiểu bối cảnh ra đời của OpenTofu).

## Chuẩn đầu ra

- Cấu hình HCL khai báo mạng, hai subnet và ba tài nguyên kiểu security group, *hoặc* một cấu hình dùng **Docker provider** nếu thực hành 100% cục bộ.
- Tệp `plan` (hoặc bản xuất văn bản) mà nhóm có thể giải thích rõ từng dòng thay đổi.
- Một đoạn trình bày ngắn về trạng thái `state` sau khi chạy `apply`, cùng lý do hai người không được phép chạy `apply` đồng thời từ hai máy vào cùng một remote state khi chưa có cơ chế khóa (state locking).

## Hướng 1 — Chạy hoàn toàn cục bộ (khuyến nghị)

Sử dụng **Docker provider** để có thể chạy `init` và `plan` thực tế trên máy cá nhân mà không cần tài khoản đám mây.

- [ ] Cài đặt OpenTofu **hoặc** Terraform và ghi rõ phiên bản vào `NOTES.md`.
- [ ] Chạy `tofu init` (hoặc `terraform init`) thành công trong thư mục `lab-b/` mới.
- [ ] Khai báo tối thiểu: một mạng Docker, một volume có tên và một container (dùng ảnh từ Lab A hoặc `nginx:1.27-alpine`).
- [ ] Gắn nhãn `project=iuh-courseboard` và `pair=...`.
- [ ] Chạy và lưu tệp kế hoạch `tofu plan -out=lab.tfplan`; giải thích rõ tài nguyên nào sẽ được tạo mới, thay đổi hoặc xóa.
- [ ] Kiểm tra kỹ bản `plan` để chắc chắn không có thao tác `destroy` ngoài ý muốn.
- [ ] **Nếu** chạy `apply` cục bộ, bắt buộc chạy `destroy` trước khi kết thúc buổi thực hành và lưu lại bản tóm tắt kết quả `destroy`.
- [ ] Ghi vào `NOTES.md`: `state` là gì, được lưu ở đâu và điều gì xảy ra nếu cả hai thành viên chạy `apply` cùng lúc.

Hướng 1 **không** cấu hình provider của AWS, Azure hay GCP.

## Hướng 2 — Chỉ chạy `plan` trên đám mây (khi giảng viên cho phép)

Chỉ thực hiện khi lớp đã được cấp tài khoản **và** nhóm đã hoàn thành thiết lập ngân sách trong [Lab C]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_03_Lab_Free_Tier_Notes %}).

- [ ] Cố định ràng buộc phiên bản cho provider.
- [ ] Chọn đúng vùng địa lý (region) do giảng viên chỉ định.
- [ ] Khai báo tối thiểu: một mạng ảo, một subnet công khai, một subnet riêng tư và ba security group (`edge`, `api`, `db`). **Không tạo GPU, tránh tạo NAT Gateway và không khởi tạo cụm Kubernetes.**
- [ ] Cả hai thành viên cùng rà soát kết quả `plan`.
- [ ] Theo mặc định, **dừng lại sau bước `plan`**; chỉ chạy `apply` khi có xác nhận bằng văn bản của giảng viên.
- [ ] Nếu được phép `apply`: kiểm tra thẻ gắn tài nguyên, xác nhận ngân sách an toàn và chạy **`destroy`** trong cùng ngày.

## Câu hỏi khái niệm (áp dụng cho cả hai hướng)

1. Cách tiếp cận khai báo (IaC) khác với thao tác thủ công trên giao diện web (ClickOps) ở điểm nào?
2. Vì sao lệnh `plan` không tự động đảm bảo cấu hình an toàn bảo mật (xem bài 14-02)?
3. Nêu một nhận xét ngắn về CDK hoặc Pulumi dựa trên [mục so sánh trong bài 14-01]({{ site.baseurl }}{% multilang_post_url contents/chapter14/21-01-01-14_01_Infrastructure_as_Code %}).
4. Liệt kê những loại thông tin **tuyệt đối không** lưu trực tiếp ở dạng rõ trong tệp state (như mật khẩu hay dữ liệu sinh viên chưa mã hóa).

## Những điều không nên làm

- Không commit tệp `*.tfstate` hoặc tệp `plan` có thể chứa thông tin nhạy cảm lên Git.
- Không sao chép các module dựng sẵn quá lớn (như toàn bộ kiến trúc VPC doanh nghiệp) mà nhóm không giải thích được.
- Không khởi tạo tài nguyên cam kết 1–3 năm (xem bài FinOps).

## Tiêu chí hoàn thành

Cả hai thành viên đều có thể giải thích rõ từng phần trong kết quả `plan` cho người thứ ba, kèm minh chứng đã `destroy` nếu có chạy `apply`.

## Video tham khảo

- **Xem trước:** [Introduction to HashiCorp Terraform with Armon Dadgar](https://www.youtube.com/watch?v=h970ZBgKINg) — HashiCorp, an IBM Company (tiếng Anh). Minh họa quy trình `plan` trước khi `apply` để tránh thay đổi hạ tầng ngoài ý muốn.
- [What is OpenTofu Project?](https://www.youtube.com/watch?v=J2JKxY2elSo) — Oleksiy Pototskyy (tiếng Anh). Giới thiệu dự án OpenTofu đặt song song với Terraform trong bài lab.
- [Terraform là gì? Nhập môn Infrastructure as Code - Sử dụng Terraform để provision infra trên GCP](https://www.youtube.com/watch?v=ORnsaa7wDv8) — FullstacKAGE (tiếng Việt). Hướng dẫn nhập môn Terraform bằng tiếng Việt trước khi thực hành chạy `plan`.
