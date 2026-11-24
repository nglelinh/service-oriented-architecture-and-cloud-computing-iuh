---
layout: post
title: 14-02 OpenTofu, chính sách dưới dạng mã và platform engineering
chapter: '14'
order: 3
owner: Nguyen Le Linh
lang: vi
categories:
- chapter14
lesson_type: optional
---

Khi toàn bộ hạ tầng đám mây đã được viết thành mã nguồn HCL trong kho Git, một câu hỏi quản trị mới lập tức xuất hiện: *làm sao đảm bảo một lập trình viên mới vào nhóm không vô tình mở cổng SSH `0.0.0.0/0`, tạo bucket S3 công khai toàn thế giới, hay bật nhầm máy chủ 64 nhân tốn 2.000 USD/tháng chỉ bằng một Pull Request?* Nếu chỉ dựa vào mắt người duyệt code (manual review), sai sót chắc chắn sẽ lọt qua vào một chiều thứ Sáu bận rộn.

Bài học này khám phá bộ đôi công nghệ định hình ngành **Platform Engineering** giai đoạn 2023–2026: **OpenTofu** (nhánh mã nguồn mở dưới quỹ Linux Foundation tương thích HCL với Terraform) và **Policy as Code** (OPA/Rego, Checkov)—biến mọi quy định bảo mật và ngân sách thành các bài kiểm thử tự động chạy ngay trên đầu ra `plan` trước khi một tài nguyên đám mây nào kịp được tạo ra.

## Bản đồ kiến thức nền: Bốn trụ cột của Hạ tầng mở (OpenTofu) & Chính sách dưới dạng mã

1. **HCL khai báo & Quản trị mã nguồn mở (OpenTofu vs. Terraform)**:
   Sau khi Terraform chuyển sang giấy phép BSL (2023), cộng đồng đã tạo ra **OpenTofu** (thuộc CNCF/Linux Foundation, giấy phép MPL-2.0) giữ nguyên cú pháp HCL, tương thích rộng rãi với hệ sinh thái provider và bổ sung các tính năng bảo mật được mong đợi như **mã hóa state phía client (state encryption)**.

2. **Chính sách dưới dạng mã (Policy as Code — OPA / Rego / Checkov)**:
   Thay vì viết một bản tài liệu PDF dài 40 trang nhắc nhở *"cấm mở S3 public"*, đội bảo mật viết các quy tắc thực thi tự động bằng **Rego (Open Policy Agent)** hoặc **Checkov**. Quy tắc đọc trực tiếp tệp kế hoạch JSON (`tofu show -json tfplan`)—nơi mọi biến và biểu thức đã được giải quyết xong—để quyết định `ALLOW` hoặc `DENY` ngay trên CI.

3. **Hai tầng kiểm tra: Shift-Left (Static/Plan) vs. Runtime (Admission)**:
   Kiểm tra trên `tofu plan` (**Shift-Left**) chặn lỗi từ lúc mở Pull Request—rẻ nhất và nhanh nhất. Kiểm tra ở tầng **Runtime** (Kubernetes Admission / Cloud SCP) đóng vai trò lưới an toàn cuối cùng phòng trường hợp ai đó bấm tay trên Console.

4. **Platform Engineering & "Đường ray vàng" (Golden Paths)**:
   Thay vì bắt mọi đội ứng dụng tự viết 500 dòng HCL mạng và bảo mật từ đầu, đội nền tảng đóng gói sẵn các **Module IaC chuẩn hóa** (đã tích hợp sẵn mã hóa, tag FinOps, log và quyền tối thiểu) như một sản phẩm nội bộ tự phục vụ (*Internal Developer Platform*).

## Mục tiêu học tập

- Trình bày sự kiện ngày 10/8/2023 và bối cảnh ra đời của OpenTofu 1.6 (tháng 1/2024).
- Viết quy tắc kiểm tra kiểu OPA/Conftest để ngăn lệnh `apply` vô tình tạo kho lưu trữ công khai.
- Hiểu vai trò của kho module chuẩn hóa do nhóm nền tảng cung cấp dưới góc nhìn kiến trúc dịch vụ nội bộ.

## 1. Hai công cụ CLI, cùng một nền tảng HCL

