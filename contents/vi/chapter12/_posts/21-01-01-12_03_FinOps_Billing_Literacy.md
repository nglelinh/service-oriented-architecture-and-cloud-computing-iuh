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

Bài 01-06 giới thiệu FinOps như *measured service* có vòng phản hồi. Bài bắt buộc Chương 12 nhắc máy tính giá và egress. Bài này là **biết đọc hóa đơn cho cặp IUH**: hóa đơn cấu trúc thế nào, đồng hồ nào bất ngờ, và phải bật gì *trước* thí nghiệm free-tier.

## Mục tiêu học tập

- Đọc hóa đơn như **đồng hồ × đơn giá × thời gian**.
- Gắn thẻ để trả lời “Bảng tin tuần này tốn bao nhiêu?”
- Đặt **ngân sách và cảnh báo** *trước* `apply` đầu tiên; destroy thuộc Định nghĩa xong việc.
- So on-demand, cam kết, và dung lượng kiểu spot như *rủi ro*, không như phiếu giảm giá.

## 1. Đo lường là sản phẩm, không phải lịch sự

| Họ đồng hồ | Ví dụ | Vì sao sinh viên bỏ sót |
| --- | --- | --- |
| Giờ tính toán | VM, cân bằng tải, NAT | “Tắt VM rồi” nhưng NAT còn |
| GB-tháng lưu trữ | Đĩa, object, snapshot, log | Snapshot sống sau khi xóa VM |
| I/O và số lần gọi | PUT object, IOPS | Health check nói nhiều |
| Egress | Byte ra khỏi vùng / ra internet | Bạn cùng lớp tải ảnh từ nước khác |
| Phụ quản lý | IPv4 công, registry | IPv4 thành dòng riêng trong những năm 2020 |

$$
C = \sum_i (q_i \cdot p_i), \quad u = C / N_{\text{success}}
$$

Nếu $$N_{\text{success}} = 0$$, bạn không có đơn giá — bạn có sở thích vẫn bị tính tiền.

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

Số trong bài là **minh họa**. Luôn mở trang giá hiện hành.

## 2. Vòng FinOps ở quy mô sinh viên

FinOps Foundation: **Inform → Optimize → Operate**. Ở IUH:

1. **Inform** — thẻ, ngân sách, ảnh cost-by-tag mỗi tuần.
2. **Optimize** — tắt idle, chọn Docker local khi đám mây không thêm chuẩn đầu ra.
3. **Operate** — lịch destroy; một người sở hữu email cảnh báo.

```text
project=iuh-courseboard
pair=<mssv-hoac-ten-nhom>
env=lab|staging|prod
owner=<github-handle>
```

<div class="content-box warning-box">
<p><strong>Ngân sách trước, tài nguyên sau.</strong> Nếu không tạo được budget, đừng <code>apply</code> API tính tiền. Dùng Lab A trên localhost.</p>
</div>

## 3. Dáng giá

| Dáng | Mua gì | Hợp lab hai tuần? |
| --- | --- | --- |
| **On-demand** | Theo giờ/giây | Mặc định. Dễ xóa. |
| **Cam kết / reserved** | Rẻ hơn nếu hứa 1–3 năm | Hầu như không. Bạn trả sau học kỳ. |
| **Spot / preemptible** | Dung lượng thừa, có thể mất | Tùy cho batch; tệ cho ngày chấm. |

“Free tier” là **dáng thứ tư** có hạn mức và danh sách sản phẩm — phiếu trên *một số* đồng hồ, không phải chăn.

## 4. Thực đơn bất ngờ (bản IUH)

1. **NAT** theo giờ *và* theo GB.
2. **Cân bằng tải** idle sau khi xóa máy.
3. **IPv4 công** thành đồng hồ riêng.
4. **Nuốt quan sát** (GB log, số span) nếu bật debug ồn (13-03).
5. **Egress** object storage khi demo trên 4G.
6. **Snapshot / AMI** sống sau `destroy` vì tạo ngoài stack.
7. **GPU / API AI quản lý** — giữ khỏi đồ án mặc định.

Đồng hồ không phục vụ chuẩn đầu ra thì **đừng bật**.

## 5. Kiến trúc cho quy mô *và* chi phí

1. Đơn vị công việc và đích $$u$$?
2. Thành phần nào *phải* đàn hồi?
3. Tải **bằng không** thì tiền thế nào?
4. Egress cắt ranh tính tiền ở đâu?
5. Có cần **hai AZ** cho chuẩn đầu ra không?
6. Cơ sở dữ liệu quản lý có rẻ hơn giờ vá Postgres — và giờ idle — không?
7. Thứ tự destroy, ai chạy sau demo?
8. Cảnh báo ngân sách cặp này *nhìn thấy* chứ?

## 6. Đạo đức và tài khoản dùng chung

- Không đào coin, không huấn luyện mô hình lớn, không làm kho file công trên hạn mức lớp.
- Không chia sẻ root trên nhóm chat.
- Ở đúng project / subscription được phát.
- Ghi chi phí ước lượng trong báo cáo kể cả khi $$0$$ vì ở local.

## Thách thức

- Trang giá đổi; blog 2022 không phải báo giá.
- Nhà cung cấp đo đĩa; bạn chọn để lại.
- ClickOps làm trôi thẻ (Chương 14).
- Dòng thuế / tỷ giá bị bỏ qua.

## Bài tập

1. Thêm NAT quên 200 giờ vào mô hình `invoice_total`. $$N_{\text{success}} = 40$$ thì đơn giá còn trung thực?
2. Checklist destroy cuối lab (snapshot, IP công, log group).
3. Bạn cùng nhóm muốn reserved 1 năm cho đồ án hai tuần. Tranh luận bằng bảng §3.
4. Ước idle 14 ngày: một VM nhỏ + một đĩa + một cân bằng tải không dùng. Ghi ngày xem giá.

## Tiếp theo

- [01-06]({{ site.baseurl }}{% multilang_post_url contents/chapter01/21-01-01-01_06_Modern_Cloud_Applications %})
- [Lab C]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_03_Lab_Free_Tier_Notes %})
- [Đồ án]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_04_Capstone %})

## Tài liệu

1. FinOps Foundation, *FinOps Framework*.
2. Trang giá và free-tier AWS, Azure, GCP (kiểm đúng ngày ước lượng).
3. Flexera *State of the Cloud* (bài 01-06).
4. [infracourse.cloud](https://infracourse.cloud/) — cảm hứng; số liệu và quy tắc IUH là nguyên tác.
