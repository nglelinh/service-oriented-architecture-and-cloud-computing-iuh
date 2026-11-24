---
layout: post
title: 09 Giới thiệu — bài ứng dụng tùy chọn
chapter: '09'
order: 1
owner: Nguyen Le Linh
lang: vi
categories:
- chapter09
---

Chạy 3 container bằng `docker compose up` trên một máy chủ đơn lẻ rất dễ chịu—cho đến 2 giờ sáng ngày Chủ Nhật khi máy chủ đó đột ngột tắt nguồn, ổ cứng đầy, hoặc lượng truy cập tăng gấp 10 lần khiến toàn bộ dịch vụ treo cứng. Ai sẽ tự động khởi động lại container bị chết trên một máy khác? Ai sẽ cập nhật phiên bản mới mà không làm rớt kết nối của người dùng đang thanh toán?

**Kubernetes (K8s)** ra đời để giải quyết chính bài toán điều phối (orchestration) quy mô cụm đó. Thay vì ra lệnh mệnh lệnh kiểu *"hãy chạy container X trên máy B"*, bạn chỉ cần khai báo **trạng thái mong muốn (desired state)** bằng YAML, và hệ thống tự động liên tục kéo trạng thái thực tế về khớp với bản khai báo đó.

## Bản đồ kiến thức nền: Bốn nguyên lý làm nên bộ máy tự phục hồi của Kubernetes

Để đọc hiểu mọi tài liệu Kubernetes mà không lạc lối giữa hàng chục loại tài nguyên, hãy nắm bốn trục cốt lõi:

1. **Vòng lặp hòa giải (Reconciliation Loop / Control Loop)**:
   Mọi Controller trong Kubernetes hoạt động hệt như bộ điều nhiệt của máy lạnh: liên tục quan sát **Trạng thái hiện tại (Current State)**, so sánh với **Trạng thái mong muốn (Desired State)** lưu trong kho khóa–giá trị phân tán **etcd**, rồi thực hiện hành động điều chỉnh nếu hai trạng thái lệch nhau.

2. **Kiến trúc Control Plane vs. Worker Nodes**:
   **Control Plane** (`kube-apiserver`, `etcd`, `kube-scheduler`, `kube-controller-manager`) đóng vai trò bộ não ra quyết định và lưu trữ nguồn sự thật duy nhất. Các **Worker Node** (`kubelet`, `kube-proxy`, container runtime như `containerd`) là nơi thực sự chạy các **Pod** của ứng dụng.

3. **Pod là đơn vị lập lịch nhỏ nhất (và mang tính phù du — ephemeral)**:
   Kubernetes không chạy trực tiếp container trần mà bọc một (hoặc vài container dùng chung mạng/volume) trong một **Pod**. Mỗi Pod có một IP nội bộ riêng, nhưng IP đó sẽ **thay đổi** mỗi khi Pod bị xóa và tạo lại—vì vậy tuyệt đối không bao giờ gắn cứng IP Pod vào mã nguồn.

4. **Service & Deployment (Ổn định địa chỉ và vòng đời)**:
   **Deployment** quản lý `ReplicaSet` để giữ đúng số bản sao Pod và hỗ trợ cập nhật cuốn chiếu (*rolling update*) không gián đoạn. **Service** cung cấp một tên DNS nội bộ cố định (ví dụ `orders.default.svc.cluster.local`) cùng IP ảo ổn định để cân bằng tải động tới các Pod đang khỏe mạnh (vượt qua `readinessProbe`).

### Khung giải thích (nên đọc chậm): Trace vòng lặp hòa giải khi 1 Node chết trên cụm 3 Node (`replicas: 4`)

Giả sử bạn triển khai ứng dụng `api-web` với `replicas: 4` trên một cụm gồm 3 Worker Node (`Node-1`, `Node-2`, `Node-3`). Trạng thái ban đầu đang khỏe mạnh:
- `Node-1`: `Pod-A`, `Pod-B`
- `Node-2`: `Pod-C`
- `Node-3`: `Pod-D`
- `Service api-web` trỏ bảng Endpoint tới 4 IP: `[10.0.1.2, 10.0.1.3, 10.0.2.4, 10.0.3.5]`.

