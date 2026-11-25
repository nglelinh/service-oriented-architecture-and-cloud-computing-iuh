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

Bạn không cần thẻ tín dụng hay tài khoản AWS/GCP để học bản chất của Infrastructure as Code. Nhờ kiến trúc **Provider dạng plugin** của OpenTofu, bạn có thể luyện tập trọn vẹn vòng đời `init → fmt → validate → plan → apply → destroy` hoàn toàn ngoại tuyến ngay trên máy cá nhân bằng các provider cục bộ (`local`, `random`), hiểu tận gốc tệp State và đồ thị phụ thuộc mà rủi ro phát sinh hóa đơn bằng **đúng 0 đồng**.

## Kiến thức cần nhớ: Bốn bước vòng đời bắt buộc của mọi dự án OpenTofu

1. **Khởi tạo & Khóa phiên bản (`tofu init` + `.terraform.lock.hcl`)**:
   Lệnh `tofu init` tải các binary provider cần thiết và ghi mã băm SHA-256 của chúng vào tệp `.terraform.lock.hcl`. Tệp khóa này **phải được commit vào Git** để đồng đội và CI dùng chính xác cùng một phiên bản provider.

2. **Kiểm tra cú pháp & Chuẩn hóa (`tofu fmt`, `tofu validate`)**:
   `fmt` định dạng mã HCL thống nhất toàn nhóm; `validate` kiểm tra kiểu biến, tham chiếu thuộc tính và cấu trúc khối trước khi cần chạm tới bất kỳ API bên ngoài nào.

3. **Lập kế hoạch & Kiểm duyệt (`tofu plan`)**:
   Đọc mã `.tf` + đọc tệp trạng thái `terraform.tfstate` + dựng đồ thị phụ thuộc DAG để in ra bản diff chi tiết (`+ create`, `~ update`, `- destroy`, `-/+ replace`) mà **chưa thay đổi gì trong thực tế**.

4. **Tách biệt Mã nguồn khỏi Giá trị Môi trường (`variables.tf` vs. `*.tfvars`)**:
   Mã cấu trúc (`main.tf`) viết một lần, còn tham số thay đổi theo môi trường (tên dự án, số bản sao, vùng triển khai) được truyền qua tệp `.tfvars` hoặc biến môi trường `TF_VAR_*`—và tuyệt đối không commit tệp chứa secret hay tệp `terraform.tfstate` lên Git công khai!

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

### Giải thích: Trace vòng đời 2 tài nguyên cục bộ (`random_id` → `local_file`) qua `plan`, `apply` và đổi biến

Giả sử trong bài lab cục bộ, tệp `main.tf` khai báo 2 tài nguyên nối nhau theo quan hệ phụ thuộc:
```hcl
resource "random_id" "suffix" {
  byte_length = 2
}

resource "local_file" "config" {
  filename = "${path.module}/app-${random_id.suffix.hex}.conf"
  content  = "env=${var.env}\n"
}
```

Theo dõi chính xác OpenTofu dựng DAG và cập nhật `terraform.tfstate` qua 3 thao tác:

| Thao tác | Đồ thị / So sánh State | Ký hiệu `plan` & Kết quả |
| :--- | :--- | :--- |
| **1. Lần đầu chạy `tofu apply` (`var.env = "dev"`)** | State rỗng. DAG xác định `random_id.suffix` phải tạo **trước** `local_file.config` vì thuộc tính `.hex` phụ thuộc vào nó. | `+ random_id.suffix` (sinh `a3f1`) → `+ local_file.config` (tạo tệp `app-a3f1.conf`). State lưu 2 bản ghi. |
| **2. Chạy lại `tofu plan` ngay lập tức (không sửa gì)** | Mã HCL khớp 100% với State và tệp trên đĩa. | **`No changes. Your infrastructure matches the configuration.`** (Tính lũy đẳng!) |
| **3. Sửa `var.env = "staging"` rồi chạy `tofu plan`** | `random_id.suffix` không đổi (giữ nguyên `a3f1`), nhưng `local_file.config.content` thay đổi (thuộc tính *ForceNew* của `local_file`). | **`-/+ local_file.config must be replaced`** (`1 to add, 0 to change, 1 to destroy`) — xóa tệp cũ và tạo lại tệp mới với `env=staging`. |

Nhìn qua 3 bước nhỏ chạy mất chưa đầy 2 giây này, bạn đã nắm đúng 100% cơ chế lõi mà OpenTofu dùng để quản lý hàng nghìn máy chủ trên AWS, Azure hay GCP!

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
