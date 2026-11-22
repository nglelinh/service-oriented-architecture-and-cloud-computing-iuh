---
layout: post
title: 01-05 Trung tâm dữ liệu và mở rộng quy mô
chapter: '01'
order: 6
owner: Nguyen Le Linh
lang: vi
categories:
- chapter01
lesson_type: optional
---

Đằng sau khái niệm trừu tượng "đám mây" là một hệ thống hạ tầng vật lý quy mô lớn: các trung tâm dữ liệu (data center). Hiểu cách năng lực tính toán mở rộng từ một máy tính cá nhân thành một hệ thống máy tính quy mô nhà kho (warehouse-scale computer) là kiến thức nền tảng của kỹ thuật đám mây.

## Nhu cầu về quy mô

Các dịch vụ web toàn cầu hiện nay vận hành với khối lượng dữ liệu và lưu lượng truy cập vượt xa khả năng đáp ứng của một máy chủ đơn lẻ. Nhiều hệ thống xử lý từ hàng **petabyte (PB)** đến **exabyte (EB)** dữ liệu (để tham chiếu, 1 zettabyte tương đương 1 nghìn tỷ gigabyte). Các nền tảng như Facebook hay YouTube phục vụ hàng tỷ lượt truy cập mỗi ngày, đòi hỏi các cụm máy chủ lớn phối hợp xử lý song song để duy trì độ trễ thấp.

## Hai hướng tiếp cận mở rộng quy mô

Khi một máy chủ chạm ngưỡng giới hạn hiệu năng, kiến trúc sư hệ thống có hai chiến lược chính để nâng công suất: mở rộng dọc và mở rộng ngang.

### 1. Mở rộng dọc — Scale Up (Vertical Scaling)

Mở rộng dọc ("scale up") là nâng cấp phần cứng (CPU mạnh hơn, nhiều RAM hơn hoặc ổ lưu trữ tốc độ cao hơn) trên cùng một máy chủ hiện có. Hướng tiếp cận này thể hiện rõ trong tiến trình phát triển từ máy tính cá nhân lên máy trạm, máy chủ chuyên dụng và cuối cùng là **máy tính lớn (mainframe)**.

Về mặt khái niệm, cách này đơn giản vì không cần thay đổi nhiều trong kiến trúc phần mềm, nhưng gặp ba giới hạn lớn. Thứ nhất là **giới hạn vật lý của phần cứng**: tốc độ CPU và dung lượng RAM tối đa của bo mạch chủ đều có trần cứng. Thứ hai, **chi phí tăng phi tuyến**: phần cứng cao cấp nhất thường đắt hơn nhiều lần so với phần cứng phổ thông nhưng mức tăng hiệu năng không tương xứng. Thứ ba, và đặc biệt quan trọng đối với độ tin cậy trên đám mây, một máy chủ đơn lẻ luôn là **điểm lỗi đơn (single point of failure)** — nếu máy đó gặp sự cố, toàn bộ hệ thống ngừng hoạt động.

### 2. Mở rộng ngang — Scale Out (Horizontal Scaling)

Mở rộng ngang ("scale out") là bổ sung thêm nhiều máy chủ vào hệ thống thay vì cố gắng nâng cấp một máy duy nhất. Đây là nguyên lý kiến trúc nền tảng của Internet hiện đại: thay cho một mainframe đơn lẻ, hệ thống được tổ chức thành một **cụm (cluster)** gồm nhiều máy chủ tiêu chuẩn, mở rộng thành trung tâm dữ liệu và mạng lưới trung tâm dữ liệu toàn cầu.

Hướng tiếp cận này mang lại nhiều ưu điểm quan trọng. Hệ thống có thể sử dụng **phần cứng phổ thông (commodity hardware)** với giá thành thấp hơn nhiều so với mainframe; chi phí mở rộng tăng gần như tuyến tính theo nhu cầu công suất. Quan trọng hơn, mở rộng ngang mang lại **khả năng chịu lỗi cao**: trong một cụm 1.000 máy chủ, nếu một máy hỏng, 999 máy còn lại vẫn tiếp tục gánh tải mà không gây gián đoạn dịch vụ. Đổi lại, **độ phức tạp phần mềm** tăng lên đáng kể do phải quản lý trạng thái phân tán và tính nhất quán dữ liệu trên hàng nghìn nút.

## Trung tâm dữ liệu dưới góc nhìn một hệ thống máy tính

Trung tâm dữ liệu hiện đại không chỉ là phòng đặt máy chủ mà được thiết kế như một "máy tính quy mô nhà kho" thống nhất, tối ưu đồng thời cho hiệu suất, độ tin cậy và quy mô.

### Kiến trúc vật lý

