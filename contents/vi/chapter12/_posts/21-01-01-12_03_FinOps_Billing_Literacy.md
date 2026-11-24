---
layout: post
title: 12-03 FinOps và đọc hóa đơn đám mây
chapter: '12'
order: 4
owner: Nguyen Le Linh
lang: vi
categories:
- chapter12
lesson_type: optional
---

Có một câu chuyện có thật xảy ra với vô số sinh viên và kỹ sư mới ra trường: tạo một cụm thử nghiệm trên đám mây để làm đồ án cuối kỳ, thấy mọi thứ chạy êm nên đóng laptop đi ngủ—và ba tuần sau nhận được hóa đơn thẻ tín dụng báo nợ **150 USD** chỉ vì một chiếc **NAT Gateway** hoặc một máy ảo GPU quên tắt vẫn đang chạy không tải 24/7.

Trên đám mây, **mọi quyết định kiến trúc đều là một quyết định tài chính**. **FinOps (Cloud Financial Operations)** không phải là việc kế toán cắt giảm chi tiêu mù quáng, mà là năng lực đọc hiểu đơn vị tính phí, dự báo chi phí trước khi bấm *Deploy*, và thiết lập vòng phản hồi tự động để mỗi đồng chi ra đều mang lại giá trị kỹ thuật rõ ràng.

## Bản đồ kiến thức nền: Bốn trục tư duy đọc hiểu hóa đơn đám mây (FinOps)

1. **Bốn đơn vị đo lường cốt lõi của Cloud Billing**:
   Hóa đơn đám mây quy mọi tài nguyên về bốn nhóm đồng hồ đếm: **Tính toán** (`vCPU-giờ`, `GB-RAM-giờ`, `GB-giây` cho serverless), **Lưu trữ** (`GB-tháng` + số lượng thao tác `IOPS`/`PUT`/`GET`), **Mạng** (`GB` truyền ra ngoài *egress*, truyền xuyên Availability Zone, và phí giữ IP công khai/NAT Gateway theo giờ), và **Dịch vụ quản lý** (Managed DB, Load Balancer, kho log).

2. **Mô hình định giá (On-Demand, Reserved/Savings Plans, Spot, Serverless)**:
   **On-Demand** linh hoạt tối đa nhưng đắt nhất. **Savings Plans / Reserved** giảm $30\%\text{–}60\%$ khi cam kết dùng ổn định 1–3 năm. **Spot / Preemptible** giảm tới $70\%\text{–}90\%$ cho tác vụ chịu được thu hồi bất ngờ (batch, CI runner). **Serverless / Scale-to-Zero** tính theo mili-giây thực chạy—lý tưởng cho đồ án sinh viên có lưu lượng thấp và ngắt quãng.

3. **Các "kẻ đốt tiền thầm lặng" (Hidden Cost Traps)**:
   Chi phí bất ngờ thường không đến từ máy chủ chính mà từ: ổ đĩa rời (*unattached EBS/Persistent Disk*) bị bỏ quên sau khi xóa VM, **NAT Gateway** tính phí theo giờ lẫn mỗi GB đi qua, lưu lượng **Cross-AZ** giữa hai máy nằm khác vùng sẵn sàng, và log không có chính sách xóa tự động (*retention policy*).

4. **Phân bổ chi phí bằng Nhãn (Tagging) & Hạn mức cảnh báo (Budget Alerts)**:
   Nguyên tắc số 1 của FinOps: mọi tài nguyên sinh ra phải gắn nhãn (`project`, `env`, `owner`) và đặt **Budget Alert** ở các ngưỡng $50\%$, $80\%$, $100\%$ ngân sách ngay ngày đầu mở tài khoản.

## Mục tiêu học tập

- Đọc hiểu hóa đơn đám mây theo công thức **số lượng đo × đơn giá × thời gian**.
- Thiết lập quy ước gắn thẻ (tagging) để trả lời được câu hỏi: "Hệ thống Bảng tin học phần tuần này tốn bao nhiêu chi phí?"
- Tạo **ngân sách và cảnh báo chi tiêu** *trước* lệnh `apply` đầu tiên, đồng thời xem bước thu hồi tài nguyên (`destroy`) là điều kiện bắt buộc để hoàn thành bài thực hành.
- Đánh giá mô hình giá on-demand, cam kết dài hạn và spot dưới góc độ *mức độ rủi ro* thay vì chỉ nhìn vào tỷ lệ chiết khấu.