Bây giờ giả sử lúc $t = 0\text{ s}$, **`Node-1` mất điện đột ngột**. Hãy theo dõi từng giây Kubernetes tự phục hồi mà không cần con người can thiệp:

| Thời điểm | Thành phần hành động | Trạng thái thực tế vs. Mong muốn (`4`) | Hệ quả tức thì |
| :---: | :--- | :--- | :--- |
| $t = 0\text{ s}$ | `Node-1` ngừng gửi *heartbeat* | Thực tế: `2` Pod sống (`C`, `D`) nhưng Control Plane chưa biết | `kube-proxy` vẫn tạm thời có thể gửi 50% lưu lượng vào IP chết cho tới khi probe thất bại |
| $t = 40\text{ s}$ | `node-controller` đánh dấu `Node-1` là `NotReady` | `Pod-A`, `Pod-B` được gắn cờ mất kết nối | `EndpointSlice` loại bỏ `10.0.1.2` và `10.0.1.3`; **100% lưu lượng lập tức dồn sang `Pod-C`, `Pod-D`** |
| $t = 45\text{ s}$ | `ReplicaSet Controller` phát hiện lệch | Hiện tại `2` $\neq$ Mong muốn `4` (thiếu `2`) | Tạo 2 đối tượng Pod mới (`Pod-E`, `Pod-F`) ở trạng thái `Pending` |
| $t = 46\text{ s}$ | `kube-scheduler` chọn Node còn dư CPU/RAM | Gán `Pod-E → Node-2`, `Pod-F → Node-3` | `kubelet` trên `Node-2` và `Node-3` kéo ảnh (đã cache sẵn) và khởi chạy container |
| $t = 50\text{ s}$ | `readinessProbe` của `Pod-E`, `Pod-F` trả về `200 OK` | Hiện tại `4` $=$ Mong muốn `4` (**Hòa giải xong!**) | `Service api-web` cập nhật 4 IP sống mới; cụm trở lại đủ 100% công suất |

Hiểu đúng vòng lặp hòa giải này giúp bạn giải thích được mọi hành vi kỳ diệu—cũng như nguyên nhân lỗi vòng lặp `CrashLoopBackOff`—khi vận hành Kubernetes thực tế.

> **Lưu ý lộ trình:** Phần lý thuyết bắt buộc của **Chương 09 (Kubernetes)** hiện được trình bày bằng tiếng Anh. Lộ trình tiếng Việt ở chương này bổ sung **một bài ứng dụng tùy chọn**.
> 
> - [09-04 Service mesh, Gateway API và mạng eBPF]({{ site.baseurl }}{% multilang_post_url contents/chapter09/21-01-01-09_04_Service_Mesh_and_Gateway_API %})
> - Bài Services và Deployments ở bản tiếng Anh có bổ sung phần so sánh chiến lược cập nhật **rolling và canary**.
> - [Lộ trình triển khai]({{ site.baseurl }}/contents/vi/chapter15/)
> - [Bản tiếng Anh của chương]({{ site.baseurl }}/contents/en/chapter09/)

## Video tham khảo

- **Xem trước:** [Kubernetes là gì \| giải thích siêu dễ hiểu bằng hình ảnh trong 10 phút](https://www.youtube.com/watch?v=fbsBWQHxcpw) — Nguyen Duc Hoang (tiếng Việt). Giới thiệu trực quan các khái niệm Kubernetes bằng tiếng Việt trước khi đọc phần lý thuyết tiếng Anh.
- [Istio & Service Mesh - simply explained in 15 mins](https://www.youtube.com/watch?v=16fgzklcF7Y) — TechWorld with Nana (tiếng Anh). Giải thích hoạt động của Istio và service mesh trong khoảng 15 phút.
- [What is Cilium?](https://www.youtube.com/watch?v=ELyib78vjRY) — eBPF & Cilium Community (tiếng Anh). Giới thiệu Cilium — ví dụ tiêu biểu cho hạ tầng mạng dựa trên eBPF trong bài tùy chọn.
