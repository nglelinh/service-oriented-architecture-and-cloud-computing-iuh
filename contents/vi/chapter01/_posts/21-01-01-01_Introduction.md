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

Điện toán đám mây (cloud computing) đã thay đổi căn bản cách tổ chức tiếp cận hạ tầng CNTT, triển khai ứng dụng và cung cấp dịch vụ. Sự chuyển đổi mô hình này là một trong những tiến bộ công nghệ quan trọng nhất của thế kỷ 21: doanh nghiệp có thể sử dụng tài nguyên tính toán theo nhu cầu mà không cần đầu tư lớn trước cho phần cứng và hạ tầng.

## Điện toán đám mây là gì?

Theo Viện Tiêu chuẩn và Công nghệ Quốc gia Hoa Kỳ (National Institute of Standards and Technology, NIST), điện toán đám mây được định nghĩa như sau:

> "A model for enabling ubiquitous, convenient, on-demand network access to a shared pool of configurable computing resources (e.g., networks, servers, storage, applications, and services) that can be rapidly provisioned and released with minimal management effort or service provider interaction."

Tạm dịch: *Một mô hình cho phép truy cập mạng mọi lúc, thuận tiện, theo nhu cầu tới một nhóm tài nguyên tính toán dùng chung và có thể cấu hình (ví dụ: mạng, máy chủ, lưu trữ, ứng dụng và dịch vụ), vốn có thể được cấp phát và thu hồi nhanh với nỗ lực quản trị và tương tác nhà cung cấp ở mức tối thiểu.*

Định nghĩa này nắm bản chất của điện toán đám mây: khả năng sử dụng tài nguyên tính toán dễ như bật công tắc đèn, mà không phải lo phức tạp của hạ tầng bên dưới.

## Sự tiến hóa của các mô hình tính toán

Để hiểu ý nghĩa của điện toán đám mây, cần nhìn lại sự tiến hóa của các mô hình tính toán:

### 1. Thời đại máy tính lớn — Mainframe (những năm 1960–1980)
- Tính toán tập trung với các thiết bị đầu cuối (terminal)
- Chi phí cao, khả năng tiếp cận hạn chế
- Xử lý theo lô (batch processing) và hệ thống chia sẻ thời gian (time-sharing)

### 2. Thời đại máy tính cá nhân (những năm 1980–1990)
- Tính toán phân tán trên từng máy
- Kiến trúc khách–chủ (client–server)
- Mạng cục bộ (LAN)

### 3. Thời đại Internet (những năm 1990–2000)
- Ứng dụng dựa trên web
- Hệ thống phân tán và tính toán lưới (grid computing)
- Kiến trúc hướng dịch vụ (Service-Oriented Architecture, SOA)

### 4. Thời đại điện toán đám mây (những năm 2000–nay)
- Cấp phát tài nguyên theo nhu cầu (on-demand)
- Mô hình trả tiền theo mức sử dụng (pay-as-you-use)
- Khả năng mở rộng quy mô lớn và truy cập toàn cầu

## Vì sao điện toán đám mây quan trọng

Điện toán đám mây giải quyết nhiều thách thức then chốt của tổ chức hiện đại:

### Hiệu quả kinh tế
- **Từ chi phí vốn (Capital Expenditure, CapEx) sang chi phí vận hành (Operational Expenditure, OpEx)**: Tổ chức chuyển từ đầu tư lớn ban đầu sang chi phí hàng tháng có thể dự đoán
- **Lợi thế kinh tế theo quy mô (economy of scale)**: Nhà cung cấp đám mây có thể chào giá thấp hơn nhờ vận hành ở quy mô rất lớn
- **Tối ưu tài nguyên**: Chỉ trả cho những gì sử dụng, vào thời điểm sử dụng

### Lợi thế công nghệ
- **Triển khai nhanh**: Ứng dụng có thể được triển khai trong vài phút thay vì vài tháng
- **Phạm vi toàn cầu**: Dịch vụ có thể được cung cấp trên toàn thế giới với nỗ lực tối thiểu
- **Tăng tốc đổi mới**: Tập trung vào logic nghiệp vụ cốt lõi thay vì quản trị hạ tầng

### Sự linh hoạt trong kinh doanh (business agility)
- **Khả năng mở rộng (scalability)**: Tài nguyên có thể tăng hoặc giảm theo nhu cầu
- **Tính linh hoạt**: Hỗ trợ nhiều ngôn ngữ lập trình, khung làm việc và công cụ
- **Tốc độ ra thị trường (speed to market)**: Chu kỳ phát triển và triển khai nhanh hơn

## Tác động trong thực tiễn

Điện toán đám mây đã mở đường cho nhiều đổi mới và mô hình kinh doanh:

- **Khởi nghiệp**: Các công ty như Netflix, Airbnb và Uber xây dựng toàn bộ nền tảng trên hạ tầng đám mây
- **Chuyển đổi doanh nghiệp**: Các tổ chức truyền thống như GE và Capital One đã chuyển khối lượng công việc then chốt lên đám mây
- **Hợp tác toàn cầu**: Làm việc từ xa và nhóm phân tán được hỗ trợ bởi các công cụ cộng tác trên đám mây
- **Phân tích dữ liệu**: Xử lý dữ liệu lớn (big data) và học máy (machine learning) ở quy mô lớn

## Mục tiêu học tập

Kết thúc chương này, sinh viên sẽ hiểu:

1. Các đặc trưng cốt yếu định nghĩa điện toán đám mây
2. Các mô hình dịch vụ (IaaS, PaaS, SaaS) và tình huống sử dụng
3. Các mô hình triển khai khác nhau và hệ quả của chúng
4. Lợi ích và thách thức khi áp dụng đám mây
5. Những cân nhắc then chốt khi xây dựng chiến lược và triển khai đám mây

## Bài tiếp theo?

Các bài sau sẽ đi sâu từng khía cạnh của điện toán đám mây: chi tiết kỹ thuật, triển khai thực tế và các cân nhắc chiến lược giúp bạn ra quyết định có cơ sở về việc áp dụng và sử dụng đám mây.

Hành trình điện toán đám mây không chỉ là hiểu công nghệ — mà còn là tái hình dung cách chúng ta xây dựng, triển khai và quản lý ứng dụng trong một thế giới ngày càng kết nối và số hóa.

## Bài ứng dụng tùy chọn

Phần lý thuyết bắt buộc của chương không đổi. Bài bổ sung về ứng dụng 2022–2026 (đa đám mây, FinOps, phục vụ mô hình AI) nằm ở [01-06 Ứng dụng đám mây hiện đại (2022–2026)]({% multilang_post_url contents/chapter01/21-01-01-01_06_Modern_Cloud_Applications %}).