## 1. Các đơn vị đo lường chi phí đám mây

| Nhóm đơn vị đo | Ví dụ cụ thể | Lý do sinh viên thường bỏ sót |
| --- | --- | --- |
| Giờ tính toán và mạng | Máy ảo, bộ cân bằng tải, NAT Gateway | Đã tắt máy ảo nhưng quên xóa NAT Gateway |
| GB-tháng lưu trữ | Ổ đĩa, object storage, snapshot, log | Snapshot vẫn tồn tại sau khi xóa máy ảo |
| Thao tác I/O và số lượt gọi | Lệnh PUT object, IOPS đĩa | Cơ chế health check gửi yêu cầu quá dày |
| Lưu lượng ra ngoài (egress) | Dữ liệu đi ra khỏi vùng hoặc ra Internet | Nhiều người tải tệp dung lượng lớn từ mạng ngoài |
| Tài nguyên phụ trợ quản lý | Địa chỉ IPv4 công khai, container registry | IPv4 công khai đã được tách thành mục tính phí riêng trong thập niên 2020 |

$$
C = \sum_i (q_i \cdot p_i), \quad u = C / N_{\text{success}}
$$

Nếu $$N_{\text{success}} = 0$$, hệ thống không có chi phí đơn vị có ý nghĩa mà chỉ đang phát sinh chi phí cho tài nguyên nhàn rỗi.

```python
from dataclasses import dataclass

@dataclass
class Line:
    name: str
    quantity: float
    unit_price: float

    def amount(self) -> float:
        return self.quantity * self.unit_price

def invoice_total(lines: list[Line]) -> float:
    return sum(line.amount() for line in lines)
```

Mọi số liệu trong bài chỉ mang tính **minh họa**; khi ước tính thực tế, luôn kiểm tra trực tiếp trang bảng giá hiện hành của nhà cung cấp.

## 2. Quy trình FinOps ở quy mô thực hành sinh viên

Theo FinOps Foundation, vòng đời FinOps gồm ba giai đoạn: **Inform → Optimize → Operate**. Áp dụng vào thực hành tại IUH:

1. **Inform (Minh bạch):** Gắn thẻ tài nguyên đầy đủ, thiết lập ngưỡng ngân sách và chụp báo cáo chi phí theo thẻ hằng tuần.
2. **Optimize (Tối ưu):** Tắt tài nguyên không sử dụng; ưu tiên chạy Docker cục bộ khi việc triển khai lên đám mây không phục vụ thêm chuẩn đầu ra nào.
3. **Operate (Vận hành):** Lên lịch dọn dẹp (`destroy`) cố định và phân công rõ một thành viên theo dõi email cảnh báo ngân sách.

```text
project=iuh-courseboard
pair=<mssv-hoac-ten-nhom>
env=lab|staging|prod
owner=<github-handle>
```

<div class="content-box warning-box">
<p><strong>Thiết lập ngân sách trước, khởi tạo tài nguyên sau.</strong> Nếu tài khoản chưa bật cảnh báo ngân sách (budget alert), không nên chạy <code>apply</code> đối với các tài nguyên có tính phí. Hãy sử dụng Lab A trên máy cục bộ trước.</p>
</div>

## 3. Các mô hình định giá

| Mô hình giá | Bản chất cam kết | Phù hợp với bài lab hai tuần? |
| --- | --- | --- |
| **On-demand** | Trả theo giờ hoặc giây sử dụng | Mặc định nên dùng vì dễ tạo và xóa ngay. |
| **Cam kết / Reserved** | Giảm giá khi cam kết 1–3 năm | Hầu như không phù hợp vì vẫn phải trả phí sau khi kết thúc học kỳ. |
| **Spot / Preemptible** | Tận dụng công suất dư, có thể bị thu hồi đột ngột | Chỉ phù hợp cho tác vụ lô; rủi ro cao vào ngày chấm bài. |

**Free tier** có thể xem như một dạng chính sách thứ tư với hạn mức và danh mục sản phẩm cụ thể — nó chỉ miễn phí trên *một số* mục đo trong giới hạn nhất định chứ không bao trùm toàn bộ dịch vụ.

## 4. Những khoản chi phí dễ gây bất ngờ

