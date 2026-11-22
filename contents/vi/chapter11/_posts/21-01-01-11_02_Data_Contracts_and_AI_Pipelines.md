---
layout: post
title: 11-02 Hợp đồng dữ liệu, cổng chất lượng và pipeline AI
chapter: '11'
order: 3
owner: Nguyen Le Linh
lang: vi
categories:
- chapter11
lesson_type: optional
---

Phần bắt buộc của chương đề cập quy trình thu thập, làm sạch và kiểm chứng dữ liệu. Bài tùy chọn này trình bày cách các nhóm nền tảng giai đoạn 2022–2026 chuẩn hóa những bước đó thành **hợp đồng kiểu SOA**: lược đồ có quản lý phiên bản giữa bên cung cấp (producer) và bên sử dụng (consumer), cổng kiểm soát chất lượng tự động, cùng các yêu cầu xử lý đặc thù cho pipeline **LLM/RAG** vốn phức tạp hơn dữ liệu bảng CSV thông thường.

> Lý thuyết bắt buộc hiện ở bản tiếng Anh. Đây là bài ứng dụng tùy chọn bằng tiếng Việt.

## Mục tiêu học tập

- Định nghĩa hợp đồng dữ liệu (data contract) như một giao diện kỹ thuật thực thi được (gồm lược đồ, SLO và đơn vị sở hữu), không chỉ là tài liệu mô tả trên slide.
- Đặt các công cụ kiểm thử dữ liệu (Great Expectations, dbt tests, schema registry) vào đúng vị trí trong quy trình kiểm chứng.
- Liệt kê các quy tắc làm sạch bổ sung đối với prompt, vector embedding và ngữ liệu huấn luyện AI.

## 1. Hợp đồng dữ liệu: giao diện chuẩn cho hệ thống phân tích

**Hợp đồng dữ liệu (data contract)** — được nhiều tổ chức như PayPal hay GoCardless chia sẻ rộng rãi trong giai đoạn 2022–2024 — là bản đặc tả do bên tạo dữ liệu (producer) cam kết duy trì, bao gồm:

- Lược đồ dữ liệu (kiểu dữ liệu, khả năng rỗng `nullability`, miền giá trị `enum`, đánh dấu trường PII).
- Cam kết SLO về độ tươi (freshness) và độ đầy đủ (completeness).
- Chính sách tương thích phiên bản (tương thích ngược hay thay đổi phá vỡ).
- Đơn vị sở hữu và đầu mối trực vận hành (on-call).

Theo tư duy SOA, một bảng trong kho dữ liệu cũng là một *dịch vụ*: tự ý thay đổi kiểu dữ liệu của cột `user_id` gây ra hậu quả tương đương việc phá vỡ hợp đồng một trường trong Protocol Buffers.

```yaml
name: campus.enrollments.v2
owner: registrar-platform
sla:
  freshness_minutes: 60
  completeness: 0.995
schema:
  - { name: student_id, type: string, pii: true }
  - { name: course_id, type: string, pii: false }
  - { name: enrolled_at, type: timestamp }
compat: backward
```

Các hệ thống tiêu thụ dữ liệu (tác vụ Spark, dashboard báo cáo, feature store) đăng ký đọc theo đúng phiên bản **v2**, thay vì phụ thuộc vào cấu trúc dữ liệu thô bất kỳ rơi xuống vùng đệm (landing zone).

## 2. Cổng kiểm soát chất lượng trong pipeline

```python
import pandas as pd

REQUIRED = ["student_id", "course_id", "enrolled_at"]

def quality_gate(df: pd.DataFrame) -> None:
    missing = [c for c in REQUIRED if c not in df.columns]
    if missing:
        raise ValueError(f"phá hợp đồng: thiếu {missing}")
    null_rate = df["student_id"].isna().mean()
    if null_rate > 0.005:
        raise ValueError(f"trượt SLO đầy đủ: {null_rate:.3%}")
    if df.duplicated(["student_id", "course_id"]).any():
        raise ValueError("ghi danh trùng")
```

Các công cụ phổ biến trong thực tế giai đoạn 2024–2026 gồm **dbt tests**, Great Expectations, Soda, Monte Carlo và schema registry (Confluent, AWS Glue). Bản chất của chúng là tự động hóa các bước kiểm chứng trong pipeline CI/CD để dữ liệu lỗi bị chặn ngay từ đầu, không lọt vào snapshot Iceberg sản xuất.

## 3. Dữ liệu cho AI/LLM có yêu cầu làm sạch riêng

