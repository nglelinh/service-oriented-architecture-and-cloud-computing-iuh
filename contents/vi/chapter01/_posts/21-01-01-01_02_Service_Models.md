---
layout: post
title: 01-02 Các mô hình dịch vụ đám mây
chapter: '01'
order: 3
owner: Nguyen Le Linh
lang: vi
categories:
- chapter01
lesson_type: required
---

Các dịch vụ điện toán đám mây thường được phân loại thành ba mô hình chính, mỗi mô hình mang lại mức độ kiểm soát, tính linh hoạt và phạm vi trách nhiệm quản trị khác nhau. Hiểu rõ sự khác biệt giữa ba mô hình này là cơ sở để lựa chọn chiến lược đám mây phù hợp với năng lực và nhu cầu của tổ chức.

## Ngăn xếp dịch vụ đám mây

Các mô hình dịch vụ có thể được hình dung như một ngăn xếp (stack) phân tầng, trong đó mỗi lớp được xây dựng dựa trên lớp ngay bên dưới:

```
┌─────────────────────────────────────┐
│        Software as a Service        │  ← SaaS
│              (SaaS)                 │
├─────────────────────────────────────┤
│       Platform as a Service         │  ← PaaS
│              (PaaS)                 │
├─────────────────────────────────────┤
│     Infrastructure as a Service     │  ← IaaS
│              (IaaS)                 │
├─────────────────────────────────────┤
│        Physical Infrastructure      │  ← On-Premises
└─────────────────────────────────────┘
```

## Hạ tầng như một dịch vụ (Infrastructure as a Service — IaaS)

IaaS là tầng nền tảng của điện toán đám mây, cung cấp tài nguyên tính toán ảo hóa qua Internet để doanh nghiệp thuê dùng linh hoạt thay vì phải mua sắm phần cứng.

### Định nghĩa và khái niệm cốt lõi

Ở mức cơ bản nhất, IaaS cung cấp các khối tài nguyên hạ tầng thiết yếu gồm máy ảo (virtual machine — VM), lưu trữ, mạng và hệ điều hành. Thay vì đầu tư máy chủ vật lý đặt tại trung tâm dữ liệu nội bộ (on-premises), tổ chức cấp phát tài nguyên theo mô hình trả tiền theo mức sử dụng từ nhà cung cấp đám mây. IaaS mang lại mức độ linh hoạt và quyền kiểm soát quản trị cao nhất, tương tự vận hành một trung tâm dữ liệu truyền thống nhưng trong môi trường ảo hóa.

### Các thành phần chính

Một môi trường IaaS tiêu chuẩn thường bao gồm ba nhóm thành phần chính. **Tài nguyên tính toán (compute)** là thành phần chủ lực, trải rộng từ máy ảo thông thường đến máy chủ vật lý chuyên dụng (bare metal) và các nền tảng thực thi theo sự kiện. **Dịch vụ lưu trữ** cung cấp nhiều phương án mở rộng cho từng loại dữ liệu: lưu trữ khối (block storage) cho cơ sở dữ liệu và lưu trữ đối tượng (object storage) cho dữ liệu phi cấu trúc như bản sao lưu hoặc tệp đa phương tiện. **Mạng** cho phép định nghĩa mạng ảo riêng biệt — bao gồm subnet, bảng định tuyến và tường lửa — tương đương cách cấu hình thiết bị mạng vật lý.

### Mô hình trách nhiệm

Trong mô hình IaaS, nhà cung cấp quản lý toàn bộ hạ tầng vật lý bên dưới gồm máy chủ vật lý, lớp ảo hóa (hypervisor), phần cứng lưu trữ và mạng vật lý. Khách hàng chịu trách nhiệm quản lý mọi lớp phía trên hypervisor: hệ điều hành, middleware, môi trường runtime, dữ liệu và mã ứng dụng, bao gồm cả việc cập nhật bản vá bảo mật và cấu hình tường lửa.

### Tình huống sử dụng

IaaS phù hợp khi tổ chức cần quyền kiểm soát sâu ở cấp hệ thống. Mô hình này rất thích hợp cho **môi trường phát triển và kiểm thử**, nơi các nhóm có thể khởi tạo môi trường tạm thời trong vài phút rồi thu hồi ngay sau khi dùng xong. Nó cũng hỗ trợ hiệu quả cho **phục hồi thảm họa (disaster recovery)** bằng cách nhân bản hạ tầng trọng yếu sang vùng địa lý khác mà không phải duy trì một trung tâm dữ liệu vật lý thứ hai. Ngoài ra, các bài toán **tính toán hiệu năng cao (High-Performance Computing — HPC)** đòi hỏi cấu hình phần cứng chuyên biệt cho mô phỏng khoa học hoặc mô hình tài chính cũng tận dụng tốt khả năng mở rộng của IaaS.

### Ưu và nhược điểm

Ưu điểm nổi bật của IaaS là **quyền kiểm soát cao**: bạn có thể tùy chỉnh môi trường sát theo yêu cầu kỹ thuật, tránh chi phí vốn ban đầu cho phần cứng và mở rộng quy mô nhanh chóng. Ngược lại, nhược điểm chính là **gánh nặng quản trị**: đội ngũ vận hành phải có đủ năng lực kỹ thuật để quản lý hệ điều hành, cập nhật bản vá bảo mật và duy trì cấu hình mạng.