1. **NAT Gateway:** Tính phí đồng thời theo giờ hoạt động *và* theo dung lượng GB đi qua.
2. **Bộ cân bằng tải (load balancer):** Vẫn tiếp tục tính phí theo giờ ngay cả khi đã xóa máy chủ phía sau.
3. **Địa chỉ IPv4 công khai:** Được tính phí riêng theo giờ.
4. **Chi phí nạp dữ liệu quan sát:** Dung lượng GB log và số lượng span tăng nhanh nếu để chế độ ghi log debug liên tục (xem bài 13-03).
5. **Phí egress từ object storage:** Tăng nhanh khi trình diễn hoặc tải dữ liệu lớn qua mạng di động/Internet ngoài vùng.
6. **Snapshot và AMI:** Không tự bị xóa khi chạy `destroy` nếu được tạo thủ công ngoài cấu hình IaC.
7. **GPU và API AI quản lý:** Không nên đưa vào đồ án mặc định nếu không có yêu cầu cụ thể và kiểm soát ngân sách.

Nguyên tắc đơn giản: bất kỳ mục tính phí nào không phục vụ trực tiếp chuẩn đầu ra của bài học thì **không nên bật**.

## 5. Tám câu hỏi thiết kế cân bằng quy mô và chi phí

1. Đơn vị công việc là gì và mục tiêu chi phí đơn vị $$u$$ ra sao?
2. Thành phần nào thực sự *cần* tự động co giãn đàn hồi?
3. Khi lưu lượng **bằng không**, hệ thống vẫn tốn bao nhiêu tiền mỗi ngày?
4. Dữ liệu egress đi qua những ranh giới tính phí nào?
5. Chuẩn đầu ra của bài có thực sự đòi hỏi triển khai trên **hai vùng sẵn sàng (AZ)** hay không?
6. Chi phí thuê cơ sở dữ liệu quản lý có hợp lý hơn công sức tự vá lỗi PostgreSQL (và chi phí thời gian nhàn rỗi) hay không?
7. Thứ tự chạy lệnh `destroy` như thế nào và ai chịu trách nhiệm kiểm tra sau buổi demo?
8. Cả hai thành viên trong nhóm có cùng nhận được email cảnh báo ngân sách hay không?

## 6. Quy tắc sử dụng tài khoản dùng chung

- Tuyệt đối không khai thác tiền mã hóa, không huấn luyện mô hình lớn và không mở kho lưu trữ tệp công khai trên hạn mức tài khoản của lớp.
- Không chia sẻ thông tin đăng nhập tài khoản root trong nhóm chat.
- Luôn thao tác đúng trong project hoặc subscription được cấp.
- Ghi rõ bảng ước lượng chi phí trong báo cáo đồ án, kể cả khi chi phí thực tế bằng $$0$$ do chạy hoàn toàn trên máy cục bộ.

### Khung giải thích (nên đọc chậm): Tính tay hóa đơn 1 tháng của đồ án sinh viên — Kiến trúc "quên tối ưu" vs. Kiến trúc FinOps

Giả sử nhóm hai sinh viên triển khai ứng dụng web Bảng tin học phần có khoảng **200 lượt truy cập/ngày** trong suốt 1 tháng (`730 giờ`). Hãy so sánh hai cấu hình chạy cùng một ứng dụng trên đám mây (đơn giá minh họa sát thực tế AWS/GCP/Azure):

| Hạng mục tài nguyên | Cấu hình A (Dùng máy ảo + NAT Gateway 24/7) | Cấu hình B (FinOps: Scale-to-Zero + Managed Free Tier) |
| :--- | :--- | :--- |
| **Compute (Ứng dụng web)** | 1 VM `2 vCPU, 4 GB RAM` chạy 24/7: $730 \times \$0{,}042 = \mathbf{\$30{,}66}$ | Container Serverless (Cloud Run / App Runner), chỉ chạy thực 15 giờ/tháng → **$\$0{,}00$** (trong hạn mức miễn phí) |
| **NAT Gateway** | 1 NAT Gateway chạy 24/7: $730 \times \$0{,}045 = \mathbf{\$32{,}85}$ (+ `$0,045/GB`) | Không dùng NAT Gateway (dùng Public Subnet an toàn + Security Group chặt) → **$\$0{,}00$** |
| **Cơ sở dữ liệu** | Managed Postgres `db.t3.small` 24/7 + 50 GB đĩa: `~$28,00` | Postgres tier nhỏ/tự động ngủ hoặc chia sẻ trong lớp + 10 GB đĩa → **$\$0{,}00\text{–}\$5{,}00$** |
| **Ổ đĩa & Snapshot bỏ quên** | 2 ổ 100 GB quên xóa sau khi thử nghiệm: $200 \times \$0{,}10 = \mathbf{\$20{,}00}$ | Tự động dọn bằng `tofu destroy` + lifecycle 7 ngày → **$\$0{,}00$** |
| **Log không đặt retention** | In `DEBUG` log 20 GB/tháng lưu vĩnh viễn: `~$10,00` | Chỉ ghi `INFO/ERROR`, retention 14 ngày (`< 1 GB`) → **$\$0{,}00$** |
| **TỔNG HÓA ĐƠN / THÁNG** | **$\approx \$121{,}51$ (~3.000.000 VNĐ/tháng!)** | **$\$0{,}00\text{–}\$5{,}00$ (~0–125.000 VNĐ/tháng)** |

