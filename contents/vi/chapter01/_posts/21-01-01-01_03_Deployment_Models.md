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

Mô hình triển khai đám mây (deployment model) xác định cách thức xây dựng hạ tầng, đối tượng được phép truy cập và cơ chế quản lý hệ thống. Nắm rõ các mô hình này giúp tổ chức chọn được kiến trúc đáp ứng đồng thời yêu cầu về bảo mật, tuân thủ pháp lý và hiệu quả nghiệp vụ.

## Tổng quan các mô hình triển khai

Bốn mô hình triển khai chính mang lại các mức độ khác nhau về kiểm soát, bảo mật và chi phí:

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

Đám mây công cộng cung cấp môi trường hạ tầng dùng chung, trong đó tài nguyên tính toán được phân phối rộng rãi qua Internet.

### Định nghĩa và đặc điểm

Trong mô hình này, các nhà cung cấp bên thứ ba như AWS, Microsoft Azure và Google Cloud Platform sở hữu và vận hành toàn bộ hạ tầng, đồng thời cung cấp máy chủ, lưu trữ và dịch vụ ứng dụng qua Internet. Nhiều tổ chức (tenant) cùng chia sẻ phần cứng vật lý nhưng dữ liệu và tài nguyên vẫn được cô lập logic. Nhờ kiến trúc **hạ tầng dùng chung** ở quy mô rất lớn, đám mây công cộng đạt được lợi thế kinh tế theo quy mô và tối ưu chi phí vận hành.

### Ưu và nhược điểm

Điểm mạnh nổi bật của đám mây công cộng là **hiệu quả chi phí**: doanh nghiệp không phải đầu tư vốn lớn ban đầu cho phần cứng mà chuyển sang chi phí vận hành trả theo mức dùng. Mô hình này cũng mang lại **khả năng mở rộng linh hoạt**, cho phép khởi tạo hàng nghìn máy chủ trong vài phút khi lưu lượng tăng đột biến. Đổi lại, khách hàng có **quyền kiểm soát hạn chế** đối với vị trí vật lý của dữ liệu và cấu hình hạ tầng bên dưới — yếu tố cần đặc biệt lưu ý trong các ngành chịu sự quản lý chặt chẽ. Ngoài ra, do dùng chung phần cứng, về mặt lý thuyết tồn tại hiện tượng "noisy neighbor" (tác động hiệu năng từ khách hàng dùng chung máy chủ), dù công nghệ ảo hóa hiện đại đã giảm thiểu đáng kể rủi ro này.

### Tình huống sử dụng

Đám mây công cộng là lựa chọn phổ biến cho phần lớn ứng dụng hiện đại như máy chủ web, môi trường phát triển/kiểm thử và nền tảng phân tích dữ liệu, đặc biệt phù hợp với doanh nghiệp khởi nghiệp cần triển khai nhanh hoặc các khối lượng công việc có biến động tải lớn.

## Đám mây riêng (Private Cloud)

Đám mây riêng cung cấp môi trường hạ tầng dành riêng, chỉ phục vụ duy nhất một doanh nghiệp hoặc tổ chức.

### Định nghĩa và đặc điểm

Hạ tầng đám mây riêng có thể đặt tại trung tâm dữ liệu nội bộ của tổ chức hoặc do nhà cung cấp bên thứ ba lưu trữ chuyên biệt. Điểm cốt lõi là tài nguyên và dịch vụ được vận hành trên mạng riêng, không chia sẻ với khách hàng khác, nhờ đó mang lại mức độ kiểm soát và bảo mật cao nhất.

### Các dạng đám mây riêng

- **Đám mây riêng tại chỗ (On-Premises Private Cloud):** Đặt ngay trong trung tâm dữ liệu của tổ chức, mang lại quyền kiểm soát toàn diện nhưng đòi hỏi đội ngũ kỹ thuật nội bộ đủ năng lực vận hành nền tảng ảo hóa (như VMware, OpenStack).
- **Đám mây riêng được lưu trữ (Hosted Private Cloud):** Thuê máy chủ vật lý chuyên biệt từ nhà cung cấp bên ngoài, đơn vị này đảm nhận phần quản lý phần cứng.
- **Đám mây riêng ảo (Virtual Private Cloud — VPC):** Mô hình trung gian trong đó nhà cung cấp đám mây công cộng tạo ra một phân vùng mạng cô lập logic dành riêng cho tổ chức ngay trên hạ tầng công cộng.