## Nền tảng như một dịch vụ (Platform as a Service — PaaS)

PaaS giảm bớt gánh nặng vận hành hạ tầng bên dưới để đội ngũ phát triển tập trung vào thiết kế và xây dựng ứng dụng.

### Định nghĩa và khái niệm cốt lõi

PaaS cung cấp môi trường phát triển và triển khai hoàn chỉnh trên đám mây. Bên cạnh hạ tầng tính toán, lưu trữ và mạng, PaaS tích hợp sẵn middleware, công cụ phát triển, dịch vụ phân tích dữ liệu và hệ quản trị cơ sở dữ liệu. Mô hình này được thiết kế để hỗ trợ trọn vẹn vòng đời ứng dụng: xây dựng, kiểm thử, triển khai, vận hành và cập nhật.

### Các thành phần chính

Một nền tảng PaaS thường cung cấp sẵn **công cụ phát triển** và **môi trường runtime** cho nhiều ngôn ngữ phổ biến như Java, Python hay Node.js. Ngoài ra còn có các **dịch vụ cơ sở dữ liệu được quản lý** (cả SQL và NoSQL), **lớp bộ nhớ đệm (cache)** và **hàng đợi thông điệp (message queue)**, giúp lập trình viên không phải tự cài đặt và vận hành thủ công các thành phần phức tạp này. Nhiều giải pháp PaaS cũng tích hợp sẵn **quy trình CI/CD** và **tự động mở rộng (auto-scaling)** để ứng dụng thích ứng với biến động lưu lượng.

### Mô hình trách nhiệm

Ở mô hình PaaS, phần trách nhiệm chuyển sang nhà cung cấp mở rộng đáng kể: ngoài hạ tầng vật lý, nhà cung cấp còn quản lý hệ điều hành, middleware và môi trường runtime. Khách hàng chủ yếu tập trung quản lý **mã ứng dụng** và **dữ liệu**, thay vì dành thời gian bảo trì hay vá lỗi máy chủ.

### Tình huống sử dụng

PaaS là lựa chọn phù hợp cho **phát triển ứng dụng web và di động**, tạo điều kiện cho các nhóm phân tán phối hợp hiệu quả. Mô hình này cũng rất thuận tiện khi triển khai **API và kiến trúc microservice**, nơi các dịch vụ nhỏ, độc lập cần được phát hành và cập nhật thường xuyên.

### Ưu và nhược điểm

Lợi thế lớn nhất của PaaS là **tốc độ phát triển và triển khai**, giúp rút ngắn thời gian ra thị trường vì toàn bộ phần hạ tầng kết nối bên dưới đã được nhà cung cấp xử lý. Tuy nhiên, hạn chế chính là nguy cơ **phụ thuộc vào nhà cung cấp (vendor lock-in)** khi ứng dụng dựa nhiều vào công cụ hoặc API đặc thù của nền tảng, gây khó khăn khi chuyển đổi về sau. Đồng thời, quyền kiểm soát đối với môi trường hệ thống bên dưới cũng bị thu hẹp hơn so với IaaS.

## Phần mềm như một dịch vụ (Software as a Service — SaaS)

SaaS là mô hình gần gũi nhất với người dùng cuối, cung cấp trực tiếp các ứng dụng hoàn chỉnh qua Internet.

### Định nghĩa và khái niệm cốt lõi

SaaS cho phép người dùng kết nối và sử dụng phần mềm chạy trên đám mây thông qua trình duyệt web hoặc ứng dụng khách nhẹ, thường theo mô hình thuê bao (subscription). Các ví dụ quen thuộc gồm thư điện tử, lịch làm việc và bộ công cụ văn phòng trực tuyến. Toàn bộ ngăn xếp công nghệ — từ máy chủ vật lý đến mã nguồn ứng dụng — đều do nhà cung cấp vận hành.

### Đặc điểm chính

SaaS thường được xây dựng trên kiến trúc **đa thuê bao (multi-tenant)**: một phiên bản phần mềm phục vụ đồng thời nhiều tổ chức trong khi vẫn cô lập dữ liệu của từng khách hàng. Phần mềm được cung cấp theo hình thức **đăng ký định kỳ** (theo tháng hoặc năm) và **cập nhật tự động**, giúp người dùng luôn sử dụng phiên bản mới nhất mà không cần tự cài đặt bản vá.

### Mô hình trách nhiệm

Trong mô hình SaaS, trách nhiệm của khách hàng ở mức thấp nhất, chủ yếu xoay quanh **quản trị dữ liệu** và **phân quyền truy cập người dùng**. Nhà cung cấp đảm nhận toàn bộ phần mềm ứng dụng, cơ sở dữ liệu, bảo mật nền tảng, máy chủ và mạng.

### Các nhóm ứng dụng SaaS phổ biến

