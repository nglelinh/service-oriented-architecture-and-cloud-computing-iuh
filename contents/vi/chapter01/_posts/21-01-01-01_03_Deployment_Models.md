---
layout: post
title: 01-03 Các mô hình triển khai đám mây
chapter: '01'
order: 4
owner: Nguyen Le Linh
lang: vi
categories:
- chapter01
lesson_type: required
---


Mô hình triển khai đám mây (deployment model) xác định hạ tầng đám mây được triển khai thế nào, ai được truy cập và cách nó được quản lý. Hiểu các mô hình này giúp tổ chức chọn chiến lược đám mây phù hợp với yêu cầu bảo mật, tuân thủ và nghiệp vụ.

## Tổng quan các mô hình triển khai

Bốn mô hình triển khai chính, mỗi mô hình có mức kiểm soát, bảo mật và chi phí khác nhau:

```
┌─────────────────┬─────────────────┬─────────────────┬─────────────────┐
│   Public Cloud  │  Private Cloud  │  Hybrid Cloud   │ Community Cloud │
│ Shared          │ Dedicated       │ Mixed           │ Shared by Group │
│ Multi-tenant    │ Single-tenant   │ Best of Both    │ Common Interests│
│ Cost-effective  │ High Control    │ Flexible        │ Cost Sharing    │
│ Scalable        │ Secure          │ Complex         │ Specialized     │
└─────────────────┴─────────────────┴─────────────────┴─────────────────┘
```

## Đám mây công cộng (Public Cloud)

Đám mây công cộng tạo môi trường dùng chung, trong đó tài nguyên tính toán được công chúng truy cập qua Internet.

### Định nghĩa và đặc điểm
Trong đám mây công cộng, nhà cung cấp bên thứ ba như AWS, Microsoft Azure và Google Cloud Platform sở hữu và vận hành hạ tầng. Họ cung cấp tài nguyên tính toán — máy chủ, lưu trữ và ứng dụng — qua Internet. Nhiều tổ chức, hay “tenant”, chia sẻ cùng phần cứng vật lý, dù dữ liệu vẫn được cô lập logic. Mô hình **hạ tầng dùng chung** này tạo lợi thế kinh tế theo quy mô lớn, khiến đám mây công cộng có hiệu quả chi phí cao.

### Ưu và nhược điểm
Sức hút chính của đám mây công cộng là **hiệu quả chi phí**. Không cần đầu tư vốn trước cho phần cứng, doanh nghiệp coi CNTT như chi phí vận hành, chỉ trả cho những gì dùng. Mô hình cung cấp **khả năng mở rộng** gần như không giới hạn: bạn có thể khởi tạo hàng nghìn máy chủ trong vài phút để chịu đột biến lưu lượng. Tuy nhiên, có sự đánh đổi. Bạn **kiểm soát hạn chế** nơi dữ liệu thực sự nằm và cách hạ tầng bên dưới được cấu hình — điều đáng lo với ngành bị quản lý chặt. Ngoài ra, vì tài nguyên dùng chung, tồn tại rủi ro lý thuyết “noisy neighbor” ảnh hưởng hiệu năng, dù hypervisor hiện đại đã giảm đáng kể vấn đề này.

### Tình huống sử dụng
Đám mây công cộng là lựa chọn mặc định cho hầu hết ứng dụng hiện đại: máy chủ web, môi trường phát triển và nền tảng phân tích dữ liệu. Mô hình phù hợp cho startup cần ra mắt nhanh và doanh nghiệp muốn chuyển khối lượng công việc biến động ra ngoài.

## Đám mây riêng (Private Cloud)

Đám mây riêng cung cấp môi trường dành riêng, trong đó tài nguyên tính toán chỉ phục vụ một doanh nghiệp hoặc tổ chức.

### Định nghĩa và đặc điểm
Đám mây riêng có thể đặt vật lý tại trung tâm dữ liệu của tổ chức hoặc được nhà cung cấp bên thứ ba lưu trữ. Dù vị trí nào, điểm khác biệt then chốt là dịch vụ và hạ tầng được duy trì trên mạng riêng, chỉ dành cho tổ chức của bạn. Mô hình này trao mức bảo mật và kiểm soát cao nhất vì tài nguyên không chia sẻ với tenant khác.

