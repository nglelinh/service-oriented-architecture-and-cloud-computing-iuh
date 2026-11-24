---
layout: post
title: 08 Giới thiệu — bài ứng dụng tùy chọn
chapter: '08'
order: 1
owner: Nguyen Le Linh
lang: vi
categories:
- chapter08
---

Nếu từng nghe câu nói kinh điển *"Trên máy em chạy bình thường mà thầy!"*, bạn đã chạm vào nỗi đau lớn nhất từng hành hạ ngành phần mềm suốt hàng thập kỷ. Sự khác biệt nhỏ giữa phiên bản Python, thư viện C (`glibc`), biến môi trường hay cấu hình hệ điều hành giữa máy sinh viên và máy chủ sản xuất có thể đánh sập cả một hệ thống ngay giờ cao điểm. **Ảo hóa (Virtualization)** và đặc biệt là **Container** ra đời để đóng gói trọn vẹn môi trường thực thi thành một đơn vị bất biến, di động từ laptop lên bất kỳ đám mây nào.

## Bản đồ kiến thức nền: Ảo hóa và Container đứng trên những trụ cột hệ điều hành nào?

Để hiểu sâu sự khác biệt giữa máy ảo (VM) và container thay vì chỉ thuộc lòng vài câu lệnh `docker run`, bạn cần nắm bốn trụ cột cốt lõi:

1. **Hypervisor & Ảo hóa phần cứng (VM)**:
   Một máy ảo chạy trọn vẹn một **hệ điều hành khách (Guest OS)** cùng nhân (kernel) riêng bên trên lớp Hypervisor (KVM, Xen, VMware ESXi). Ưu điểm là cô lập bảo mật cực mạnh ở mức phần cứng (Intel VT-x / AMD-V), nhưng cái giá phải trả là hàng trăm MB–hàng GB RAM cho mỗi Guest OS và thời gian khởi động tính bằng chục giây.

2. **Linux Namespaces & cgroups (Trái tim của Container)**:
   Container **không** phải là "máy ảo thu nhỏ". Về bản chất, một container chỉ là một tiến trình Linux bình thường được nhân hệ điều hành chủ cô lập tầm nhìn bằng **Namespaces** (`pid`, `net`, `mnt`, `uts`, `user`) và giới hạn tài nguyên phần cứng bằng **cgroups** (CPU quota, giới hạn RAM, băng thông I/O). Nhờ dùng chung nhân Linux của máy chủ, container khởi động chỉ trong vài trăm mili-giây.

3. **Hệ thống tệp phân lớp (OverlayFS) & Chuẩn OCI**:
   Một ảnh container (OCI Image) không phải là một file khổng lồ nguyên khối, mà là chồng các **lớp chỉ đọc (read-only layers)** ghép chồng lên nhau bằng cơ chế *copy-on-write*. Mười container cùng dùng nền `alpine` hay `python:3.12-slim` sẽ dùng chung đúng một bản lưu trên đĩa và bộ nhớ đệm trang (page cache), chỉ tạo thêm một lớp ghi mỏng riêng cho từng container.

4. **Hạ tầng bất biến (Immutable Infrastructure)**:
   Thay vì SSH vào máy chủ để cài vá từng gói (dễ gây *configuration drift*), kiến trúc đám mây hiện đại xây dựng lại ảnh container mới cho mỗi bản sửa lỗi, kiểm thử đúng ảnh đó và thay thế toàn bộ container cũ.

### Khung giải thích (nên đọc chậm): Tính tay bài toán gom 20 microservice lên máy chủ 16 GB RAM — VM hay Container?

Giả sử nhóm bạn cần triển khai **20 microservice** nhẹ (mỗi dịch vụ thực tế chỉ cần `250 MB` RAM cho mã ứng dụng và `120 MB` tệp ứng dụng) lên một máy chủ vật lý có **16 GB RAM** (`16.384 MB`) và nhân Linux chủ tiêu tốn sẵn `1.024 MB` RAM. Hãy đặt lên bàn cân hai phương án bằng con số cụ thể:

