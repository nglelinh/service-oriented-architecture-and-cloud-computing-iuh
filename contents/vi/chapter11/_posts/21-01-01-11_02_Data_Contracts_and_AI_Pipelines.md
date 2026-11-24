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

Hãy hình dung kịch bản quen thuộc ở hầu hết các công ty công nghệ: chiều thứ Sáu, đội Backend đổi tên cột `price` thành `unit_price_vnd` và đổi định dạng ngày từ chuỗi ISO sang timestamp số nguyên để tối ưu ứng dụng di động. Dịch vụ đặt hàng vẫn chạy xanh rì—nhưng sáng thứ Hai, bảng điều khiển doanh thu của giám đốc trống trơn và hệ thống gợi ý AI trả về kết quả vô nghĩa vì toàn bộ pipeline dữ liệu phía sau âm thầm vỡ nát.

Sự cố đó không phải do đội dữ liệu yếu kém, mà do thiếu **Hợp đồng dữ liệu (Data Contracts)**—giao diện có phiên bản, được kiểm chứng tự động giữa bên sinh dữ liệu (producer) và bên tiêu thụ dữ liệu (consumer). Và khi doanh nghiệp nạp dữ liệu vào các mô hình **AI/LLM (RAG, fine-tuning)**, yêu cầu làm sạch dữ liệu còn khắt khe gấp bội: dữ liệu trùng lặp, rác HTML hay rò rỉ thông tin cá nhân (PII) sẽ trực tiếp đầu độc câu trả lời của mô hình.

## Bản đồ kiến thức nền: Bốn trụ cột của kỹ thuật chất lượng dữ liệu thời đại AI

1. **Hợp đồng dữ liệu (Data Contracts) & Schema Registry**:
   Hợp đồng dữ liệu quy định rõ bốn cam kết: **lược đồ (schema)** (tên cột, kiểu dữ liệu, bắt buộc/tùy chọn), **ngữ nghĩa (semantics)** (đơn vị tiền tệ, múi giờ), **SLA độ tươi mới** (dữ liệu trễ tối đa bao nhiêu phút) và **quyền sở hữu**. Các công cụ như Avro/Protobuf kết hợp Schema Registry chặn mọi thay đổi phá vỡ tương thích ngược (*breaking changes*) ngay từ bước CI trước khi đẩy lên sản xuất.

2. **Cổng kiểm soát chất lượng (Data Quality Gates — Pattern Write-Audit-Publish)**:
   Thay vì đổ thẳng dữ liệu thô vào bảng sản xuất rồi cầu nguyện, kỹ thuật **WAP (Write–Audit–Publish)** ghi batch mới vào một nhánh/vùng tạm, chạy bộ kiểm định tự động (kiểm tra `NOT NULL`, miền giá trị, tỷ lệ trùng khóa, phân phối bất thường) rồi mới công bố nguyên tử sang bảng chính nếu đạt ngưỡng; các bản ghi lỗi được chuyển vào **Dead-Letter Queue (DLQ)** để điều tra.

3. **Làm sạch chuyên biệt cho đường ống AI/LLM (Chunking, Deduplication & PII Redaction)**:
   Với SQL truyền thống, thiếu một dấu phẩy chỉ gây lỗi dòng đó. Với hệ thống **RAG (Retrieval-Augmented Generation)**, các tài liệu trùng lặp làm tràn top-$k$ kết quả tìm kiếm vector, HTML/footer rác đốt sạch cửa sổ ngữ cảnh (context window) và tốn chi phí token, còn thông tin nhạy cảm (CCCD, số điện thoại, mật khẩu) nếu không che lọc (*PII redaction*) có thể bị LLM lặp lại cho người dùng khác.

4. **Nguồn gốc dữ liệu (Data Lineage) & Khả năng truy vết**:
   Mỗi tập dữ liệu huấn luyện hay chỉ mục vector phải gắn mã phiên bản nguồn gốc để khi phát hiện một tài liệu chứa thông tin sai hoặc yêu cầu xóa theo quyền riêng tư, hệ thống biết chính xác cần gỡ bỏ đoạn vector nào mà không phải xây lại toàn bộ từ con số không.

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

### Khung giải thích (nên đọc chậm): Trace 10.000 tài liệu đi qua 3 cổng kiểm soát chất lượng và tính chi phí token tiết kiệm được cho RAG

Giả sử bạn xây dựng hệ thống hỏi–đáp nội bộ (RAG) bằng cách thu thập **10.000 trang tài liệu** từ wiki công ty (trung bình `1.200 token/trang`, tương đương **12.000.000 token** thô) để tạo embedding và lưu vào Vector Database.

Hãy theo dõi từng cổng làm sạch biến đổi tập dữ liệu này ra sao:

| Cổng kiểm soát (Quality Gate) | Luật kiểm tra áp dụng | Số trang / token bị loại hoặc chỉnh sửa | Còn lại sau cổng |
| :--- | :--- | :--- | :---: |
| **Đầu vào thô** | Thu thập từ HTML/PDF nội bộ | — | `10.000` trang (`12,00M` token) |
| **Cổng 1: Hợp đồng lược đồ & Xác thực** | Loại trang rỗng, lỗi mã hóa UTF-8, thiếu `doc_id` hoặc `updated_at` | Đẩy `400` trang lỗi (`4%`) sang DLQ | `9.600` trang (`11,52M` token) |
| **Cổng 2: Khử trùng lặp (Exact + MinHash near-dup)** | Loại các bản sao lưu nháp, trang copy từ năm cũ trùng $> 85\%$ nội dung | Loại `2.600` trang trùng lặp (`27%` của 9.600) | `7.000` trang (`8,40M` token) |
| **Cổng 3: Tẩy rác HTML/Navbar & Che PII** | Gỡ thanh điều hướng, chân trang lặp lại (chiếm `25%` số token mỗi trang) và che số điện thoại/email cá nhân | Giảm `2,10M` token rác + che `340` chuỗi PII | `7.000` trang (**`6,30M` token sạch**) |

#### Kiểm tra tác động bằng con số cụ thể

1. **Giảm chi phí & dung lượng Vector DB**:
   Số token cần tính embedding giảm từ $12{,}00\text{M}$ xuống $6{,}30\text{M}$—tiết kiệm **$47{,}5\%$ chi phí API embedding và dung lượng lưu trữ vector**.
2. **Tăng độ chính xác truy hồi (Recall@5)**:
   Trước khi khử trùng lặp, nếu một tài liệu quy định nghỉ phép bị nhân bản 4 lần trên wiki, cả 4/5 kết quả `top_k = 5` trả về cho LLM sẽ chỉ lặp lại đúng 1 tài liệu đó—đẩy mất các thông tin bổ sung quan trọng ra ngoài cửa sổ ngữ cảnh! Sau Cổng 2, mỗi slot trong `top_k = 5` là một mảnh tri thức độc lập, nâng chất lượng câu trả lời lên rõ rệt.

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