Con số biết nói: hai kiến trúc phục vụ **cùng một lượng người dùng** nhưng chênh nhau hơn **$24\times$ chi phí**, trong đó riêng một chiếc NAT Gateway chạy không tải đã ngốn gần **$33/tháng**—đắt hơn cả máy chủ chạy ứng dụng!

## Thách thức

- Bảng giá đám mây thay đổi thường xuyên; số liệu trong một bài viết từ năm 2022 không còn giá trị làm báo giá.
- Nhà cung cấp luôn đo dung lượng lưu trữ đã cấp phát; việc quên xóa ổ đĩa vẫn tạo hóa đơn.
- Thao tác thủ công trên giao diện web (ClickOps) dễ làm thiếu thẻ gắn phân loại chi phí (xem Chương 14).
- Thuế và biến động tỷ giá thường bị bỏ sót khi ước tính ngân sách ban đầu.

## Video tham khảo

- **Xem trước:** [What is FinOps? The Operating Model & Cultural Practice to Maximize the Value of Technology](https://www.youtube.com/watch?v=Y-c_xw9bHFw) — FinOps Foundation (tiếng Anh). Định nghĩa mô hình FinOps từ FinOps Foundation, được thu gọn xuống quy mô bài toán hóa đơn sinh viên trong bài này.
- [What is FinOps and 3 reasons why you should care about it](https://www.youtube.com/watch?v=hP5rhIcx9Ck) — Google Cloud Tech (tiếng Anh). Ba lý do thực tiễn để áp dụng FinOps trong vòng phản hồi chi phí.
- [AWS Cost Optimisation Series: Cost Explorer \| Amazon Web Services](https://www.youtube.com/watch?v=oE8TNKGmc40) — Amazon Web Services (tiếng Anh). Hướng dẫn sử dụng AWS Cost Explorer — công cụ theo dõi chi phí cần mở trước khi chạy thử nghiệm free-tier.

## Bài tập

1. Bổ sung một NAT Gateway bị quên chạy suốt 200 giờ vào hàm `invoice_total`. Với $$N_{\text{success}} = 40$$, chi phí đơn vị lúc này phản ánh điều gì?
2. Lập danh sách kiểm tra (checklist) dọn dẹp cuối buổi thực hành gồm snapshot, địa chỉ IP công khai và log group.
3. Giả sử một thành viên trong nhóm đề xuất mua gói cam kết 1 năm (reserved) cho đồ án kéo dài hai tuần; hãy dùng bảng ở Mục 3 để phản biện.
4. Ước tính chi phí nhàn rỗi trong 14 ngày cho cấu hình gồm một máy ảo nhỏ, một ổ đĩa lưu trữ và một bộ cân bằng tải không có lưu lượng, ghi rõ ngày tra cứu bảng giá.

## Đọc tiếp

- [01-06]({{ site.baseurl }}{% multilang_post_url contents/chapter01/21-01-01-01_06_Modern_Cloud_Applications %})
- [Lab C]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_03_Lab_Free_Tier_Notes %})
- [Đồ án]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_04_Capstone %})

## Tài liệu tham khảo

1. FinOps Foundation, *FinOps Framework*.
2. Trang bảng giá và chính sách free-tier của AWS, Azure, GCP (kiểm tra tại đúng ngày ước tính).
3. Flexera *State of the Cloud* (xem bài 01-06).
4. [infracourse.cloud](https://infracourse.cloud/) — nguồn tham khảo cách tiếp cận; các quy tắc và ví dụ cho IUH được biên soạn độc lập.
