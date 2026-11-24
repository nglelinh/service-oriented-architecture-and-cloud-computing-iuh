---
layout: post
title: 08-05 Serverless, microVM và WebAssembly (2022–2026)
chapter: '08'
order: 6
owner: Nguyen Le Linh
lang: vi
categories:
- chapter08
lesson_type: optional
---

Khi một nhà cung cấp đám mây như AWS phải chạy mã nguồn chưa được kiểm chứng của hàng triệu khách hàng khác nhau trên cùng một máy chủ vật lý, họ đối mặt với một bài toán hóc búa tưởng chừng vô nghiệm: **máy ảo truyền thống (VM)** đủ an toàn nhưng khởi động mất hàng chục giây; còn **container Linux** khởi động nhanh trong nửa giây nhưng lại dùng chung nhân hệ điều hành (kernel) nên một lỗ hổng nhân có thể đe dọa cả máy chủ. Để dung hòa cả hai thế giới, ngành đám mây đã phát minh ra hai lớp cô lập thế hệ mới: **microVM (Firecracker)** và **WebAssembly (WASM)**.

Các bài bắt buộc đã trình bày về hypervisor, namespace/cgroup trong Linux, Docker và AWS Lambda. Bài tùy chọn này theo dõi các bước tiến xung quanh những nền tảng đó giai đoạn 2022–2026: **snapshot microVM Firecracker** (Lambda SnapStart), hỗ trợ **hàm đóng gói bằng ảnh container**, và **WebAssembly (WASM)** như một lớp cô lập gọn nhẹ hơn — đồng thời lý giải vì sao khảo sát CNCF ghi nhận xu hướng áp dụng serverless có sự *phân hóa* thay vì tăng trưởng đồng đều.

> Lý thuyết bắt buộc hiện ở bản tiếng Anh. Đây là bài ứng dụng tùy chọn bằng tiếng Việt.

Khi AWS Lambda ra mắt và phải chạy mã nguồn không tin cậy của hàng triệu khách hàng khác nhau trên cùng một máy chủ vật lý, các kỹ sư hạ tầng vấp phải một bài toán tưởng chừng bất khả thi: **máy ảo truyền thống (VM)** cách ly an toàn tuyệt đối nhưng khởi động mất 20 giây và ngốn nửa gigabyte RAM; còn **container (Docker)** khởi động nhanh nhưng lại dùng chung nhân Linux (shared kernel) — chỉ một lỗ hổng nhân là khách hàng A có thể đọc trộm bộ nhớ của khách hàng B!

Câu trả lời của ngành điện toán đám mây giai đoạn 2022–2026 là hai công nghệ đột phá: **microVM (AWS Firecracker)** mang ranh giới cách ly phần cứng với tốc độ khởi động chỉ ~125 ms, và **WebAssembly (WASM / WASI)** trên máy chủ biên (edge) khởi động trong vài mili-giây với chi phí bộ nhớ tính bằng kilobyte!

---

## Bản đồ kiến thức nền: Bốn tầng Cách ly và Mật độ Điện toán Đám mây

Để chọn đúng hạ tầng thực thi cho từng loại tải công việc (workload), hãy nhìn qua bốn nấc thang đánh đổi giữa **độ cách ly bảo mật** và **tốc độ khởi động / mật độ**:

