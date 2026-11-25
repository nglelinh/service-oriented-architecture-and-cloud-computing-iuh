---
layout: post
title: 14 Giới thiệu — bài ứng dụng tùy chọn
chapter: '14'
order: 1
owner: Nguyen Le Linh
lang: vi
categories:
- chapter14
---

Hãy tưởng tượng bạn vừa mất ba ngày bấm chuột miệt mài trên giao diện web đám mây (ClickOps) để dựng xong hệ thống VPC, Subnet, Firewall, Load Balancer và Database cho môi trường phát triển. Sang tuần sau, sếp bảo: *"Tuyệt vời, giờ dựng thêm một bản y hệt như vậy cho môi trường Staging và một bản nữa cho Production nhé!"* — và bạn nhận ra mình không còn nhớ nổi đã tích chọn ô nào trong 40 màn hình cấu hình hôm trước.

**Hạ tầng dưới dạng mã (Infrastructure as Code — IaC)** chấm dứt hoàn toàn kỷ nguyên ClickOps. Toàn bộ máy chủ, mạng ảo, tường lửa và quyền truy cập được mô tả bằng các tệp mã nguồn văn bản có kiểm soát phiên bản (Git), duyệt qua Pull Request, kiểm thử tự động và tái tạo chính xác 100% chỉ bằng một câu lệnh.

## Kiến thức cần nhớ: Bốn nguyên lý cốt lõi của Infrastructure as Code

1. **Khai báo (Declarative) vs. Mệnh lệnh (Imperative)**:
   Script Bash/Python kiểu **mệnh lệnh** mô tả *từng bước làm* (`tạo VM → gắn IP → mở cổng 80`); chạy lại lần 2 thường báo lỗi "đã tồn tại". Công cụ **khai báo** (OpenTofu/Terraform) chỉ mô tả *trạng thái đích mong muốn*; công cụ tự so sánh với thực tế và tính ra danh sách thay đổi tối thiểu cần thiết.

2. **Tệp Trạng thái (State File) & Khóa trạng thái (State Locking)**:
   Tệp `terraform.tfstate` là cầu nối ánh xạ giữa tên tài nguyên trong mã HCL và ID thực tế trên đám mây (ví dụ `aws_vpc.main ↔ vpc-0a1b2c3d`). Trong làm việc nhóm, state phải đặt ở **Remote Backend** (như S3/GCS có mã hóa và cơ chế khóa) để tránh hai người chạy `apply` cùng lúc làm hỏng trạng thái chung.

3. **Đồ thị phụ thuộc có hướng (Resource DAG)**:
   Bạn không cần viết thứ tự chạy thủ công: khi `aws_subnet.web` tham chiếu `aws_vpc.main.id`, công cụ IaC tự dựng một **đồ thị phụ thuộc (DAG)** để tạo VPC trước, tạo song song các Subnet độc lập sau, và khi `destroy` thì xóa theo đúng thứ tự ngược lại.

4. **Lệch cấu hình (Configuration Drift)**:
   **Drift** xảy ra khi ai đó đăng nhập thẳng vào Web Console và bấm tay sửa một luật tường lửa mà không cập nhật mã IaC. Lệnh `plan` sẽ phát hiện sự sai lệch giữa mã, state và hạ tầng thật để kéo mọi thứ về đồng bộ.

### Giải thích: Trace lệnh `tofu plan` khi xảy ra Drift và phân biệt `~` (sửa tại chỗ) với `-/+` (xóa và tạo lại!)

Giả sử bạn đã chạy `tofu apply` tạo ra 1 Security Group (chỉ mở cổng `443`) và 1 máy ảo `web_server` nằm ở vùng `ap-southeast-1a`. Tối hôm đó, hai việc xảy ra:
- Đồng nghiệp bấm tay trên Console mở thêm cổng `22` vào Security Group (**Drift**).
- Bạn sửa trong file `main.tf` đổi `availability_zone` của `web_server` từ `ap-southeast-1a` sang `ap-southeast-1b`.

Khi chạy `tofu plan`, công cụ đọc lại hạ tầng thực tế (*refresh*) và in ra hai ký hiệu cực kỳ quan trọng mà bạn **phải luôn đọc kỹ trước khi gõ `yes`**:

| Tài nguyên trong `tofu plan` | Ký hiệu diff | Ý nghĩa hành động | Mức độ rủi ro downtime |
| :--- | :---: | :--- | :---: |
| `aws_security_group.web_sg` (gỡ cổng `22` bị mở tay) | **`~` (update in-place)** | Sửa trực tiếp luật tường lửa trên đối tượng hiện có | **Thấp / 0 downtime** (đóng lỗ hổng Drift) |
| `aws_instance.web_server` (đổi `availability_zone`) | **`-/+` (destroy and then create replacement)** | **XÓA BỎ** máy ảo cũ rồi mới tạo máy ảo mới ở AZ mới! | **CỰC CAO** (mất dữ liệu đĩa tạm & gián đoạn kết nối nếu không có `create_before_destroy`!) |

Quy tắc quan trọng khi dùng IaC: **không bao giờ bấm `apply` khi chưa đếm số lượng dấu `-` (destroy) và `-/+` (replace) trong bản `plan`**—đặc biệt là với cơ sở dữ liệu sản xuất!

> **Lưu ý lộ trình:** Phần lý thuyết bắt buộc của **Chương 14 (Hạ tầng dưới dạng mã — Infrastructure as Code)** hiện được trình bày bằng tiếng Anh. Lộ trình tiếng Việt ở chương này bổ sung **một bài ứng dụng tùy chọn**.
> 
> - [14-02 OpenTofu, chính sách dưới dạng mã và platform engineering]({{ site.baseurl }}{% multilang_post_url contents/chapter14/21-01-01-14_02_OpenTofu_and_Policy_as_Code %})
> - Bài Terraform ở bản tiếng Anh có bổ sung mục so sánh khái niệm với **CDK và Pulumi**.
> - [Lộ trình triển khai]({{ site.baseurl }}/contents/vi/chapter15/)
> - [Bản tiếng Anh của chương]({{ site.baseurl }}/contents/en/chapter14/)

## Video tham khảo

- **Xem trước:** [Infrastructure as Code là gì ? Tại sao sử dụng Terraform ? \| Lộ trình DevOps \| TechMaster](https://www.youtube.com/watch?v=u1foX-SAWsk) — TechMaster Vietnam (tiếng Việt). Giới thiệu khái niệm Infrastructure as Code và lý do sử dụng Terraform bằng tiếng Việt.
- [Introduction to HashiCorp Terraform with Armon Dadgar](https://www.youtube.com/watch?v=h970ZBgKINg) — HashiCorp, an IBM Company (tiếng Anh). Armon Dadgar giải thích mục tiêu và cách hoạt động của Terraform.
- [What is OpenTofu Project?](https://www.youtube.com/watch?v=J2JKxY2elSo) — Oleksiy Pototskyy (tiếng Anh). Giới thiệu dự án OpenTofu — trọng tâm của bài tùy chọn trong chương này.
