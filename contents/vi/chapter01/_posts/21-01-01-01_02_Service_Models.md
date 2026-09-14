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


Dịch vụ điện toán đám mây thường được phân thành ba mô hình chính, mỗi mô hình trao mức kiểm soát, linh hoạt và trách nhiệm quản trị khác nhau. Hiểu các mô hình này là then chốt để chọn chiến lược đám mây phù hợp với nhu cầu tổ chức.

## Chồng dịch vụ đám mây

Các mô hình dịch vụ có thể hình dung như một chồng (stack), mỗi lớp xây trên lớp trước:

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

## Hạ tầng như một dịch vụ (Infrastructure as a Service, IaaS)

IaaS là nền tảng của điện toán đám mây. Mô hình này cung cấp tài nguyên tính toán ảo hóa qua Internet, cho phép doanh nghiệp thuê thay vì mua hạ tầng CNTT.

### Định nghĩa và khái niệm cốt lõi
Ở lớp cơ bản, IaaS cung cấp các khối xây dựng của điện toán đám mây: máy ảo (virtual machine, VM), lưu trữ, mạng và hệ điều hành. Thay vì mua máy chủ vật lý và đặt trong trung tâm dữ liệu tại chỗ (on-premises), tổ chức cấp phát tài nguyên theo kiểu trả tiền theo mức sử dụng từ nhà cung cấp đám mây. Mô hình này trao mức linh hoạt và kiểm soát quản trị cao nhất, mô phỏng trung tâm dữ liệu truyền thống nhưng trong môi trường ảo hóa.

### Các thành phần chính
Một môi trường IaaS điển hình gồm nhiều thành phần. **Tài nguyên tính toán (compute)** là “ngựa thồ”, từ máy ảo chuẩn đến máy chủ bare metal và hàm serverless. **Dịch vụ lưu trữ** cung cấp lựa chọn mở rộng cho nhu cầu khác nhau: block storage cho cơ sở dữ liệu, object storage cho khối lượng lớn dữ liệu phi cấu trúc như bản sao lưu và tệp đa phương tiện. **Mạng** cho phép bạn định nghĩa cấu trúc mạng ảo riêng — subnet, bảng định tuyến, tường lửa — giống như với switch và router vật lý.

### Mô hình trách nhiệm
Trong IaaS, nhà cung cấp quản lý hạ tầng vật lý bên dưới — máy chủ vật lý, lớp ảo hóa (hypervisor), phần cứng lưu trữ và mạng vật lý. Khách hàng chịu trách nhiệm mọi thứ phía trên hypervisor: hệ điều hành, middleware, môi trường runtime, dữ liệu và ứng dụng. Đặc biệt, bạn cũng chịu trách nhiệm cấu hình bảo mật các thành phần này, chẳng hạn vá hệ điều hành và cấu hình tường lửa.

### Tình huống sử dụng
IaaS phù hợp khi cần kiểm soát chi tiết. Mô hình này lý tưởng cho **phát triển và kiểm thử**: nhóm có thể tạo môi trường tạm trong vài phút rồi giải phóng ngay. Nó hỗ trợ **phục hồi thảm họa (disaster recovery)** bằng cách nhân bản hạ tầng then chốt sang vùng địa lý khác mà không tốn chi phí một site vật lý thứ hai. Ngoài ra, khối lượng **tính toán hiệu năng cao (High-Performance Computing, HPC)** — thường cần cấu hình phần cứng cụ thể cho mô phỏng khoa học hay mô hình tài chính — phát huy tốt trên năng lực tính toán có thể mở rộng của IaaS.

### Ưu và nhược điểm
Ưu điểm chính của IaaS là **kiểm soát**. Bạn tự do cấu hình môi trường đúng theo yêu cầu. Mô hình tránh chi phí vốn lớn khi mua phần cứng và cho phép mở rộng nhanh. Tuy nhiên, sự tự do đó đi kèm **chi phí quản trị**: đội ngũ phải có năng lực kỹ thuật để quản lý hệ điều hành, bản vá bảo mật và cấu hình mạng — công việc phức tạp và tốn thời gian.