1. **Máy ảo truyền thống (VM — KVM/QEMU):** Mô phỏng đầy đủ phần cứng ảo (BIOS, PCI, USB) + hệ điều hành đầy đủ $$\implies$$ cách ly mạnh nhưng chậm (10–30 s) và nặng RAM.
2. **Container (Docker / Linux Namespaces & cgroups):** Tách không gian tiến trình trên *cùng một nhân Linux* $$\implies$$ nhẹ và phổ biến cho microservice nội bộ, nhưng ranh giới cách ly yếu hơn cho mã đa khách thuê (*multi-tenant untrusted code*).
3. **MicroVM (Firecracker / Cloud Hypervisor):** Bóc sạch thiết bị ảo thừa của QEMU, giữ lại đúng lớp ảo hóa phần cứng KVM tối giản nhất $$\implies$$ đạt **cách ly cấp độ phần cứng** mà khởi động chỉ ~125 ms và chỉ tốn ~5 MB RAM nền!
4. **WebAssembly trên Máy chủ (WASM + WASI / Component Model):** Cách ly bằng hộp cát phần mềm (linear memory + capability-based security) $$\implies$$ khởi động lạnh **1–5 ms**, chạy gần tốc độ gốc và lý tưởng cho hàm biên (Edge Functions) và hệ thống plugin đa ngôn ngữ.

---

## Bản đồ kiến thức nền: Bốn trụ cột của công nghệ cô lập và Serverless hiện đại

1. **Quang phổ Cô lập (VM → microVM → Container → WebAssembly)**:
   Đặt bốn công nghệ trên cùng một trục đánh đổi giữa *ranh giới bảo mật* và *thời gian khởi động / tiêu tốn RAM*: microVM (Firecracker) cắt bỏ toàn bộ thiết bị ảo thừa (USB, BIOS cũ) để giữ cô lập phần cứng KVM nhưng boot chỉ ~125 ms; còn WASM bỏ luôn cả nhân Linux khách để khởi chạy trong vài mili-giây.
2. **Khởi động nguội (Cold Start) & Kỹ thuật Snapshot (Lambda SnapStart)**:
   Thay vì chạy lại toàn bộ quá trình nạp thư viện (JVM/Python import) mỗi khi có tải đột biến, nền tảng chạy trước pha khởi tạo một lần, chụp ảnh toàn bộ RAM và đĩa của microVM rồi nạp nhanh lại khi có yêu cầu.
3. **Tính duy nhất trạng thái sau khôi phục Snapshot**:
   Bản chụp bộ nhớ đóng băng cả hạt giống sinh số ngẫu nhiên (RNG), kết nối TCP cũ và mã định danh tạm — buộc lập trình viên phải phân biệt rạch ròi giữa *tài nguyên khởi tạo một lần (init)* và *dữ liệu phải sinh mới ở từng lần gọi (invoke)*.
4. **Kinh tế học Serverless (Pay-per-millisecond vs. Always-on Container)**:
   Serverless tối ưu chi phí cực tốt cho lưu lượng gián đoạn/đột biến (`scale-to-zero`), nhưng khi lưu lượng ổn định 24/7, một Pod/Container chạy thường trực thường rẻ hơn và có độ trễ p99 ổn định hơn.

## Mục tiêu học tập

- Giải thích cơ chế SnapStart như bản chụp trạng thái (snapshot) của một microVM Firecracker đã khởi tạo sẵn.
- Đặt WebAssembly/WASI cạnh container Linux để so sánh kích thước môi trường cô lập và giao diện lời gọi hệ thống.
- Lựa chọn hợp lý giữa FaaS, Knative và Kubernetes Deployment chạy thường trực cho từng ranh giới dịch vụ SOA.

## 1. SnapStart: giảm độ trễ khởi động nguội mà không phải viết lại ứng dụng

AWS ra mắt Lambda SnapStart cho **Java** (28/11/2022), mở rộng sang **Python và .NET** (phát hành rộng rãi năm 2024), và tiếp tục hỗ trợ hàm chạy từ **ảnh container** (AWS What’s New, tháng 7/2026). Theo tài liệu AWS, khi phiên bản hàm được xuất bản, Lambda khởi tạo môi trường thực thi, tạo **bản chụp (snapshot) đã mã hóa của bộ nhớ và đĩa trên microVM Firecracker**, rồi khôi phục nhanh từ bản chụp đó mỗi khi có lời gọi hoặc mở rộng tải.

