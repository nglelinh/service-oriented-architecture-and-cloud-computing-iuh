---
layout: post
title: 01 Giới thiệu điện toán đám mây
chapter: '01'
order: 1
owner: Nguyen Le Linh
lang: vi
categories:
- chapter01
---

Cách đây hơn một thế kỷ, mỗi nhà máy dệt hay xưởng cơ khí muốn có điện đều phải tự mua máy phát, tự trữ than và thuê đội thợ máy túc trực ngày đêm. Mãi đến khi lưới điện công cộng ra đời, doanh nghiệp mới chỉ cần cắm phích vào ổ điện và trả tiền theo số kilowatt-giờ thực dùng. Điện toán đám mây (cloud computing) tạo ra đúng bước ngoặt lịch sử đó cho ngành công nghệ thông tin: biến năng lực tính toán từ tài sản phần cứng phải tự mua sắm và bảo trì thành một dịch vụ tiện ích cấp phát theo nhu cầu.

## Bản đồ kiến thức nền: Bốn trụ cột của Chương 01 — Nền tảng điện toán đám mây

Để không bị lạc giữa hàng trăm tên dịch vụ thương mại, hãy nhìn toàn bộ Chương 01 qua bốn trụ cột kiến thức nền gắn kết chặt chẽ với nhau:

1. **Định nghĩa & Đặc trưng cốt yếu (Chuẩn NIST SP 800-145)**:
   Trả lời câu hỏi "thế nào mới thực sự là đám mây?" qua 5 đặc trưng kỹ thuật (tự phục vụ, truy cập mạng rộng, gom nhóm tài nguyên, đàn hồi nhanh, đo lường được), giúp phân biệt đám mây thật với việc chỉ thuê máy chủ đặt tủ rack truyền thống.
2. **Mô hình dịch vụ & Phân chia trách nhiệm (IaaS, PaaS, SaaS)**:
   Xác định ranh giới giữa những gì nhà cung cấp quản lý và những gì kỹ sư ứng dụng phải tự lo (hệ điều hành, bản vá bảo mật, runtime, dữ liệu). Chọn đúng tầng trừu tượng giúp cân bằng giữa tốc độ phát triển và mức độ kiểm soát.
3. **Mô hình triển khai & Hạ tầng vật lý (Public, Private, Hybrid, Data Center)**:
   Giải thích cách tài nguyên đám mây vận hành ngoài đời thực bên trong các trung tâm dữ liệu theo vùng (Region) và vùng sẵn sàng (AZ), cũng như cách cân bằng giữa bảo mật tuân thủ pháp lý với khả năng mở rộng ngang.
4. **Kinh tế đám mây & Mạng nền tảng (CapEx vs. OpEx, FinOps, VPC/DNS/TLS)**:
   Cung cấp tư duy định lượng về chi phí vận hành và năm khái niệm mạng cốt lõi để chuẩn bị trực tiếp cho các chương về hệ phân tán, container và Kubernetes phía sau.

## Điện toán đám mây là gì?

Theo Viện Tiêu chuẩn và Công nghệ Quốc gia Hoa Kỳ (National Institute of Standards and Technology — NIST), điện toán đám mây được định nghĩa như sau:

> "A model for enabling ubiquitous, convenient, on-demand network access to a shared pool of configurable computing resources (e.g., networks, servers, storage, applications, and services) that can be rapidly provisioned and released with minimal management effort or service provider interaction."

Tạm dịch: *Một mô hình cho phép truy cập mạng thuận tiện, mọi lúc, theo nhu cầu vào một nhóm tài nguyên tính toán dùng chung và có thể cấu hình (như mạng, máy chủ, lưu trữ, ứng dụng và dịch vụ), với khả năng cấp phát và thu hồi nhanh chóng cùng nỗ lực quản trị hoặc tương tác với nhà cung cấp ở mức tối thiểu.*

Định nghĩa này nêu bật bản chất của điện toán đám mây: tài nguyên tính toán được sử dụng linh hoạt như một dịch vụ tiện ích, giúp người dùng không phải trực tiếp xử lý sự phức tạp của hạ tầng vật lý bên dưới.

## Sự tiến hóa của các mô hình tính toán

Để hiểu rõ vai trò của điện toán đám mây, cần nhìn lại quá trình phát triển của các mô hình tính toán qua từng giai đoạn:

### 1. Thời đại máy tính lớn — Mainframe (những năm 1960–1980)
- Tính toán tập trung thông qua các thiết bị đầu cuối (terminal).
- Chi phí đầu tư cao, khả năng tiếp cận hạn chế.
- Chủ yếu sử dụng xử lý theo lô (batch processing) và hệ thống chia sẻ thời gian (time-sharing).