| Thông số tính toán | Phương án A: 20 Máy ảo (VM) | Phương án B: 20 Container (Docker/OCI) |
| :--- | :---: | :---: |
| Chi phí hệ điều hành khách / tiến trình nền mỗi đơn vị | `700 MB` RAM (Guest OS kernel + systemd) | `15 MB` RAM (runtime shim, dùng chung Host OS kernel) |
| RAM cho mã ứng dụng mỗi đơn vị | `250 MB` | `250 MB` |
| **Tổng RAM cho 1 đơn vị** | $700 + 250 = \mathbf{950\text{ MB}}$ | $15 + 250 = \mathbf{265\text{ MB}}$ |
| **Tổng RAM cho 20 đơn vị + Host** | $1.024 + 20 \times 950 = \mathbf{20.024\text{ MB}}$ ($\approx 19{,}6\text{ GB}$) | $1.024 + 20 \times 265 = \mathbf{6.324\text{ MB}}$ ($\approx 6{,}2\text{ GB}$) |
| Kết quả trên máy chủ 16 GB RAM | **Quá tải (OOM / không đủ chỗ)** — chỉ chứa tối đa 15 VM | **Chạy mượt**, còn dư gần **10 GB RAM** cho cache & đột biến tải |
| Thời gian khởi động 1 bản sao mới (scale-out) | $25\text{–}45\text{ giây}$ (boot kernel + OS) | $0{,}3\text{–}1{,}5\text{ giây}$ (fork tiến trình + mount OverlayFS) |

**Kiểm tra thêm dung lượng đĩa với OverlayFS**: Nếu cả 20 microservice đều dùng chung lớp nền `python:3.12-slim` nặng `150 MB` và mỗi dịch vụ có `30 MB` mã nguồn riêng:
- Không chia sẻ lớp (20 VM độc lập): $20 \times (150 + 30) = \mathbf{3.600\text{ MB}}$.
- Chia sẻ lớp nền OCI: $150 + 20 \times 30 = \mathbf{750\text{ MB}}$ (tiết kiệm gần **$5\times$** dung lượng ổ đĩa!).

Đó chính là lý do container trở thành viên gạch tiêu chuẩn của mọi nền tảng đám mây hiện đại.

> **Lưu ý lộ trình:** Phần lý thuyết bắt buộc của **Chương 08 (Ảo hóa và Container)** hiện được trình bày bằng tiếng Anh. Lộ trình tiếng Việt ở chương này bổ sung **một bài ứng dụng tùy chọn**.
> 
> - [08-05 Serverless, microVM và WebAssembly]({{ site.baseurl }}{% multilang_post_url contents/chapter08/21-01-01-08_05_Serverless_MicroVMs_and_WASM %})
> - Bài Docker ở bản tiếng Anh có bổ sung mục so sánh lộ trình **từ triển khai thủ công sang nền tảng được quản lý** (Docker Compose trên máy ảo so với dịch vụ container quản lý).
> - [Lộ trình triển khai]({{ site.baseurl }}/contents/vi/chapter15/)
> - [Bản tiếng Anh của chương]({{ site.baseurl }}/contents/en/chapter08/)

## Video tham khảo

- **Xem trước:** [Virtualization - Ảo hóa là gì? Hiểu rõ trong 5 phút \| TING3S](https://www.youtube.com/watch?v=tx5zNXp5jPA) — TING 3S (tiếng Việt). Giới thiệu nhanh khái niệm ảo hóa bằng tiếng Việt trước khi đọc phần lý thuyết hypervisor bằng tiếng Anh.
- [How AWS’s Firecracker virtual machines work \| Amazon Science](https://www.youtube.com/watch?v=BIRv2FnHJAg) — Amazon Science (tiếng Anh). Giải thích cơ chế hoạt động của microVM Firecracker trong bài tùy chọn.
- [Introduction to WebAssembly and WASI](https://www.youtube.com/watch?v=Of07IsplHPY) — Root Access (tiếng Anh). Giới thiệu WebAssembly và WASI — mô hình cô lập nhẹ đặt cạnh microVM trong bài.