## Nền tảng như một dịch vụ (Platform as a Service, PaaS)

PaaS loại bỏ gánh nặng quản lý hạ tầng bên dưới, để bạn tập trung vào năng suất và phát triển ứng dụng.

### Định nghĩa và khái niệm cốt lõi
PaaS cung cấp môi trường phát triển và triển khai đầy đủ trên đám mây. Nó gồm không chỉ hạ tầng (máy chủ, lưu trữ, mạng) mà còn middleware, công cụ phát triển, dịch vụ trí tuệ kinh doanh, hệ quản trị cơ sở dữ liệu và nhiều thành phần khác. Mô hình được thiết kế để hỗ trợ toàn bộ vòng đời ứng dụng web: xây dựng, kiểm thử, triển khai, quản lý và cập nhật.

### Các thành phần chính
Gói PaaS thường gồm bộ **công cụ phát triển** và **môi trường runtime** hỗ trợ nhiều ngôn ngữ như Java, Python và Node.js. Chúng thường cung cấp **dịch vụ cơ sở dữ liệu được quản lý** (cả SQL và NoSQL), **lớp cache** và **hàng đợi thông điệp**, giúp bạn không phải cài đặt và cấu hình các hệ thống phức tạp này thủ công. Ngoài ra, giải pháp PaaS thường có sẵn **đường ống triển khai (CI/CD)** và **tự động mở rộng (auto-scaling)**, bảo đảm ứng dụng chịu được đột biến lưu lượng mà không cần can thiệp thủ công.

### Mô hình trách nhiệm
Sự dịch chuyển trách nhiệm ở PaaS là đáng kể. Nhà cung cấp quản lý hệ điều hành, middleware và môi trường runtime, ngoài hạ tầng vật lý. Trách nhiệm của bạn thường còn lại hai phần: **ứng dụng** và **dữ liệu**. Lập trình viên tập trung viết mã thay vì vá máy chủ.

### Tình huống sử dụng
PaaS là lựa chọn hàng đầu cho **phát triển ứng dụng web và di động**. Nó cho phép các nhóm đa dạng cộng tác bất kể vị trí địa lý. Mô hình cũng phù hợp để triển khai **API và microservice**, khi các thành phần nhỏ, độc lập có thể được triển khai và quản lý dễ dàng.

### Ưu và nhược điểm
Lợi ích lớn nhất của PaaS là **tốc độ**. Nó rút ngắn thời gian ra thị trường bằng cách lo phần “ống nước” của việc giao ứng dụng. Mô hình giảm độ phức tạp phát triển và có sẵn khả năng mở rộng. Nhược điểm là **khóa nhà cung cấp (vendor lock-in)**: ứng dụng có thể được xây bằng công cụ hoặc API độc quyền, khó chuyển sang nền tảng khác. Bạn cũng có **ít kiểm soát hơn** với môi trường bên dưới — có thể là ràng buộc với ứng dụng đòi hỏi yêu cầu hệ thống rất cụ thể.

## Phần mềm như một dịch vụ (Software as a Service, SaaS)

SaaS là mô hình quen thuộc nhất với người dùng cuối: giao ứng dụng hoàn chỉnh qua Internet.

### Định nghĩa và khái niệm cốt lõi
SaaS cho phép người dùng kết nối và sử dụng ứng dụng trên đám mây qua Internet. Ví dụ phổ biến: thư điện tử, lịch và bộ công cụ văn phòng. Nhà cung cấp quản lý toàn bộ chồng công nghệ — từ máy chủ vật lý đến mã ứng dụng. Người dùng thường truy cập phần mềm qua trình duyệt hoặc ứng dụng khách nhẹ, theo hình thức đăng ký (subscription).

### Đặc điểm chính
SaaS được định nghĩa bởi **đa thuê bao**: một phiên bản phần mềm phục vụ nhiều khách hàng (tenant) trong khi cô lập dữ liệu của họ. Mô hình thường vận hành theo **đăng ký** (phí tháng hoặc năm) và **cập nhật tự động**. Người dùng luôn dùng phiên bản mới nhất mà không cần tải bản vá hay tự nâng cấp.