### 2. Thời đại máy tính cá nhân (những năm 1980–1990)
- Năng lực tính toán phân tán xuống từng máy trạm cá nhân.
- Phổ biến kiến trúc khách–chủ (client–server).
- Kết nối qua mạng cục bộ (LAN).

### 3. Thời đại Internet (những năm 1990–2000)
- Sự phát triển mạnh của các ứng dụng nền web.
- Hình thành các hệ thống phân tán và tính toán lưới (grid computing).
- Ra đời kiến trúc hướng dịch vụ (Service-Oriented Architecture — SOA).

### 4. Thời đại điện toán đám mây (những năm 2000–nay)
- Cấp phát tài nguyên theo nhu cầu (on-demand).
- Mô hình trả tiền theo mức sử dụng (pay-as-you-use).
- Khả năng mở rộng quy mô lớn và truy cập trên phạm vi toàn cầu.

## Vì sao điện toán đám mây quan trọng

Điện toán đám mây giải quyết nhiều thách thức cốt lõi của các tổ chức hiện đại:

### Hiệu quả kinh tế
- **Chuyển từ chi phí vốn (CapEx) sang chi phí vận hành (OpEx):** Doanh nghiệp thay thế khoản đầu tư phần cứng lớn ban đầu bằng chi phí định kỳ dễ dự báo hơn.
- **Lợi thế kinh tế theo quy mô (economy of scale):** Các nhà cung cấp đám mây vận hành hạ tầng quy mô rất lớn nên có thể cung cấp mức giá cạnh tranh.
- **Tối ưu hóa sử dụng tài nguyên:** Chỉ chi trả cho lượng tài nguyên thực tế sử dụng tại thời điểm có nhu cầu.

### Lợi thế công nghệ
- **Triển khai nhanh:** Ứng dụng và môi trường chạy có thể được thiết lập trong vài phút thay vì vài tháng mua sắm phần cứng.
- **Phạm vi toàn cầu:** Dịch vụ có thể mở rộng phục vụ người dùng ở nhiều khu vực địa lý với công sức vận hành thấp hơn.
- **Tập trung vào nghiệp vụ cốt lõi:** Đội ngũ kỹ thuật dành thời gian phát triển sản phẩm thay vì bảo trì hạ tầng vật lý.

### Tính linh hoạt trong kinh doanh (business agility)
- **Khả năng mở rộng (scalability):** Tăng hoặc giảm tài nguyên linh hoạt theo biến động tải.
- **Tính đa dạng công nghệ:** Hỗ trợ nhiều ngôn ngữ lập trình, khung làm việc và dịch vụ quản lý sẵn có.
- **Rút ngắn thời gian ra thị trường (speed to market):** Tăng tốc chu kỳ phát triển, thử nghiệm và phát hành sản phẩm.

### Khung giải thích (nên đọc chậm): Tính tay bài toán kinh tế CapEx so với OpEx cho cổng đăng ký học phần

Giả sử một trường đại học cần vận hành **Cổng đăng ký học phần** với đặc thù tải cực kỳ lệch: trong **10 ngày cao điểm mỗi năm** (2 kỳ đăng ký, mỗi kỳ 5 ngày), hệ thống cần tới **20 máy chủ** hoạt động hết công suất; còn **355 ngày còn lại** chỉ cần đúng **2 máy chủ** là dư sức phục vụ tra cứu điểm và lịch học. Hãy đặt lên bàn cân hai phương án bằng phép tính tay:

| Tiêu chí so sánh | Phương án A: Mua máy chủ tại chỗ (CapEx) | Phương án B: Đám mây đàn hồi (OpEx) |
| :--- | :---: | :---: |
| Số máy chủ phải trang bị / trả tiền | Mua cứng **20 máy chủ** cho đỉnh tải | **2 máy** chạy 365 ngày + **18 máy** chỉ bật đúng 10 ngày |
| Đơn giá giả định | 3.000 USD/máy (khấu hao 3 năm) + điện/chỗ đặt 200 USD/máy/năm | 0,10 USD/giờ/máy (~2,40 USD/ngày/máy) |
| **Tổng chi phí 1 năm** | $$20 \times (1.000 + 200) = \mathbf{24.000\text{ USD/năm}}$$ | $$(2 \times 365 + 18 \times 10) \times 2{,}40 = (730 + 180) \times 2{,}40 = \mathbf{2.184\text{ USD/năm}}$$ |
| Hiệu suất sử dụng trung bình | $$\frac{2 \times 355 + 20 \times 10}{20 \times 365} = \frac{910}{7.300} \approx \mathbf{12{,}5\%}$$ (lãng phí 87,5%!) | Gần **100%** khớp theo đường cong nhu cầu |