Ngày 10/8/2023, HashiCorp chuyển Terraform từ phiên bản 1.6 trở đi sang giấy phép **Business Source License (BSL) 1.1** (không thuộc nhóm giấy phép nguồn mở được OSI phê duyệt). Một liên minh doanh nghiệp và cộng đồng đã tạo nhánh từ phiên bản 1.5.x, công bố tuyên ngôn OpenTF và thành lập dự án **OpenTofu** dưới sự bảo trợ của Linux Foundation (gia nhập tháng 9/2023; phát hành bản **1.6.0 ổn định ngày 10/1/2024**). Trong sử dụng hằng ngày, lệnh `tofu` tương thích phần lớn với cấu hình HCL và các provider sẵn có, đồng thời bổ sung một số tính năng riêng như mã hóa tệp state tích hợp sẵn. Thương vụ IBM mua lại HashiCorp (giai đoạn 2024–2025) cũng là một mốc quan trọng về quản trị hệ sinh thái mà sinh viên cần nắm.

**Định hướng học tập:** Trọng tâm là nắm vững *nguyên lý cốt lõi* (state, plan, provider). Trong bài lab, bạn có thể dùng `terraform` hoặc `tofu` đều phù hợp; còn trong doanh nghiệp, bộ phận pháp chế và nền tảng sẽ quyết định công cụ chính thức.

```hcl
resource "aws_s3_bucket" "raw" {
  bucket = "iuh-lab-raw-${var.env}"
}
```

## 2. Chính sách dưới dạng mã: lớp kiểm duyệt tự động cho hạ tầng

Viết hạ tầng dưới dạng mã không tự động đảm bảo hạ tầng đó *an toàn*. Các công cụ chính sách như Open Policy Agent (OPA), Sentinel hoặc Checkov cho phép kiểm tra tự động bản kế hoạch thay đổi (**JSON plan**) trước khi chạy `apply`.

```rego
package terraform.s3

deny[msg] {
  some rc
  rc := input.resource_changes[_]
  rc.type == "aws_s3_bucket_public_access_block"
  rc.change.after.block_public_acls == false
  msg := sprintf("%s cho phép ACL công cộng", [rc.address])
}
```

Cơ chế này tương đồng với *admission controller* trong Kubernetes: lập trình viên khai báo hạ tầng mong muốn, còn nền tảng tự động thực thi các ràng buộc về vùng lưu trữ dữ liệu, gắn thẻ chi phí (FinOps) và mã hóa.

## 3. Platform engineering: cung cấp IaC như sản phẩm nội bộ

Từ năm 2024 trở đi, nhiều tổ chức hạn chế cấp tài khoản đám mây thô cho từng nhóm ứng dụng. Thay vào đó, **nhóm kỹ thuật nền tảng (platform team)** xây dựng và cung cấp:

- Các module chuẩn hóa có quản lý phiên bản (golden modules, ví dụ `eks-cluster/v4`, `postgres/v2`).
- Lộ trình phát triển tiêu chuẩn (paved road) kết hợp cổng nội bộ (như Backstage), GitOps và kiểm tra OPA.
- Cam kết SLO nội bộ, chẳng hạn thời gian tạo mới một môi trường xem trước (preview environment).

Khảo sát CNCF 2024 ghi nhận sự gia tăng mạnh của CI/CD và GitOps: IaC đảm nhận phần *định nghĩa hạ tầng*, Git là *kênh phân phối cấu hình*, còn cụm điều phối (Chương 09) là *môi trường chạy*. Kiến trúc đa đám mây (Chương 12) lúc này được trừu tượng hóa thành các provider khác nhau đứng sau cùng một giao diện module, thay vì sao chép rời rạc nhiều kho mã nguồn.

```mermaid
graph LR
  D[Repo ứng dụng] -->|PR| P[Plan + OPA]
  P -->|GitOps| T[tofu/terraform apply]
  T --> K[Kubernetes / API đám mây]
```

<div class="content-box insight-box">
<p><strong>Quản lý state vẫn là bài toán cốt lõi.</strong> Dù tệp state được OpenTofu mã hóa hay lưu trên HCP Terraform, rủi ro cơ bản trong phần lý thuyết vẫn giữ nguyên: hai tiến trình ghi đồng thời vào cùng một tệp state mà không có cơ chế khóa (state locking) sẽ gây xung đột và sai lệch trạng thái (split-brain).</p>
</div>

### Khung giải thích (nên đọc chậm): Trace một tệp `tfplan.json` đi qua cổng Policy as Code (OPA/Rego)

Hãy hình dung một sinh viên mở Pull Request thêm 2 tài nguyên vào tệp `main.tf`: một bucket lưu trữ `student_uploads` và một máy ảo `analytics_vm`. Trên CI, pipeline chạy `tofu plan -out=tfplan && tofu show -json tfplan > plan.json` rồi đưa `plan.json` qua 3 luật OPA/Rego.

