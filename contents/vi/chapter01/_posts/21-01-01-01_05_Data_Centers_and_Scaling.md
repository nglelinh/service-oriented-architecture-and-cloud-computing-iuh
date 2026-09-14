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


Đằng sau khái niệm trừu tượng “Cloud” là hạ tầng vật lý khổng lồ: các trung tâm dữ liệu (data center). Hiểu cách tính toán mở rộng từ một máy tính cá nhân tới một “máy tính cỡ nhà kho” là nền tảng của kỹ thuật đám mây.

## Nhu cầu về quy mô

Dịch vụ web hiện đại vận hành ở quy mô khó hình dung. Một máy chủ đơn lẻ không còn đủ để xử lý khối lượng dữ liệu và yêu cầu tính toán của ứng dụng toàn cầu. Ứng dụng ngày nay thường xử lý từ **petabyte (PB)** đến **exabyte (EB)** dữ liệu (để hình dung: 1 zettabyte bằng 1 nghìn tỷ gigabyte). Các dịch vụ như Facebook và YouTube phục vụ hàng tỷ người dùng mỗi ngày, cần cụm máy khổng lồ làm việc song song để giao nội dung mà không bị độ trễ.

## Hai cách tiếp cận mở rộng quy mô

Khi một máy tính đạt giới hạn, kiến trúc sư hệ thống đối mặt nút thắt hiệu năng có hai chiến lược chính để tăng dung lượng: mở rộng dọc và mở rộng ngang.

### 1. Mở rộng dọc — Scale Up (Vertical Scaling)

Mở rộng dọc, thường gọi là “scale up”, là thêm sức mạnh (CPU, RAM hoặc lưu trữ nhanh hơn) cho máy hiện có. Ta thấy tiến trình này trong sự tiến hóa từ máy tính cá nhân sang máy trạm mạnh, rồi máy chủ, và cuối cùng là **máy tính lớn (mainframe)**.

Cách này về khái niệm thì đơn giản — bạn chỉ cần mua máy lớn hơn — nhưng có giới hạn nghiêm trọng. Thứ nhất, có **giới hạn phần cứng cứng**: CPU chỉ nhanh đến một mức, bo mạch chủ chỉ hỗ trợ một lượng RAM nhất định. Thứ hai, chi phí tăng theo hàm mũ với phần cứng cao cấp; bộ xử lý nhanh nhất thường đắt hơn hẳn so với dòng tầm trung, không tương xứng với phần tăng hiệu năng. Cuối cùng, và có lẽ quan trọng nhất với độ tin cậy đám mây, một máy khổng lồ đơn lẻ là **điểm lỗi đơn (single point of failure)**. Nếu siêu máy chủ đó sập, toàn bộ ứng dụng ngừng.

### 2. Mở rộng ngang — Scale Out (Horizontal Scaling)

Mở rộng ngang, hay “scale out”, là thêm máy vào hệ thống thay vì làm một máy mạnh hơn. Đây là bước đột phá đã tạo nên Internet hiện đại. Thay vì một mainframe, bạn xây một **cụm (cluster)** máy chủ chuẩn, rồi lớn dần thành trung tâm dữ liệu, và cuối cùng là mạng trung tâm dữ liệu toàn cầu.

Ưu điểm của cách này rất sâu. Nó cho phép dùng **phần cứng phổ thông (commodity hardware)**, rẻ hơn đáng kể so với mainframe cao cấp. Chi phí tăng gần tuyến tính: muốn gấp đôi sức mạnh thì mua gấp đôi số máy chủ rẻ. Quan trọng hơn, nó mang lại **khả năng chịu lỗi cao**. Trong cụm 1.000 máy chủ, nếu một máy hỏng, 999 máy còn lại gánh tải, hệ thống tiếp tục không gián đoạn. Thách thức là **độ phức tạp** tăng trong kiến trúc phần mềm: phải quản lý trạng thái phân tán và tính nhất quán trên hàng nghìn nút.