### Các dạng đám mây riêng
Đám mây riêng có nhiều hình thức. **Đám mây riêng tại chỗ (On-Premises Private Cloud)** được lưu trữ trong trung tâm dữ liệu của bạn: kiểm soát toàn diện nhưng đòi hỏi chuyên môn nội bộ để quản lý chồng ảo hóa (ví dụ VMware, OpenStack). **Đám mây riêng được lưu trữ (Hosted Private Cloud)** là thuê máy chủ dành riêng từ nhà cung cấp quản lý phần cứng giúp bạn. **Đám mây riêng ảo (Virtual Private Cloud, VPC)** là khái niệm lai: nhà cung cấp đám mây công cộng tạo một phân đoạn cô lập logic trên đám mây công cộng cho riêng bạn, nối khoảng cách giữa mô hình công cộng và riêng.

### Ưu và nhược điểm
Ưu điểm chính của đám mây riêng là **bảo mật và kiểm soát**. Bạn có thể tùy chỉnh môi trường theo yêu cầu pháp lý cụ thể (như HIPAA hoặc GDPR) và bảo đảm hiệu năng dự đoán được. Tuy nhiên, cái giá rất cao: **chi phí lớn**. Xây đám mây riêng tại chỗ đòi hỏi đầu tư vốn đáng kể vào phần cứng và chi phí vận hành liên tục cho điện, làm mát và nhân sự CNTT. Mô hình cũng thiếu tính đàn hồi khổng lồ của đám mây công cộng: hết dung lượng thì phải mua và lắp thêm máy chủ vật lý.

### Tình huống sử dụng
Đám mây riêng thường cần thiết với các ngành bị quản lý chặt như **tài chính, y tế và chính phủ**, nơi luật bảo vệ dữ liệu kiểm soát nghiêm ngặt chỗ lưu và cách lưu dữ liệu. Chúng cũng dùng cho ứng dụng kế thừa then chốt đòi hỏi cấu hình phần cứng cụ thể mà đám mây công cộng không có.

## Đám mây lai (Hybrid Cloud)

Đám mây lai kết hợp đám mây công cộng và riêng, được liên kết bằng công nghệ cho phép chia sẻ dữ liệu và ứng dụng giữa chúng.

### Định nghĩa và đặc điểm
Đám mây lai mang lại “cái tốt nhất của cả hai thế giới” bằng một môi trường thống nhất. Bạn có thể giữ dữ liệu nhạy cảm và ứng dụng then chốt trên đám mây riêng an toàn, đồng thời tận dụng năng lực tính toán của đám mây công cộng cho tác vụ ít nhạy cảm hơn. Để việc này hoạt động, cần kết nối và điều phối liền mạch giữa hai môi trường, thường qua VPN, liên kết Direct Connect, hoặc nền tảng điều phối container như Kubernetes.

### Các mẫu kiến trúc
Một mẫu phổ biến là **Cloud Bursting**. Ứng dụng chạy trên đám mây riêng khi tải bình thường nhưng “bùng” sang đám mây công cộng lúc đỉnh để xử lý lưu lượng tràn. Mẫu khác là **phân tầng dữ liệu (Data Tiering)**: dữ liệu khách hàng nhạy cảm lưu tại chỗ để tuân thủ, trong khi dữ liệu đã ẩn danh được gửi lên đám mây công cộng để phân tích học máy.

### Ưu và nhược điểm
Mô hình lai mang **tính linh hoạt** vượt trội. Bạn tối ưu chi phí bằng tài nguyên đám mây công cộng cho khối lượng tạm thời, đồng thời giữ tuân thủ cho dữ liệu nhạy cảm tại chỗ. Nó cho phép chiến lược di chuyển dần, chuyển khối lượng công việc lên đám mây theo nhịp của bạn. Tuy nhiên, đây là mô hình **phức tạp** nhất để quản lý. Cần mạng tinh vi, chính sách bảo mật nhất quán giữa các môi trường, và chuyên môn kỹ thuật cao để bảo đảm khả năng tương tác.