Đơn vị cơ bản của trung tâm dữ liệu là **máy chủ (server)**. Hàng chục máy chủ được lắp vào một khung tiêu chuẩn gọi là **tủ rack** (ví dụ khoảng 40 máy chủ mỗi rack). Mỗi rack có một bộ chuyển mạch **Top of Rack (ToR) switch** kết nối các máy chủ bên trong vào mạng chung. Hàng trăm đến hàng nghìn tủ rack được ghép nối thành các **cụm (cluster)** vận hành như một thực thể tính toán lớn.

### Các đặc điểm hạ tầng then chốt

Để vận hành ổn định ở quy mô lớn, trung tâm dữ liệu đòi hỏi **hạ tầng mạng băng thông cao, độ trễ thấp** (network fabric) liên kết toàn bộ các nút. **Cơ chế dự phòng (redundancy)** được thiết kế ở mọi tầng — từ bộ lưu điện (UPS), máy phát điện diesel, hệ thống làm mát dự phòng đến nhiều tuyến mạng độc lập — nhằm đảm bảo hoạt động liên tục. Bên cạnh đó là **an ninh vật lý nhiều lớp** như kiểm soát ra vào nghiêm ngặt, xác thực sinh trắc học và cửa ngăn hai lớp (man trap).

### Điện năng và tác động môi trường

Trung tâm dữ liệu tiêu thụ lượng điện năng rất lớn: một tủ rack có thể tiêu thụ hơn 4 kW, còn một trung tâm dữ liệu siêu quy mô (hyperscale) sử dụng lượng điện tương đương một đô thị nhỏ. Hầu hết điện năng này chuyển hóa thành nhiệt và phải được tản ra ngoài để bảo vệ thiết bị; do đó **hệ thống làm mát** thường chiếm 30–50% tổng điện năng tiêu thụ của cơ sở. Vì lý do này, nhiều trung tâm dữ liệu được đặt gần nguồn năng lượng tái tạo giá rẻ — chẳng hạn các nhà máy thủy điện dọc lưu vực sông Columbia — nhằm giảm chi phí vận hành và hạn chế tác động môi trường.

## Trung tâm dữ liệu mô-đun và phân tán

### Trung tâm dữ liệu mô-đun (Modular Data Center)

Để rút ngắn thời gian xây dựng và mở rộng công suất, nhiều đơn vị áp dụng mô hình **trung tâm dữ liệu mô-đun**, trong đó máy chủ, thiết bị mạng và hệ thống làm mát được lắp ráp sẵn trong các container tiêu chuẩn. Khi cần mở rộng, chỉ cần vận chuyển container tới địa điểm, đấu nối nguồn điện, hệ thống làm mát và đường truyền mạng. Cách tiếp cận dạng mô-đun lắp đặt nhanh này giúp tối ưu luồng khí làm mát và triển khai linh hoạt.

### Trung tâm dữ liệu phân tán

Với các dịch vụ phục vụ người dùng toàn cầu, một trung tâm dữ liệu đơn lẻ không thể đáp ứng yêu cầu về độ trễ và độ tin cậy do các giới hạn vật lý. **Độ trễ mạng (latency)** bị giới hạn bởi tốc độ truyền tín hiệu ánh sáng: người dùng tại châu Á truy cập máy chủ đặt tại Mỹ sẽ cảm nhận độ trễ rõ rệt. Hơn nữa, tập trung toàn bộ hệ thống tại một địa điểm tiềm ẩn rủi ro lớn khi xảy ra thiên tai. Để khắc phục, các nhà cung cấp xây dựng **mạng lưới trung tâm dữ liệu phân tán toàn cầu**, nhân bản dữ liệu giữa nhiều khu vực và định tuyến yêu cầu tới điểm gần người dùng nhất (Edge), vừa cải thiện hiệu năng vừa tăng khả năng chống chịu sự cố.

## Video tham khảo

- **Xem trước:** [Infra Coffee Time #30: Trung tâm dữ liệu là gì? Các loại TTDL phổ biến](https://www.youtube.com/watch?v=oDUHJxuVLiE) — Viettel IDC (tiếng Việt). Giới thiệu khái niệm trung tâm dữ liệu và các loại hình phổ biến bằng tiếng Việt.
- [Google Data Center 360° Tour](https://www.youtube.com/watch?v=zDAYZU4A3w0) — Google Cloud Tech (tiếng Anh). Tham quan thực tế không gian tủ rack, hệ thống điện và làm mát trong trung tâm dữ liệu.
- [Vertical Vs Horizontal Scaling: Key Differences You Should Know](https://www.youtube.com/watch?v=dvRFHG2-uYs) — ByteByteGo (tiếng Anh). So sánh trực quan mở rộng dọc (`scale up`) và mở rộng ngang (`scale out`).
