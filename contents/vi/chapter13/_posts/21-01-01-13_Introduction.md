---
layout: post
title: 13 Giới thiệu — bài ứng dụng tùy chọn
chapter: '13'
order: 1
owner: Nguyen Le Linh
lang: vi
categories:
- chapter13
---

Viết xong tính năng mới trên máy cá nhân mới chỉ đi được nửa chặng đường. Nửa chặng đường còn lại—làm sao đưa mã nguồn đó lên hệ thống đang có hàng nghìn người dùng mà **không làm gián đoạn dịch vụ**, bảo vệ dữ liệu bằng phân quyền tối thiểu và mã hóa, đồng thời phát hiện sự cố trong vài giây trước khi khách hàng kịp gọi điện báo lỗi—mới là thước đo của một kỹ sư hệ thống đám mây thực thụ.

Chương 13 kết nối ba trụ cột quan trọng của vận hành hiện đại: **Bảo mật (IAM, mã hóa, chuỗi cung ứng)**, **Triển khai an toàn (CI/CD, Blue/Green, Canary)** và **Khả năng quan sát (Observability)**.

## Kiến thức cần nhớ: Bốn lá chắn của triển khai và bảo mật đám mây

1. **Nguyên tắc Đặc quyền tối thiểu (Least Privilege) & IAM**:
   Mỗi con người và mỗi microservice chỉ được cấp đúng quyền tối thiểu cần thiết để làm nhiệm vụ của nó (ví dụ dịch vụ đọc ảnh chỉ có quyền `s3:GetObject` trên đúng 1 bucket, tuyệt đối không cấp `AdministratorAccess` hay `s3:*`).

2. **Mã hóa khi truyền (In Transit — TLS) & Khi lưu (At Rest — KMS)**:
   Mọi kết nối mạng đều phải bọc TLS 1.3 để chống nghe lén trên đường truyền, còn mọi ổ đĩa, cơ sở dữ liệu và bản sao lưu đều mã hóa bằng khóa quản lý tập trung (KMS) để ổ cứng bị tháo trộm cũng chỉ là tập byte vô nghĩa.

3. **Chiến lược triển khai không gián đoạn (Rolling, Blue/Green, Canary)**:
   Thay vì tắt toàn bộ máy chủ cũ rồi bật máy mới (*Recreate* gây downtime), hệ thống đám mây thay thế dần từng Pod (*Rolling*), dựng sẵn môi trường song song rồi chuyển mạch tức thì (*Blue/Green*), hoặc chỉ mở cho $5\%$ người dùng thử trước (*Canary*).

4. **Vòng phản hồi Giám sát → Cảnh báo → Quay lui (Rollback)**:
   Triển khai chỉ thực sự an toàn khi gắn với ngưỡng cảnh báo rõ ràng (tỷ lệ lỗi `5xx`, độ trễ `p99`) và khả năng quay lui về phiên bản cũ chỉ bằng một cú bấm trong dưới 60 giây.

### Giải thích: So sánh số request bị lỗi khi bản phát hành `v2` chứa lỗi cấu hình trên 3 chiến lược triển khai

Giả sử ứng dụng đang phục vụ ổn định **`100 request/giây`** trên **4 Pod `v1`**. Vô tình bản `v2` có một lỗi thiếu biến môi trường làm 100% request rơi vào `v2` trả về `500 Internal Server Error`, và hệ thống phát hiện + rollback sau đúng **`10 giây`**. Hãy tính số request bị ảnh hưởng ở từng chiến lược:

| Chiến lược triển khai | Tỷ lệ lưu lượng rơi vào bản lỗi `v2` trong 10 giây | Số request bị lỗi `500` | Thời gian quay lui (Rollback) |
| :--- | :---: | :---: | :--- |
| **1. Recreate** (tắt hết 4 Pod `v1` rồi mới bật `v2`) | $100\%$ (kèm downtime khởi động) | $\ge \mathbf{1.000\text{ requests}}$ (toàn bộ người dùng bị sập) | Chậm (phải tắt `v2` rồi khởi động lại 4 Pod `v1`) |
| **2. Rolling Update (`maxSurge: 1`, không có Readiness Probe chặt)** | $25\%$ (1/4 Pod là `v2`) | $10 \times 100 \times 0{,}25 = \mathbf{250\text{ requests}}$ | Trung bình (chạy rolling ngược về `v1`) |
| **3. Canary `5%` (hoặc Blue/Green có Smoke Test trước khi chuyển mạch)** | **$5\%$** (Canary) hoặc **$0\%$** (Blue/Green chặn ngay ở Smoke Test!) | **$50\text{ requests}$** (Canary) hoặc **$0\text{ requests}$** (Blue/Green) | **Tức thì ($< 5\text{ giây}$)** — chỉ cần đổi trọng số về `0%` / chuyển router về Blue! |

Bốn con số **`1.000` → `250` → `50` → `0` request lỗi** cho thấy rất rõ vì sao chiến lược triển khai kết hợp kiểm tra sức khỏe tự động là khoản đầu tư rẻ nhất để bảo vệ uy tín hệ thống.

> **Lưu ý lộ trình:** Phần lý thuyết bắt buộc của **Chương 13 (Triển khai và Bảo mật)** hiện được trình bày bằng tiếng Anh. Lộ trình tiếng Việt ở chương này cung cấp ba bài học ứng dụng và chuyên sâu tùy chọn:
> 
> - [13-02 Bảo mật chuỗi cung ứng, SLSA và quan sát production]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_02_Supply_Chain_Security_and_Observability %})
> - [13-03 Quan sát sâu — log, metric, trace, OpenTelemetry]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_03_Observability_Depth %})
> - [13-04 CI/CD cho ứng dụng đám mây]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_04_CICD_Cloud_Apps %})
> - [Lộ trình triển khai]({{ site.baseurl }}/contents/vi/chapter15/)
> - [Bản tiếng Anh của chương]({{ site.baseurl }}/contents/en/chapter13/)

## Video tham khảo

- **Xem trước:** [CI/CD Explained in 5 Minutes](https://www.youtube.com/watch?v=ddDJxFnv-qs) — TechWorld with Nana (tiếng Anh). Tóm tắt quy trình CI/CD trong năm phút, giúp tự động hóa phát hành và giảm sai sót thủ công.
- [SLSA: Industry-Driven Guidelines for Software Supply Chain Security \| OpenSSF Project Spotlight](https://www.youtube.com/watch?v=gdYlSuH5Srs) — OpenSSF (tiếng Anh). Giới thiệu khung bảo mật chuỗi cung ứng phần mềm SLSA được trình bày trong bài 13-02.
- [OpenTelemetry: OpenTelemetry in Five Minutes \| Project Lightning Talk](https://www.youtube.com/watch?v=l8xiNOCIdLY) — CNCF [Cloud Native Computing Foundation] (tiếng Anh). Giới thiệu OpenTelemetry trong năm phút, nền tảng cho bài 13-03.