SaaS hiện diện trong hầu hết các mảng nghiệp vụ: **bộ công cụ năng suất** như Microsoft 365 và Google Workspace phục vụ cộng tác; **hệ thống quản lý quan hệ khách hàng (CRM)** như Salesforce; **hệ thống hoạch định nguồn lực doanh nghiệp (ERP)** như NetSuite; cũng như các bộ phần mềm sáng tạo chuyên biệt như Adobe Creative Cloud.

### Ưu và nhược điểm

SaaS loại bỏ hoàn toàn chi phí cài đặt, mua sắm phần cứng và bảo trì phần mềm, giúp tổ chức **triển khai nhanh** với **chi phí định kỳ dễ dự báo**. Đổi lại, khách hàng có mức **kiểm soát và khả năng tùy biến thấp nhất**: chức năng phụ thuộc vào lộ trình của nhà cung cấp, và mức độ an toàn dữ liệu gắn chặt với năng lực bảo mật của bên cung cấp dịch vụ.

## Lựa chọn mô hình dịch vụ phù hợp

Việc chọn mô hình dịch vụ thực chất là bài toán cân bằng giữa mức độ kiểm soát và sự thuận tiện trong vận hành.

### Khung ra quyết định

- Chọn **IaaS** khi cần kiểm soát chi tiết ở cấp hạ tầng, đang di chuyển ứng dụng kế thừa có yêu cầu hệ điều hành đặc thù, hoặc tổ chức đã có đội ngũ vận hành giàu kinh nghiệm.
- Chọn **PaaS** khi phát triển ứng dụng mới, ưu tiên tốc độ đưa sản phẩm ra thị trường và muốn tối giản công việc quản trị hệ thống.
- Chọn **SaaS** cho các nghiệp vụ chuẩn hóa (thư điện tử, CRM, nhân sự) nơi việc tự xây dựng phần mềm riêng không tạo ra lợi thế cạnh tranh.

Trên thực tế, nhiều tổ chức áp dụng **mô hình kết hợp**: dùng SaaS cho công cụ làm việc chung, PaaS cho các ứng dụng hướng khách hàng mới và IaaS cho những khối lượng công việc chuyên biệt cần tùy biến sâu.

### Ma trận so sánh

| Khía cạnh | IaaS | PaaS | SaaS |
|-----------|------|------|------|
| **Kiểm soát** | Cao | Trung bình | Thấp |
| **Linh hoạt** | Cao | Trung bình | Thấp |
| **Chi phí quản trị** | Cao | Trung bình | Thấp |
| **Thời gian ra thị trường** | Chậm | Nhanh | Ngay lập tức |
| **Tùy biến** | Cao | Trung bình | Thấp |
| **Khả năng dự đoán chi phí** | Biến động | Dự đoán được | Dự đoán được |
| **Yêu cầu chuyên môn kỹ thuật** | Cao | Trung bình | Thấp |

## Xu hướng phát triển của các mô hình dịch vụ

Cùng với sự phát triển của hệ sinh thái đám mây, ranh giới giữa ba mô hình truyền thống ngày càng linh hoạt và xuất hiện thêm nhiều mô hình trung gian. **Function as a Service (FaaS)** — hay tính toán không máy chủ (serverless) — đẩy mức độ trừu tượng hóa cao hơn cả PaaS khi mã nguồn chỉ thực thi khi có sự kiện kích hoạt. **Container as a Service (CaaS)** nằm giữa IaaS và PaaS, cung cấp môi trường quản lý chuyên biệt để triển khai và điều phối các ứng dụng đóng gói trong container.

## Kết luận

Nắm vững ba mô hình dịch vụ cốt lõi — IaaS, PaaS và SaaS — giúp tổ chức đưa ra quyết định kiến trúc có cơ sở. Mỗi mô hình có sự đánh đổi riêng giữa quyền kiểm soát, độ linh hoạt và chi phí vận hành, tùy thuộc vào năng lực kỹ thuật và mục tiêu kinh doanh cụ thể.

Bài tiếp theo sẽ trình bày các mô hình triển khai đám mây và cách kết hợp chúng với các mô hình dịch vụ để hoàn thiện kiến trúc tổng thể.

## Video tham khảo

- **Xem trước:** [3 dịch vụ của cloud - IAAS, PAAS, SAAS](https://www.youtube.com/watch?v=KGvoeQE1aG0) — Mai DE (tiếng Việt). Giới thiệu trực quan ba mô hình IaaS, PaaS, SaaS bằng tiếng Việt, bám sát khung nội dung bài học.
- [AZ-900 Episode 5 \| IaaS vs PaaS vs SaaS cloud service models \| Microsoft Azure Fundamentals Course](https://www.youtube.com/watch?v=9CVBohl6w0Q) — Adam Marczak - Azure for Everyone (tiếng Anh). So sánh IaaS, PaaS, SaaS và phân định rõ trách nhiệm vận hành ở từng tầng.
- [Mô hình cloud - On premises - Iaas - PaaS - SaaS](https://www.youtube.com/watch?v=as7_vyJ3qvA) — Trophy Vietnam (tiếng Việt). Đặt mô hình on-premises cạnh IaaS, PaaS và SaaS để đối chiếu phạm vi trách nhiệm.