### Ưu và nhược điểm

Ưu điểm lớn nhất của đám mây riêng là **bảo mật, quyền kiểm soát và hiệu năng ổn định**, cho phép tùy biến sát theo các quy định pháp lý nghiêm ngặt (như HIPAA hoặc GDPR). Tuy nhiên, nhược điểm lớn là **chi phí cao**: xây dựng tại chỗ đòi hỏi vốn đầu tư phần cứng ban đầu lớn cùng chi phí duy trì điện năng, làm mát và nhân sự CNTT. Đồng thời, khả năng mở rộng đàn hồi bị giới hạn bởi phần cứng hiện có — khi hết tài nguyên, tổ chức phải mua sắm và lắp đặt thêm máy chủ vật lý.

### Tình huống sử dụng

Đám mây riêng thường được áp dụng trong các lĩnh vực quản lý chặt như **tài chính, y tế và cơ quan chính phủ**, nơi quy định bảo vệ dữ liệu kiểm soát nghiêm ngặt nơi lưu trữ. Mô hình này cũng phù hợp với các ứng dụng kế thừa trọng yếu cần cấu hình phần cứng đặc thù mà đám mây công cộng không đáp ứng.

## Đám mây lai (Hybrid Cloud)

Đám mây lai kết hợp đám mây công cộng và đám mây riêng thông qua các công nghệ kết nối và điều phối, cho phép chia sẻ dữ liệu và ứng dụng xuyên suốt hai môi trường.

### Định nghĩa và đặc điểm

Mô hình này kết hợp ưu điểm của cả hai hướng tiếp cận trong một kiến trúc thống nhất: tổ chức có thể lưu giữ dữ liệu nhạy cảm và hệ thống lõi trên đám mây riêng, đồng thời tận dụng khả năng mở rộng của đám mây công cộng cho các tác vụ ít nhạy cảm hơn. Sự phối hợp này đòi hỏi hạ tầng kết nối tin cậy như VPN, đường truyền chuyên dụng (Direct Connect) hoặc lớp điều phối container như Kubernetes.

### Các mẫu kiến trúc phổ biến

Một mẫu quen thuộc là **Cloud Bursting**: ứng dụng vận hành trên đám mây riêng ở mức tải bình thường và tự động mở rộng tạm thời sang đám mây công cộng khi lưu lượng tăng đột biến. Một mẫu khác là **phân tầng dữ liệu (Data Tiering)**: dữ liệu định danh khách hàng nhạy cảm được giữ tại chỗ để đảm bảo tuân thủ, trong khi dữ liệu đã ẩn danh được đưa lên đám mây công cộng phục vụ phân tích và học máy.

### Ưu và nhược điểm

Đám mây lai mang lại **tính linh hoạt cao**, giúp tối ưu chi phí cho các khối lượng công việc ngắn hạn mà vẫn giữ kiểm soát tuân thủ đối với dữ liệu nhạy cảm, đồng thời hỗ trợ lộ trình chuyển đổi dần lên đám mây. Ngược lại, đây là mô hình **phức tạp nhất trong quản trị**, đòi hỏi kiến trúc mạng chặt chẽ, chính sách bảo mật đồng bộ giữa hai môi trường và trình độ kỹ thuật cao để đảm bảo khả năng tương tác.

## Đám mây cộng đồng (Community Cloud)

Đám mây cộng đồng là mô hình hợp tác, trong đó một nhóm tổ chức có chung mục tiêu, yêu cầu bảo mật hoặc quy định tuân thủ cùng chia sẻ hạ tầng đám mây.

### Định nghĩa và đặc điểm

Hạ tầng đám mây cộng đồng có thể do chính các tổ chức thành viên cùng quản lý hoặc ủy thác cho bên thứ ba vận hành. Mô hình này nằm ở khoảng giữa đám mây công cộng và đám mây riêng: không mở rộng cho công chúng nhưng cũng không giới hạn ở một tổ chức đơn lẻ.

### Ưu và nhược điểm