```python
import uuid
import socket

def handler(event, context):
    node_id = str(uuid.uuid4())  # sinh mỗi invoke, không phải mỗi init
    return {"id": node_id, "host": socket.gethostname()}
```

Về bản chất, đây vẫn là **ảo hóa phần cứng** (microVM) chứ không chỉ là cô lập bằng namespace như container thuần. Vấn đề lập trình viên cần đặc biệt lưu ý là *thời điểm* sinh giá trị duy nhất, khóa mật mã hoặc khởi tạo kết nối mạng để tránh trùng lặp trạng thái khi khôi phục từ cùng một snapshot.

## 2. WebAssembly (WASM): cô lập nhẹ không cần nhân hệ điều hành khách

Các nền tảng như Fermyon **Spin**, Wasmtime và chuẩn WASI hướng tới thời gian khởi động ở mức mili-giây cùng cơ chế sandbox phân quyền theo năng lực (capability-based). So với Docker, ứng dụng chỉ cần đóng gói module biên dịch thay vì cả tầng userspace của hệ điều hành. Tuy nhiên, hệ sinh thái I/O của WASI, công cụ gỡ lỗi và mức độ phổ biến kỹ năng trong đội ngũ hiện chưa đồng đều như container Linux. Do đó, nên xem WASM là **lớp cô lập thứ ba** (bên cạnh VM và container), không phải sự thay thế tuyệt đối cho Docker.

Khảo sát CNCF 2024 (công bố năm 2025) cho thấy mức độ sử dụng serverless có sự **phân hóa**: một số tổ chức tiếp tục mở rộng, trong khi số khác thu hẹp vì bài toán chi phí và độ phức tạp vận hành. Điều này cũng phản ánh thực tế quen thuộc khi kết hợp Lambda với NAT Gateway, VPC và cold start đôi khi lại phức tạp và chậm hơn một Deployment nhỏ chạy ổn định.

## 3. Bảng đối chiếu mẫu ứng dụng

| Mẫu bài toán | Cơ chế phù hợp | Ví dụ thực tế |
| --- | --- | --- |
| API có lưu lượng đột biến | FaaS + SnapStart hoặc provisioned concurrency | Tiếp nhận webhook thanh toán |
| Dịch vụ HTTP cần `scale-to-zero` | Knative / Cloud Run | Công cụ quản trị nội bộ |
| Môi trường cô lập nhẹ không sidecar | WASM tại biên (edge) hoặc Spin | Xác thực tại tầng CDN |
| Tải GPU chạy liên tục | Không ưu tiên FaaS | Phục vụ vLLM trên Kubernetes |

<div class="content-box insight-box">
<p><strong>Góc nhìn SOA.</strong> Serverless là một lựa chọn về <em>cách đóng gói và tính phí</em>. Các hợp đồng dịch vụ cốt lõi như timeout, tính lũy đẳng (idempotency) và xác thực vẫn phải được thiết kế nghiêm ngặt tương tự microservice đóng gói trong container.</p>
</div>

## 4. Kiến trúc kết hợp: hướng sự kiện ở biên và gRPC ở lõi

Một mẫu kiến trúc phổ biến giai đoạn 2024–2026 thường kết hợp ba lớp:

1. API Gateway và hàm FaaS xử lý lưu lượng HTTP bắc–nam *biến động mạnh*.
2. Các worker gRPC chạy thường trực cho giao tiếp đông–tây *tần suất cao* (kết hợp service mesh hoặc ambient mode).
3. Lưu trữ đối tượng (object storage) và hàng đợi cho các dữ liệu kích thước lớn thay vì nhồi trực tiếp vào nội dung sự kiện.

Không nên đặt trực tiếp tác vụ sinh nội dung LLM kéo dài 30 giây bên trong giới hạn timeout của API Gateway mà không tách thành cơ chế xử lý bất đồng bộ kèm mã tác vụ (`job_id`).

### Khung giải thích (nên đọc chậm): Tính tay điểm hòa vốn chi phí giữa AWS Lambda và Container chạy thường trực