## Trung tâm dữ liệu như một máy tính

Trung tâm dữ liệu hiện đại về bản chất là “máy tính cỡ nhà kho”. Không chỉ là phòng chứa máy chủ; đó là một hệ thống tổng thể được thiết kế cho hiệu quả và quy mô.

### Kiến trúc
Khối xây dựng của trung tâm dữ liệu là **máy chủ (server)**. Hàng chục máy chủ được gắn vào khung vật lý gọi là **tủ rack** (ví dụ 40 máy chủ mỗi rack). Mỗi rack có **switch “Top of Rack”** nối các máy chủ vào mạng lớn hơn. Hàng trăm hoặc hàng nghìn rack được tổ chức thành một **cụm (cluster)**, cùng làm việc như một thực thể tính toán.

### Đặc điểm then chốt
Để hỗ trợ quy mô này, trung tâm dữ liệu cần hạ tầng **mạng khổng lồ**, với kết cấu liên kết (network fabric) băng thông cao, độ trễ thấp nối mọi nút. **Dự phòng (redundancy)** được xây ở mọi lớp: nguồn dự phòng (UPS), máy phát diesel, hệ thống làm mát dư thừa và nhiều đường mạng bảo đảm cơ sở không bao giờ tắt. **An ninh** là tối quan trọng: kiểm soát ra vào vật lý nghiêm ngặt, máy quét sinh trắc học và “man trap” ngăn người không được phép.

### Năng lượng và tác động môi trường
Trung tâm dữ liệu tiêu thụ điện rất lớn. Một rack có thể dùng hơn 4 kW, và một trung tâm dữ liệu siêu quy mô (hyperscale) tiêu thụ điện tương đương một thành phố nhỏ. Toàn bộ điện đó biến thành nhiệt, phải được đưa ra ngoài để tránh hỏng phần cứng. Hệ quả là **hệ thống làm mát** thường chiếm 30–50% tổng năng lượng của cơ sở. Đó là lý do nhiều trung tâm dữ liệu được xây gần nguồn năng lượng xanh, giá rẻ — chẳng hạn đập thủy điện lưu vực sông Columbia — để giảm chi phí vận hành và hạn chế tác động môi trường.

## Trung tâm dữ liệu mô-đun và phân tán

### Trung tâm dữ liệu mô-đun
Xu hướng hiện đại để tăng tốc triển khai là **trung tâm dữ liệu mô-đun (Modular Data Center)**. Trong mô hình này, máy chủ, mạng và làm mát được lắp sẵn trong container vận chuyển chuẩn. Để mở rộng dung lượng, công ty chỉ cần chuyển một container mới tới hiện trường, cắm điện, nước (để làm mát) và kết nối Internet. Cách tiếp cận “cắm là chạy” này có thiết kế luồng khí và làm mát được tối ưu cao, cho phép mở rộng nhanh.

### Trung tâm dữ liệu phân tán
Với dịch vụ toàn cầu, một trung tâm dữ liệu đơn lẻ là không đủ vì các định luật vật lý. **Độ trễ (latency)** — thời gian dữ liệu di chuyển — bị giới hạn bởi tốc độ ánh sáng. Người dùng ở châu Á truy cập trung tâm dữ liệu tại Mỹ sẽ cảm nhận độ trễ rõ. Hơn nữa, phụ thuộc một vị trí tạo rủi ro: thiên tai có thể xóa sổ toàn bộ dịch vụ. Để giải quyết, công ty triển khai **mạng trung tâm dữ liệu phân tán toàn cầu**, nhân bản dữ liệu giữa các vùng và định tuyến lưu lượng người dùng tới vị trí gần nhất (“Edge”). Cách này bảo đảm cả hiệu năng cao cho người dùng lẫn khả năng phục hồi cho doanh nghiệp.