Lợi ích chính là **chia sẻ chi phí đầu tư**, giúp các tổ chức có yêu cầu tương đồng cùng xây dựng một hạ tầng chuẩn mực vốn quá tốn kém nếu triển khai riêng lẻ, đồng thời thúc đẩy **hợp tác** và thống nhất tiêu chuẩn ngành. Hạn chế của mô hình này nằm ở **cơ chế quản trị dùng chung**, dễ phát sinh bất đồng khi phân bổ tài nguyên hoặc cập nhật chính sách chung.

### Tình huống sử dụng

Đám mây cộng đồng thường gặp trong khối **cơ quan chính phủ** chia sẻ tài nguyên trên mạng bảo mật, trong ngành **y tế** khi các bệnh viện liên thông hồ sơ bệnh án, và trong **nghiên cứu học thuật** khi các trường đại học dùng chung cụm tính toán hiệu năng cao.

## Lựa chọn mô hình triển khai phù hợp

Quyết định lựa chọn mô hình triển khai cần cân đối giữa chi phí, quyền kiểm soát và yêu cầu tuân thủ:

- **Đám mây công cộng:** Phù hợp cho ứng dụng web, khối lượng công việc phổ thông và các dự án ưu tiên tối ưu chi phí cùng tốc độ triển khai.
- **Đám mây riêng:** Phù hợp khi có quy định pháp lý khắt khe, yêu cầu kiểm soát tuyệt đối chủ quyền dữ liệu hoặc tải vận hành ổn định, dễ dự báo.
- **Đám mây lai:** Phù hợp khi cần giữ một phần dữ liệu nhạy cảm tại chỗ nhưng vẫn muốn tận dụng khả năng mở rộng của đám mây công cộng cho các thành phần khác.
- **Đám mây cộng đồng:** Phù hợp khi tổ chức thuộc một liên minh hoặc nhóm ngành có nhu cầu tuân thủ và chia sẻ hạ tầng tương đồng.

## Xu hướng phát triển của mô hình triển khai

Thực tiễn triển khai hiện nay đang mở rộng theo hai hướng đáng chú ý. Thứ nhất là **đa đám mây (Multi-Cloud)**, khi tổ chức sử dụng dịch vụ từ nhiều nhà cung cấp công cộng khác nhau (ví dụ AWS cho tính toán và Google Cloud cho phân tích dữ liệu/AI) nhằm giảm phụ thuộc vào một nhà cung cấp duy nhất. Thứ hai là **điện toán biên (Edge Computing)**, đưa năng lực xử lý đến gần nguồn phát sinh dữ liệu hơn (như thiết bị IoT) để giảm độ trễ truyền tải.

## Kết luận

Hiểu rõ bốn mô hình triển khai đám mây giúp tổ chức xây dựng chiến lược hạ tầng phù hợp với độ nhạy cảm của dữ liệu, ngân sách và nhu cầu mở rộng. Mỗi mô hình đều có sự đánh đổi rõ rệt giữa chi phí, mức độ kiểm soát, bảo mật và độ phức tạp vận hành.

Bài tiếp theo sẽ phân tích sâu hơn các lợi ích và thách thức đặt ra khi tổ chức triển khai điện toán đám mây.

## Video tham khảo

- **Xem trước:** [AZ-900 Episode 6 \| Public, Private & Hybrid cloud deployment models \| Azure Fundamentals Course](https://www.youtube.com/watch?v=XlNR7myQydI) — Adam Marczak - Azure for Everyone (tiếng Anh). Trình bày ba mô hình public, private và hybrid cùng quyền sở hữu hạ tầng.
- [Cloud Deployment Models - Public Cloud, Private Cloud, Hybrid Cloud & Community Cloud](https://www.youtube.com/watch?v=GZsUFbLTH6s) — Software Architecture Academy (tiếng Anh). Bổ sung mô hình community cloud cạnh ba mô hình phổ biến, khớp với cấu trúc bốn mô hình của bài.
- [Public Cloud vs Private Cloud vs Hybrid Cloud](https://www.youtube.com/watch?v=3WIJ4axzFlU) — IntelCanada (tiếng Anh). Đối chiếu ngắn gọn ba mô hình public, private và hybrid phục vụ khung lựa chọn ở cuối bài.