### Mô hình trách nhiệm
Trong SaaS, khách hàng có trách nhiệm ít nhất, chủ yếu **quản lý dữ liệu** và **quyền truy cập người dùng**. Nhà cung cấp lo phần còn lại: phần mềm ứng dụng, bảo mật, cơ sở dữ liệu, máy chủ và hạ tầng mạng.

### Các nhóm ứng dụng SaaS
SaaS phủ nhiều lĩnh vực. **Bộ năng suất** như Microsoft 365 và Google Workspace hỗ trợ cộng tác. **Quản lý quan hệ khách hàng (Customer Relationship Management, CRM)** như Salesforce giúp doanh nghiệp quản lý tương tác khách hàng. **Hoạch định nguồn lực doanh nghiệp (Enterprise Resource Planning, ERP)** như NetSuite tích hợp quy trình nghiệp vụ cốt lõi. Ngay cả công cụ sáng tạo chuyên biệt như Adobe Creative Cloud cũng được cung cấp theo dạng SaaS.

### Ưu và nhược điểm
SaaS loại bỏ nhu cầu cài đặt, bảo trì và mua phần cứng, nên rất **dễ tiếp cận** và triển khai. Chi phí dự đoán được nhờ đăng ký. Tuy nhiên, mức **kiểm soát và tùy biến thấp nhất**. Bạn bị giới hạn bởi tính năng nhà cung cấp đưa ra, và **bảo mật dữ liệu** phụ thuộc nhiều vào biện pháp của họ.

## Lựa chọn mô hình dịch vụ phù hợp

Chọn mô hình dịch vụ là sự đánh đổi giữa kiểm soát và tiện lợi.

### Khung ra quyết định
Khi chọn mô hình, hãy cân nhắc:
- Chọn **IaaS** khi cần kiểm soát tối đa, đang di chuyển ứng dụng kế thừa đòi hỏi cấu hình hệ điều hành cụ thể, hoặc có đội vận hành mạnh.
- Chọn **PaaS** khi xây ứng dụng mới và muốn tối ưu tốc độ phát triển, giảm thiểu quản trị.
- Chọn **SaaS** cho quy trình nghiệp vụ chuẩn (thư điện tử, CRM, nhân sự) khi tự xây giải pháp riêng không mang lại lợi thế cạnh tranh.

Nhiều tổ chức hiện đại theo **cách tiếp cận lai**: dùng SaaS cho năng suất, PaaS cho ứng dụng hướng khách hàng mới, và IaaS cho khối lượng công việc chuyên biệt cần tùy biến sâu.

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

## Xu hướng tương lai của mô hình dịch vụ

Khi điện toán đám mây phát triển, ranh giới giữa các mô hình ngày càng mờ, và mô hình mới xuất hiện. **Function as a Service (FaaS)**, hay tính toán không máy chủ (serverless), ngày càng phổ biến vì trừu tượng hóa quản trị hạ tầng hơn cả PaaS: mã chỉ chạy khi có sự kiện. **Container as a Service (CaaS)** nằm giữa IaaS và PaaS, cung cấp môi trường được quản lý để triển khai ứng dụng đóng gói trong container.

## Kết luận

Hiểu ba mô hình dịch vụ đám mây chính — IaaS, PaaS và SaaS — là nền tảng để ra quyết định áp dụng đám mây có cơ sở. Mỗi mô hình đánh đổi khác nhau giữa kiểm soát, linh hoạt và chi phí quản trị. Lựa chọn phụ thuộc vào năng lực kỹ thuật, yêu cầu nghiệp vụ và mục tiêu chiến lược của tổ chức.

Bài tiếp theo sẽ tìm hiểu các mô hình triển khai đám mây và cách chúng bổ sung cho các mô hình dịch vụ để tạo nên giải pháp đám mây toàn diện.
