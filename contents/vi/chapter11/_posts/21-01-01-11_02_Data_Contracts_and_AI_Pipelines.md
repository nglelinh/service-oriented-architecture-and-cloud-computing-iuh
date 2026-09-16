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

Bài bắt buộc là thu thập, làm sạch và kiểm chứng. Ghi chú tùy chọn là cách nhóm nền tảng 2022–2026 biến các việc đó thành **hợp đồng SOA**: lược đồ có phiên bản giữa producer và consumer, cổng chất lượng tự động, và pipeline *riêng* cho dữ liệu **LLM/RAG** không thể coi như thêm một CSV.

> Lý thuyết bắt buộc đang ở bản tiếng Anh. Đây là phần ứng dụng tùy chọn bằng tiếng Việt.

## Mục tiêu học tập

- Định nghĩa hợp đồng dữ liệu như giao diện (lược đồ + SLO + chủ), không phải slide.
- Đặt Great Expectations / kiểm thử dbt / schema registry lên danh sách “kiểm chứng” bắt buộc.
- Liệt kê quy tắc làm sạch thêm cho prompt, embedding và ngữ liệu huấn luyện.

## 1. Hợp đồng dữ liệu: WSDL còn thiếu của phân tích

**Hợp đồng dữ liệu** (được viết nhiều trong ngành bởi PayPal, GoCardless và khác khoảng 2022–2024) là spec do producer sở hữu:

- Lược đồ (kiểu, nullability, enum, cờ PII).
- SLO độ tươi và đầy đủ.
- Chính sách tương thích (cộng thêm vs. phá vỡ).
- Chủ và trực ca.

Đó là SOA: bảng kho là *dịch vụ*. Phá kiểu `user_id` cùng lớp lỗi với phá trường protobuf.

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

Consumer (job Spark, dashboard, feature store) đăng ký **v2**, không phải “thứ gì rơi xuống landing zone.”

## 2. Cổng chất lượng trong pipeline

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

Công cụ trên CV 2024–2026: kiểm thử **dbt**, Great Expectations, Soda, Monte Carlo, schema registry (Confluent, Glue). *Ý tưởng* là phần kiểm chứng bắt buộc, tự động trong CI để tệp xấu không bao giờ thành snapshot Iceberg “đã sạch.”

## 3. Dữ liệu AI/LLM không phải “văn bản phi cấu trúc thường”

| Giai đoạn | Thất bại nếu bỏ làm sạch |
| --- | --- |
| Nguồn (web, ticket, LMS) | Giấy phép / PII trong ngữ liệu |
| Khử trùng & che PII | Mô hình nhớ email sinh viên |
| Cắt + embed | Truy xuất trả nhầm tenant |
| Tập đánh giá | Không biết RAG có tệ hơn không |
| Kho prompt/trace | Quý sau huấn luyện trên bí mật rò |

Mẫu ứng dụng: **hai vùng**. Vùng A là hệ thống ghi sổ (hợp đồng, ACID). Vùng B là embedding dẫn xuất và log prompt với lưu giữ ngắn hơn, ACL chặt hơn, và job *embed lại* tường minh khi đổi mô hình embedding (xem Chương 07).

<div class="content-box warning-box">
<p><strong>Cào web không phải chiến lược nguồn.</strong> Robots.txt, giấy phép và luật dữ liệu cá nhân Việt Nam/EU áp trước khi gọi <code>BeautifulSoup</code>. Phần “API vs cào” của bài bắt buộc là chủ đề tuân thủ năm 2026.</p>
</div>

## Thách thức

- Hợp đồng không có điểm thực thi (trang wiki không phải cổng).
- Làm sạch làm rơi đúng các hàng quan trọng (thiên lệch).
- LLM-như-bộ-làm-sạch: rẻ để demo, đắt để kiểm toán.

## Bài tập

1. Viết hợp đồng 12 dòng cho `wifi_sessions` mà nhóm mạng không được phá.
2. Thêm một kiểm thử chất lượng bắt được lẫn múi giờ (`enrolled_at` local vs UTC).
3. Thiết kế bước che cho ngữ liệu RAG helpdesk (xóa gì, băm gì, giữ gì).

## Tài liệu tham khảo

1. Bài viết ngành về data contract (blog kỹ thuật PayPal; bài GoCardless 2022–2024) — đọc như *thực hành*, không như tiêu chuẩn.
2. Tài liệu kiểm thử dbt: [docs.getdbt.com](https://docs.getdbt.com/).
3. Tài liệu Great Expectations: [greatexpectations.io](https://greatexpectations.io/).
4. NIST AI Risk Management Framework (AI RMF 1.0, 2023).
5. Kỹ thuật bài bắt buộc (thiếu dữ liệu, trùng, kiểm lược đồ) — ghi chú này chỉ tự động hóa chúng.