**Góc nhìn phản biện quan trọng:** Nếu ngược lại, bạn chạy một cụm tính toán ổn định **20 máy chủ chạy 100% tải suốt 365 ngày/năm** mà vẫn trả giá on-demand của đám mây ($$20 \times 365 \times 2{,}40 = 17.520\text{ USD/năm}$$), chênh lệch với tự đầu tư không còn lớn — lúc đó doanh nghiệp mới cần đến gói cam kết dài hạn (Reserved/Savings Plans) hoặc hạ tầng lai. Con số luôn trả lời trung thực hơn khẩu hiệu!

## Tác động trong thực tiễn

Điện toán đám mây là nền tảng cho nhiều mô hình vận hành và kinh doanh hiện đại:

- **Doanh nghiệp khởi nghiệp:** Các công ty như Netflix, Airbnb và Uber xây dựng toàn bộ nền tảng dịch vụ trên hạ tầng đám mây.
- **Chuyển đổi số doanh nghiệp truyền thống:** Các tập đoàn lâu năm như GE hay Capital One đã di chuyển nhiều khối lượng công việc trọng yếu lên đám mây.
- **Cộng tác phân tán:** Làm việc từ xa và phối hợp nhóm xuyên biên giới được hỗ trợ bởi các nền tảng cộng tác trực tuyến.
- **Phân tích dữ liệu và học máy:** Tạo điều kiện xử lý dữ liệu lớn (big data) và huấn luyện, vận hành mô hình học máy ở quy mô lớn.

## Mục tiêu học tập

Sau khi hoàn thành chương này, sinh viên có thể:

1. Trình bày năm đặc trưng cốt yếu định nghĩa điện toán đám mây.
2. Phân biệt các mô hình dịch vụ (IaaS, PaaS, SaaS) và tình huống áp dụng phù hợp.
3. So sánh các mô hình triển khai khác nhau cùng hệ quả về bảo mật, chi phí và quản trị.
4. Phân tích lợi ích và thách thức khi tổ chức áp dụng điện toán đám mây.
5. Xác định các yếu tố cần cân nhắc khi xây dựng chiến lược triển khai đám mây.

## Nội dung tiếp theo

Các bài học tiếp theo trong chương sẽ đi sâu vào từng khía cạnh kỹ thuật, mô hình triển khai và các yếu tố chiến lược, giúp bạn có cơ sở vững chắc để đánh giá và lựa chọn giải pháp đám mây phù hợp.

Việc học điện toán đám mây không chỉ dừng ở nắm bắt công cụ, mà còn là hiểu cách thiết kế, triển khai và vận hành ứng dụng hiệu quả trong môi trường phân tán.

## Bài ứng dụng tùy chọn

Phần lý thuyết bắt buộc của chương giữ nguyên cấu trúc chuẩn. Bài bổ sung về các xu hướng giai đoạn 2022–2026 (đa đám mây, FinOps, phục vụ mô hình AI) nằm tại [01-06 Ứng dụng đám mây hiện đại (2022–2026)]({{ site.baseurl }}{% multilang_post_url contents/chapter01/21-01-01-01_06_Modern_Cloud_Applications %}). Trước khi học về container (Chương 08) hoặc [Lộ trình triển khai]({{ site.baseurl }}/contents/vi/chapter15/), bạn nên đọc thêm bài tùy chọn [01-07 Nhập môn mạng đám mây]({{ site.baseurl }}{% multilang_post_url contents/chapter01/21-01-01-01_07_Networking_Crash_Course %}) về VPC, subnet, security group, DNS và TLS.

## Video tham khảo

- **Xem trước:** [Cloud Computing - Điện toán đám mây là gì? \| Hiểu rõ trong 5 phút \| TING3S](https://www.youtube.com/watch?v=5JLBwLilDVE) — TING 3S (tiếng Việt). Giới thiệu bức tranh tổng quan về điện toán đám mây bằng tiếng Việt, gắn với định nghĩa NIST ở đầu bài.
- [What is Cloud Computing? \| Amazon Web Services](https://www.youtube.com/watch?v=mxT233EdY5c) — Amazon Web Services (tiếng Anh). Đối chiếu khái niệm truy cập theo nhu cầu và trả phí theo mức sử dụng.
- [The Six Main Benefits of Cloud Computing with Amazon Web Services](https://www.youtube.com/watch?v=yMJ75k9X5_8) — Amazon Web Services (tiếng Anh). Kết nối phần tầm quan trọng của điện toán đám mây với sáu lợi ích tiêu biểu do AWS tổng hợp.