Giả sử bạn có một API xử lý webhook cần **512 MB RAM** (0,5 GB) và mất trung bình **100 ms** (`0,1 giây`) cho mỗi yêu cầu. Hãy so sánh chi phí tháng giữa **AWS Lambda** (đơn giá minh họa: `0,20 USD / 1 triệu request` + `0,00001667 USD / GB-giây`) và **1 Container nhỏ chạy 24/7** (`0,5 vCPU, 512 MB RAM`, chi phí cố định khoảng **15 USD/tháng**):

| Kịch bản lưu lượng (1 tháng = 30 ngày) | Tổng số request/tháng | Chi phí AWS Lambda ước tính | Chi phí 1 Container 24/7 | Lựa chọn kinh tế hơn |
| :--- | :---: | :---: | :---: | :---: |
| **Kịch bản A (Đột biến / ít dùng):** trung bình `0,5 RPS` | $$1{,}3\text{ triệu}$$ | $$0{,}26 + (1{,}3\times 10^6 \times 0{,}1 \times 0{,}5 \times 1{,}667\times 10^{-5}) \approx \mathbf{1{,}34\text{ USD}}$$ | **15,00 USD** | **Lambda rẻ hơn $$11\times$$!** |
| **Kịch bản B (Tải ổn định cao):** đều đặn `20 RPS` | $$51{,}8\text{ triệu}$$ | $$10{,}36 + (51{,}8\times 10^6 \times 0{,}1 \times 0{,}5 \times 1{,}667\times 10^{-5}) \approx \mathbf{53{,}5\text{ USD}}$$ | **15,00 USD** (hoặc 30 USD cho 2 bản sao HA) | **Container rẻ hơn ~$$2\text{–}3{,}5\times$$!** |

Bảng tính tay này giải thích hoàn toàn xu hướng **phân hóa serverless** trong khảo sát CNCF 2024: serverless cực kỳ lợi hại cho webhook thưa thớt, cron job và các điểm biên đột biến; nhưng khi lưu lượng ổn định ở mức hàng chục RPS trở lên, container chạy thường trực lại thắng cả về hóa đơn lẫn độ trễ p99 ổn định.

## Thách thức

- Lỗi mất tính duy nhất khi khôi phục snapshot trong SnapStart (chứng chỉ hết hạn, bộ sinh số ngẫu nhiên RNG, hoặc kết nối cũ trong pool).
- Rủi ro phụ thuộc nền tảng trong hệ sinh thái WASM do giao diện host API giữa các nhà cung cấp còn khác nhau.
- Bài toán chi phí: tính phí theo mili-giây có thể đắt hơn một Deployment 1 replica chạy liên tục khi lưu lượng QPS ổn định.

---

### Ví dụ thực tế cầm tay chỉ việc: Tính mật độ đóng gói trên một máy chủ vật lý 16 GB RAM

Giả sử bạn có một máy chủ vật lý dành **16 GB (16.384 MB) RAM** cho các thực thể chạy hàm serverless (mỗi hàm cần 10 MB RAM nghiệp vụ cộng thêm RAM nền của lớp ảo hóa):

| Công nghệ thực thi | Thời gian Cold Start | RAM nền / thực thể | Tổng RAM / thực thể | Số thực thể tối đa trên 16 GB |
| :--- | :--- | :--- | :--- | :--- |
| **Traditional VM** | $$15	ext{–}30	ext{ giây}$$ | $$500	ext{ MB}$$ | $$510	ext{ MB}$$ | **$$32$$ máy ảo** |
| **MicroVM (Firecracker)** | $$\sim 125	ext{ ms}$$ | $$5	ext{ MB}$$ | $$15	ext{ MB}$$ | **$$1{,}092$$ microVM** (gấp **34×**!) |
| **WASM Isolate (WASI)** | **$$1	ext{–}5	ext{ ms}$$** | **$$< 1	ext{ MB}$$** | $$11	ext{ MB}$$ | **$$1{,}489$$ isolate** (và cực nhanh!) |