| Giai đoạn | Hậu quả nếu bỏ qua bước làm sạch |
| --- | --- |
| Thu thập nguồn (web, phiếu hỗ trợ, LMS) | Dính nội dung vi phạm bản quyền hoặc chứa PII |
| Khử trùng lặp và che giấu PII | Mô hình có nguy cơ ghi nhớ và phát lại email sinh viên |
| Phân đoạn (chunking) và tạo embedding | Hệ thống truy xuất trả về tài liệu thuộc tenant khác |
| Tập dữ liệu đánh giá | Không đo được hệ thống RAG đang tốt lên hay kém đi |
| Kho lưu trữ prompt và trace | Vô tình dùng nhật ký chứa bí mật rò rỉ để huấn luyện lại |

Một mẫu thiết kế an toàn là **phân tách hai vùng dữ liệu**: Vùng A là hệ thống ghi sổ chính (có hợp đồng dữ liệu và giao dịch ACID); Vùng B chứa embedding dẫn xuất và nhật ký prompt với thời hạn lưu giữ ngắn hơn, phân quyền ACL chặt hơn, cùng quy trình *tạo lại embedding* tường minh khi nâng cấp mô hình embedding (xem Chương 07).

<div class="content-box warning-box">
<p><strong>Thu thập tự động (web scraping) không thay thế chiến lược nguồn dữ liệu hợp lệ.</strong> Cần kiểm tra <code>robots.txt</code>, điều khoản giấy phép và quy định bảo vệ dữ liệu cá nhân tại Việt Nam/EU trước khi chạy mã thu thập như <code>BeautifulSoup</code>. So sánh giữa dùng API chính thức và scraping hiện là vấn đề tuân thủ pháp lý quan trọng.</p>
</div>

## Thách thức

- Hợp đồng dữ liệu chỉ tồn tại trên tài liệu wiki mà không gắn vào điểm kiểm tra tự động trong pipeline.
- Quy tắc làm sạch quá chặt vô tình loại bỏ nhóm bản ghi quan trọng, gây thiên lệch (bias) cho phân tích hoặc mô hình.
- Dùng LLM làm công cụ làm sạch dữ liệu dễ triển khai ở mức thử nghiệm nhưng tốn kém và khó kiểm toán ở quy mô sản xuất.

## Video tham khảo

- **Xem trước:** [What is a Data Contract?](https://www.youtube.com/watch?v=-n3OD-ml_k0) — IBM Technology (tiếng Anh). Định nghĩa hợp đồng dữ liệu — mảnh ghép giao diện còn thiếu trong hệ thống phân tích.
- [What Are Data Contracts? (And How They Actually Work in Production)](https://www.youtube.com/watch?v=6svVYbAcfHc) — The Data and AI Guy (tiếng Anh). Minh họa cách hợp đồng dữ liệu vận hành giữa bên tạo và bên dùng trong pipeline thực tế.
- [What is a Data Contract?](https://www.youtube.com/watch?v=3DU6vuES5eQ) — Monte Carlo (tiếng Anh). Góc nhìn ngắn gọn về hợp đồng dữ liệu từ đơn vị chuyên về độ tin cậy dữ liệu.

## Bài tập

1. Soạn một hợp đồng dữ liệu dài khoảng 12 dòng cho tập dữ liệu `wifi_sessions` quy định các ràng buộc mà nhóm vận hành mạng không được phép phá vỡ.
2. Bổ sung một bước kiểm thử chất lượng nhằm phát hiện sự lẫn lộn múi giờ (ví dụ `enrolled_at` trộn giữa giờ địa phương và UTC).
3. Thiết kế bước ẩn danh dữ liệu cho ngữ liệu RAG của bộ phận hỗ trợ kỹ thuật (helpdesk): trường nào cần xóa hẳn, trường nào cần băm (hash) và trường nào được giữ nguyên?

## Tài liệu tham khảo

1. Các bài viết thực hành trong ngành về data contract (blog kỹ thuật PayPal; bài viết của GoCardless giai đoạn 2022–2024) — tham khảo như *thực hành kỹ thuật*, không phải chuẩn bắt buộc.
2. Tài liệu kiểm thử dbt: [docs.getdbt.com](https://docs.getdbt.com/).
3. Tài liệu Great Expectations: [greatexpectations.io](https://greatexpectations.io/).
4. NIST AI Risk Management Framework (AI RMF 1.0, 2023).
5. Các kỹ thuật làm sạch cốt lõi trong phần lý thuyết bắt buộc (dữ liệu thiếu, trùng lặp, kiểm tra lược đồ) — bài này tập trung vào tự động hóa chúng.
