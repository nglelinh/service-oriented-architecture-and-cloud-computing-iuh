---
layout: post
title: 12 Giới thiệu — bài ứng dụng tùy chọn
chapter: '12'
order: 1
owner: Nguyen Le Linh
lang: vi
categories:
- chapter12
---

Mở trang quản trị của AWS, Microsoft Azure hay Google Cloud Platform lần đầu tiên, hầu hết sinh viên đều choáng ngợp trước hơn 200 tên dịch vụ dài dằng dặc. Nhưng nếu biết cách nhìn xuyên qua lớp tên gọi tiếp thị, bạn sẽ nhận ra một bí mật thú vị: **cả ba nhà cung cấp lớn đều xây dựng trên đúng một bản thiết kế hạ tầng vật lý giống hệt nhau**—từ Region, Availability Zone, Compute, Object Storage cho đến IAM.

Chương 12 giúp bạn xây dựng "từ điển ánh xạ" giữa AWS, Azure và GCP, hiểu rõ ý nghĩa thực sự của các cam kết mức độ sẵn sàng (SLA) và rèn tư duy quản trị chi phí FinOps ngay từ khi còn ngồi trên ghế nhà trường.

## Kiến thức cần nhớ: Bốn khái niệm phổ quát trên mọi nhà cung cấp đám mây

1. **Vùng địa lý (Region) & Vùng sẵn sàng (Availability Zone — AZ)**:
   Một **Region** (ví dụ Singapore `ap-southeast-1`) là một khu vực địa lý chứa từ 3 **Availability Zone (AZ)** trở lên. Mỗi AZ là một hoặc nhiều trung tâm dữ liệu vật lý độc lập hoàn toàn về nguồn điện, hệ thống làm mát và đường cáp quang, nối với nhau bằng mạng sợi quang tốc độ cao độ trễ $< 2\text{ ms}$ để nhân bản dữ liệu đồng bộ.

2. **Bản đồ tương đương dịch vụ cốt lõi (AWS ↔ Azure ↔ GCP)**:
   - Máy ảo (Compute): `EC2` ↔ `Azure VMs` ↔ `Compute Engine`.
   - Lưu trữ đối tượng (Object Storage): `S3` ↔ `Blob Storage` ↔ `Cloud Storage (GCS)`.
   - Kubernetes quản lý: `EKS` ↔ `AKS` ↔ `GKE`.
   - Hàm không máy chủ (Serverless): `Lambda` ↔ `Azure Functions` ↔ `Cloud Run Functions`.

3. **Mô hình Trách nhiệm chia sẻ (Shared Responsibility Model)**:
   Nhà cung cấp đám mây chịu trách nhiệm **bảo mật *của* đám mây** (an ninh vật lý trung tâm dữ liệu, phần cứng, hypervisor). Khách hàng (bạn!) chịu trách nhiệm **bảo mật *trên* đám mây** (phân quyền IAM, mã hóa dữ liệu, vá lỗi hệ điều hành trong VM, không để bucket S3 công khai nhầm).

4. **Toán học của Độ sẵn sàng (SLA & "Số chín")**:
   Một hệ thống gồm nhiều thành phần mắc nối tiếp có độ sẵn sàng tổng hợp là **tích** của các thành phần; còn các thành phần dự phòng song song (Multi-AZ) giúp tăng độ sẵn sàng lên vượt bậc.

### Giải thích: Tính tay SLA tổng hợp và số phút ngừng hoạt động (downtime) mỗi năm

Giả sử ứng dụng thương mại điện tử của bạn cần đi qua **3 tầng mắc nối tiếp**:
1. **Load Balancer** (SLA $99{,}99\% = 0{,}9999$)
2. **Tầng Ứng dụng** (chạy 1 máy ảo đơn lẻ ở 1 AZ: SLA $99{,}5\% = 0{,}995$)
3. **Cơ sở dữ liệu** (Managed DB đơn vùng: SLA $99{,}9\% = 0{,}999$)

#### Trường hợp 1: Tầng Ứng dụng chạy đơn lẻ (Single-AZ)
Vì cả 3 tầng phải cùng sống thì người dùng mới mua hàng được:
$$A_{\text{tổng}} = 0{,}9999 \times 0{,}995 \times 0{,}999 = 0{,}993905 \approx \mathbf{99{,}39\%}.$$
Một năm có $365 \times 24 \times 60 = 525.600\text{ phút}$, nên thời gian gián đoạn kỳ vọng mỗi năm là:
$$\text{Downtime}_1 = (1 - 0{,}993905) \times 525.600 \approx \mathbf{3.203\text{ phút}} \approx \mathbf{53{,}4\text{ giờ/năm}}!$$

#### Trường hợp 2: Nhân bản Tầng Ứng dụng song song trên 2 AZ độc lập
Hai cụm App chạy song song chỉ cùng chết khi *cả hai* cùng lỗi đồng thời:
$$A_{\text{App, 2-AZ}} = 1 - (1 - 0{,}995)^2 = 1 - (0{,}005)^2 = 1 - 0{,}000025 = \mathbf{99{,}9975\%}.$$
Khi kết hợp với DB Multi-AZ ($99{,}95\%$):
$$A_{\text{tổng, mới}} = 0{,}9999 \times 0{,}999975 \times 0{,}9995 \approx \mathbf{99{,}9375\%} \implies \text{Downtime}_2 \approx \mathbf{328\text{ phút}} \approx \mathbf{5{,}5\text{ giờ/năm}}.$$

Chỉ bằng một phép nhân xác suất đơn giản, bạn thấy ngay vì sao thiết kế **Multi-AZ** cắt giảm thời gian ngừng hệ thống gần **$10\times$** và vì sao độ sẵn sàng toàn hệ thống luôn bị kéo tụt bởi mắt xích nối tiếp yếu nhất!

> **Lưu ý lộ trình:** Phần lý thuyết bắt buộc của **Chương 12 (Các nhà cung cấp đám mây)** hiện được trình bày bằng tiếng Anh. Lộ trình tiếng Việt ở chương này cung cấp bài đọc ứng dụng và bài thực hành FinOps tùy chọn.
> 
> - [12-02 Kiến trúc đa đám mây và dịch vụ AI quản lý]({{ site.baseurl }}{% multilang_post_url contents/chapter12/21-01-01-12_02_Multicloud_and_AI_Services %})
> - [12-03 FinOps và đọc hóa đơn đám mây]({{ site.baseurl }}{% multilang_post_url contents/chapter12/21-01-01-12_03_FinOps_Billing_Literacy %})
> - [Lộ trình triển khai]({{ site.baseurl }}/contents/vi/chapter15/)
> - [Bản tiếng Anh của chương]({{ site.baseurl }}/contents/en/chapter12/)

## Video tham khảo

- **Xem trước:** [What is AWS? \| Amazon Web Services](https://www.youtube.com/watch?v=a9__D53WsUs) — Amazon Web Services (tiếng Anh). Giới thiệu tổng quan về AWS — nhà cung cấp đầu tiên trong nhóm ba nền tảng lớn của chương.
- [How does Microsoft Azure work?](https://www.youtube.com/watch?v=KXkBZCe699A) — Microsoft Azure (tiếng Anh). Cách Microsoft Azure tổ chức các khái niệm và dịch vụ đám mây.
- [What is Google Cloud?](https://www.youtube.com/watch?v=kzKFuHk8ovk) — Google Cloud Tech (tiếng Anh). Tổng quan về Google Cloud, hoàn thiện bộ ba nhà cung cấp để đối chiếu trong chương.