Hãy nhìn cột Cold Start kết hợp với mật độ: với **microVM**, nhà cung cấp đám mây nhồi được hơn 1.000 hàm tách biệt hoàn toàn về phần cứng trên một máy chủ; còn với **WASM**, hàm khởi động nhanh hơn cả một vòng ping mạng (5 ms), xóa sổ gần như hoàn toàn nỗi ám ảnh "cold start" của serverless!

### Khung giải thích: Bốn quy tắc chọn giữa Container, MicroVM và WASM (nên đọc chậm)

1. **Chạy mã tin cậy nội bộ công ty (microservice chạy dài hạn):** Container trên Kubernetes vẫn là lựa chọn thực dụng nhất nhờ hệ sinh thái trưởng thành và đầy đủ thư viện hệ điều hành.
2. **Chạy mã khách hàng không tin cậy (FaaS, sandbox AI chạy code, CI runners):** Bắt buộc dùng **microVM (Firecracker / Kata Containers)** để bảo vệ nhân hệ điều hành máy chủ.
3. **Chạy logic nhẹ ở máy chủ biên toàn cầu (Edge routing, lọc API, plugin mở rộng):** **WASM/WASI** vượt trội tuyệt đối nhờ khởi động dưới 5 ms và tệp thực thi chỉ vài trăm KB không phụ thuộc hệ điều hành.
4. **Cẩn thận giới hạn của WASM hiện tại:** Các thư viện phụ thuộc sâu vào lời gọi hệ thống Linux (syscall đặc thù, trình điều khiển GPU CUDA) vẫn cần container hoặc microVM thay vì WASM thuần.

**Kiểm tra nhanh.** Tại sao AWS Lambda và Fly.io lại chọn **Firecracker microVM** thay vì chạy thẳng Docker container trên cùng một nhân Linux để thực thi mã của nhiều khách hàng khác nhau?

---

## Video tham khảo

- **Xem trước:** [How AWS’s Firecracker virtual machines work \| Amazon Science](https://www.youtube.com/watch?v=BIRv2FnHJAg) — Amazon Science (tiếng Anh). Trình bày kiến trúc microVM Firecracker đứng sau AWS Lambda.
- [Introducing AWS Lambda SnapStart \| Amazon Web Services](https://www.youtube.com/watch?v=AgxvrZLI1mc) — Amazon Web Services (tiếng Anh). Giới thiệu cơ chế tạo và khôi phục snapshot của Lambda SnapStart.
- [Introduction to WebAssembly and WASI](https://www.youtube.com/watch?v=Of07IsplHPY) — Root Access (tiếng Anh). Giải thích mô hình cô lập của WebAssembly và WASI.

## Bài tập

1. Liệt kê ba loại đối tượng được khởi tạo lúc init cần phải làm mới lại sau khi khôi phục từ snapshot SnapStart.
2. So sánh (có thể định tính) giữa chạy AWS Lambda cấu hình 128 MB và chạy một container 1 replica thường trực cho một API tốn CPU ở mức ổn định 20 yêu cầu/giây (RPS).
3. Phác thảo kịch bản nên dùng WASM tại tầng biên (edge) so với dùng microVM Firecracker trong vùng đám mây (region).

## Tài liệu tham khảo

1. AWS, “Improving startup performance with Lambda SnapStart”.
2. AWS News Blog, SnapStart cho Python và .NET GA.
3. AWS What’s New, SnapStart cho hàm ảnh container (7/2026).
4. Fermyon Spin: [fermyon.com/spin](https://www.fermyon.com/spin).
5. CNCF *Cloud Native 2024 Annual Survey* (xu hướng phân hóa serverless).
6. Firecracker: [firecracker-microvm.github.io](https://firecracker-microvm.github.io/).