## Đám mây cộng đồng (Community Cloud)

Đám mây cộng đồng là nỗ lực hợp tác, trong đó hạ tầng được chia sẻ giữa một số tổ chức thuộc một cộng đồng có mối quan tâm chung.

### Định nghĩa và đặc điểm
Trong đám mây cộng đồng, hạ tầng được chia sẻ bởi một số tổ chức có mối quan tâm chung (ví dụ sứ mệnh, yêu cầu bảo mật, chính sách và tuân thủ). Nó có thể do các tổ chức tự quản lý hoặc do bên thứ ba. Mô hình nằm giữa công cộng và riêng: không mở cho mọi người, nhưng cũng không giới hạn ở một tổ chức.

### Ưu và nhược điểm
Lợi ích then chốt là **chia sẻ chi phí**. Các tổ chức có nhu cầu tương tự có thể góp tài nguyên để xây hạ tầng chất lượng cao — thứ quá đắt nếu làm riêng. Mô hình thúc đẩy **hợp tác** và bảo đảm mọi thành viên đáp ứng cùng chuẩn đặc thù ngành. Nhược điểm là **quản trị dùng chung**, có thể dẫn đến xung đột về phân bổ tài nguyên và cập nhật chính sách.

### Tình huống sử dụng
Đám mây cộng đồng phổ biến trong **chính phủ**, khi các cơ quan chia sẻ tài nguyên trên mạng an toàn. Chúng cũng xuất hiện trong **y tế** (chia sẻ hồ sơ bệnh nhân giữa bệnh viện) và **nghiên cứu học thuật** (các trường đại học chia sẻ cụm tính toán hiệu năng cao).

## Lựa chọn mô hình triển khai phù hợp

Chọn mô hình triển khai là quyết định chiến lược cân bằng chi phí, kiểm soát và tuân thủ.

### Khung ra quyết định
- **Đám mây công cộng**: Chọn cho khối lượng công việc đa năng, ứng dụng web, và khi chi phí cùng tốc độ là động lực chính.
- **Đám mây riêng**: Chọn khi có yêu cầu pháp lý nghiêm ngặt, cần kiểm soát tuyệt đối chủ quyền dữ liệu, hoặc có khối lượng công việc ổn định, dự đoán được.
- **Đám mây lai**: Chọn khi cần giữ một phần dữ liệu tại chỗ để tuân thủ nhưng muốn khả năng mở rộng của đám mây công cộng cho phần còn lại của ứng dụng.
- **Đám mây cộng đồng**: Chọn khi bạn thuộc liên minh hoặc nhóm ngành có nhu cầu tuân thủ và hạ tầng dùng chung.

## Xu hướng tương lai của mô hình triển khai

Bức tranh đang tiến tới **đa đám mây (Multi-Cloud)**: tổ chức dùng dịch vụ từ nhiều nhà cung cấp đám mây công cộng (ví dụ AWS cho tính toán và Google Cloud cho AI) để tránh khóa nhà cung cấp. **Điện toán biên (Edge Computing)** cũng đang nổi, đẩy năng lực đám mây gần nguồn dữ liệu hơn (như thiết bị IoT) để giảm độ trễ.

## Kết luận

Hiểu các mô hình triển khai đám mây là cần thiết để ra quyết định chiến lược đám mây có cơ sở. Mỗi mô hình đánh đổi khác nhau về chi phí, kiểm soát, bảo mật và độ phức tạp. Lựa chọn phụ thuộc yêu cầu cụ thể của tổ chức, gồm độ nhạy cảm dữ liệu, ngân sách và nhu cầu mở rộng.

Bài tiếp theo sẽ xem lợi ích và thách thức của điện toán đám mây, giúp bạn hiểu tác động toàn diện khi tổ chức áp dụng đám mây.