Giả sử đoạn trích `resource_changes` trong `plan.json` sau khi giải biến có dạng:
```json
[
  {
    "address": "aws_s3_bucket_acl.uploads",
    "change": { "after": { "acl": "public-read" } }
  },
  {
    "address": "aws_instance.analytics_vm",
    "change": { "after": { "instance_type": "m5.4xlarge", "tags": { "env": "dev" } } }
  }
]
```

Theo dõi kết quả đánh giá của từng luật kiểm duyệt tự động:

| Luật Policy as Code | Điều kiện vi phạm (Rego) | Kết quả trên `plan.json` | Hành động của CI |
| :--- | :--- | :---: | :--- |
| **Luật 1 (Bảo mật)**: Cấm bucket công khai | `r.type == "aws_s3_bucket_acl"` và `acl == "public-read"` | **VI PHẠM** (`uploads`) | Trả `DENY`: *"Bucket không được phép đặt `public-read`"* |
| **Luật 2 (FinOps)**: Môi trường `dev` chỉ dùng `t3.micro`/`t3.small` | `tags.env == "dev"` và `instance_type` không thuộc danh sách cho phép | **VI PHẠM** (`analytics_vm`: `m5.4xlarge` $\approx \$560/\text{tháng}$!) | Trả `DENY`: *"Môi trường dev cấm dùng máy chủ vượt `t3.small`"* |
| **Luật 3 (Quản trị)**: Bắt buộc có nhãn `owner` | Thiếu khóa `tags.owner` | **VI PHẠM** (`analytics_vm`) | Trả `DENY`: *"Mọi tài nguyên phải gắn nhãn `owner`"* |

**Kết quả cuối cùng**: Pipeline CI lập tức báo đỏ (`3 violations found`), khóa nút **Merge** và in hướng dẫn sửa cụ thể ngay dưới Pull Request—**trước khi lệnh `tofu apply` kịp chạm vào đám mây**! Không tốn một xu nào và không có dữ liệu nào bị lộ ra Internet.

## Thách thức

- Nguy cơ phân mảnh kho provider, registry và lockfile khi hai hệ sinh thái tiến hóa độc lập.
- Bài toán cân bằng giữa việc cố định phiên bản module để ổn định với nhu cầu cập nhật nhanh bản vá bảo mật.
- Bộ quy tắc chính sách quá cứng nhắc có thể vô tình chặn cả những bài thử nghiệm hợp lệ trong môi trường học tập.

## Video tham khảo

- **Xem trước:** [What is OpenTofu Project?](https://www.youtube.com/watch?v=J2JKxY2elSo) — Oleksiy Pototskyy (tiếng Anh). Giới thiệu dự án OpenTofu và mối quan hệ với Terraform.
- [OPA Rego Tutorial: Write and Test Policy as Code](https://www.youtube.com/watch?v=x_IfmWXlHbI) — Spacelift (tiếng Anh). Hướng dẫn viết và kiểm thử chính sách bằng ngôn ngữ Rego cho mục Policy as Code.
- [What is a Platform Engineer?](https://www.youtube.com/watch?v=q6vbxk3hq-o) — HashiCorp, an IBM Company (tiếng Anh). Giải thích vai trò của kỹ sư nền tảng và cách cung cấp hạ tầng như sản phẩm nội bộ.

## Bài tập

1. Viết một quy tắc chính sách OPA yêu cầu mọi tài nguyên hạ tầng đều phải khai báo đầy đủ hai thẻ `tags.owner` và `tags.cost_center`.
2. Viết một đoạn ngắn phân tích vì sao sự khác biệt giữa giấy phép BSL và MPL tác động lớn đến các *nhà cung cấp dịch vụ SaaS* xây dựng sản phẩm quanh IaC, trong khi ít ảnh hưởng đến sinh viên chỉ chạy `apply` tạo một máy ảo thử nghiệm.
3. Thiết kế giao diện (biến đầu vào, giá trị đầu ra và giới hạn trần chi phí) cho một module chuẩn hóa phục vụ "môi trường preview cho đồ án sinh viên".

## Tài liệu tham khảo

1. Dự án OpenTofu: [opentofu.org](https://opentofu.org/).
2. Thông báo chuyển giấy phép BSL của HashiCorp (10/8/2023) và ghi chú phát hành OpenTofu 1.6.0 (10/1/2024).
3. Open Policy Agent: [openpolicyagent.org](https://www.openpolicyagent.org/).
4. CNCF *Cloud Native 2024 Annual Survey* (xu hướng CI/CD và GitOps).
5. Bài Terraform bắt buộc trong chương — các khái niệm provider, resource và state vẫn là kiến thức tiên quyết.
