---
layout: post
title: 07-02 CSDL serverless, SQL phân tán và cơ sở vector (2022–2026)
chapter: '07'
order: 3
owner: Nguyen Le Linh
lang: vi
categories:
- chapter07
lesson_type: optional
---

Phần lý thuyết bắt buộc trình bày định lý CAP, so sánh BASE với ACID và bốn họ cơ sở dữ liệu NoSQL. Bài tùy chọn này bổ sung ba dòng hệ quản trị dữ liệu nổi bật giai đoạn 2022–2026: cơ sở dữ liệu vận hành theo mô hình **serverless**, **SQL phân tán** kết hợp giao dịch ACID với khả năng mở rộng ngang, và **chỉ mục vector** phục vụ ứng dụng tìm kiếm ngữ nghĩa và RAG/AI.

> Lý thuyết bắt buộc hiện ở bản tiếng Anh. Đây là bài ứng dụng tùy chọn bằng tiếng Việt.

## Mục tiêu học tập

- Xác định vị trí của các dịch vụ cơ sở dữ liệu serverless (như Aurora, Cosmos DB, Firestore) trên trục IaaS–PaaS.
- Đối chiếu hướng tiếp cận "dùng NoSQL để mở rộng" với mô hình SQL phân tán (nhóm hệ thống kiểu Spanner).
- Hiểu đúng vai trò của chỉ mục vector như một *tầng truy xuất bổ trợ*, không thay thế hệ thống cơ sở dữ liệu ghi sổ chính (system of record).

## 1. Serverless: áp dụng tính đàn hồi vào tầng cơ sở dữ liệu

Trước đây, đặc trưng *đàn hồi nhanh* của NIST thường gắn với việc tự động thêm hoặc bớt máy ảo. Với cơ sở dữ liệu serverless, năng lực tính toán và lưu trữ co giãn trực tiếp theo lưu lượng kết nối, thậm chí hỗ trợ tạm dừng khi không có tải (`scale-to-zero`):

- Các đại diện phổ biến gồm Amazon Aurora Serverless v2 (co giãn theo đơn vị ACU), Cloud Spanner / AlloyDB, Azure Cosmos DB serverless, và các nền tảng Postgres hỗ trợ phân nhánh kiểu Neon hay PlanetScale.
- Sự đánh đổi của định lý CAP vẫn giữ nguyên: khi xảy ra phân vùng mạng hoặc mất một vùng sẵn sàng, hệ thống vẫn phải lựa chọn giữa tính nhất quán và độ trễ/tính sẵn sàng.

```python
import os
import psycopg

dsn = os.environ["DATABASE_URL"]

def fetch_order(order_id: str):
    with psycopg.connect(dsn, connect_timeout=5) as conn:
        with conn.cursor() as cur:
            cur.execute("select status from orders where id = %s", (order_id,))
            return cur.fetchone()
```

Hiện tượng khởi động nguội (cold start) lúc này dịch chuyển từ hàm FaaS (Chương 08) sang **truy vấn đầu tiên sau trạng thái tạm dừng**. Do đó, cấu hình timeout và bộ gom kết nối (connection pooling) trở thành một phần quan trọng trong hợp đồng truy cập dữ liệu của kiến trúc SOA.

## 2. SQL phân tán: giao dịch ACID vượt ra ngoài một nút chính đơn lẻ

Các hệ thống như CockroachDB, YugabyteDB và Google Spanner cung cấp giao dịch SQL với mức cô lập cao (thường là serializable) trên nhiều phân mảnh (shard). Chúng không vượt qua định lý CAP mà kết hợp thuật toán đồng thuận (Paxos hoặc Raft) cùng cơ chế đồng bộ thời gian (TrueTime hoặc Hybrid Logical Clock — HLC) để trong điều kiện bình thường, ứng dụng có thể tương tác gần như với một máy chủ PostgreSQL duy nhất. Lợi ích thực tế là giảm đáng kể công sức tự quản lý phân mảnh thủ công.

Nên chọn SQL phân tán khi ứng dụng thực sự cần **ghi dữ liệu đa vùng** và các phép nối SQL phức tạp; không nên dùng nó như một lớp bộ nhớ đệm giá rẻ.

## 3. Cơ sở dữ liệu vector: tầng truy xuất cho ứng dụng AI

Các hệ thống RAG và tìm kiếm ngữ nghĩa dựa trên embedding bổ sung một dạng chỉ mục thứ năm bên cạnh bốn họ NoSQL truyền thống (key–value, tài liệu, cột, đồ thị): **tìm kiếm láng giềng gần đúng (Approximate Nearest Neighbor — ANN)** trên vector số thực (`pgvector`, Pinecone, Weaviate, Milvus, MongoDB Atlas Vector Search).

```sql
CREATE TABLE docs (
  id uuid PRIMARY KEY,
  body text,
  embedding vector(1536)
);
SELECT id, body
FROM docs
WHERE tenant_id = 'iuh'
ORDER BY embedding <-> $1
LIMIT 8;
```

**Nguyên tắc kiến trúc:** Kho vector đóng vai trò là *dịch vụ truy xuất*. Dữ liệu ghi sổ cốt lõi (đơn hàng, điểm số, giao dịch thanh toán) vẫn phải nằm trong hệ quản trị ACID hoặc NoSQL được kiểm soát chặt chẽ. Việc gộp chung dữ liệu nhiều khách hàng vào một chỉ mục vector mà thiếu lớp kiểm tra quyền truy cập là nguyên nhân phổ biến gây rò rỉ dữ liệu giữa các tenant trong hệ thống RAG.

<div class="content-box warning-box">
<p><strong>Lọc trước khi tìm ANN hay tìm ANN rồi mới lọc?</strong> Điều kiện lọc theo quyền truy cập (tenant, ACL) cần được áp dụng gắn liền với quá trình duyệt chỉ mục; nếu lọc sau, kết quả trả về có thể bị thiếu hụt hoặc tệ hơn là lọt dữ liệu không được phép. Đây là bài toán phân quyền bảo mật, không chỉ là chỉ số chất lượng truy xuất <code>recall@k</code>.</p>
</div>

## 4. Ánh xạ vào phân loại lý thuyết

| Nhóm khối lượng công việc | Lựa chọn phổ biến thập niên 2020 | Ghi chú theo CAP |
| --- | --- | --- |
| Phiên đăng nhập / giỏ hàng | Key–Value hoặc Redis | Ưu tiên tính sẵn sàng (AP) |
| Danh mục sản phẩm | Document database | Lược đồ dữ liệu linh hoạt |
| Sổ cái / đăng ký học phần | SQL phân tán hoặc RDBMS | Ưu tiên tính nhất quán (CP) |
| Tìm kiếm ngữ nghĩa | Chỉ mục vector + lọc metadata | Tìm kiếm gần đúng |

## Thách thức

- Chi phí dung lượng tối thiểu (`min capacity`) của cơ sở dữ liệu serverless dễ khiến đội ngũ nhầm lẫn rằng `scale-to-zero` hoàn toàn miễn phí.
- Thay đổi mô hình sinh embedding đồng nghĩa với việc phải tính toán lại toàn bộ vector và xây dựng lại chỉ mục — đây là một đợt di chuyển dữ liệu thực thụ chứ không chỉ là thao tác reindex đơn giản.
- Nhân bản đồng bộ trên đa đám mây cho cả cơ sở dữ liệu SQL và kho vector tạo ra hai bài toán nhất quán song song.

## Video tham khảo

- **Xem trước:** [What is a Vector Database? Powering Semantic Search & AI Applications](https://www.youtube.com/watch?v=gl1r1XV0SLw) — IBM Technology (tiếng Anh). Giải thích cơ sở dữ liệu vector — tầng truy xuất AI bổ sung cạnh bốn họ NoSQL.
- [What is a serverless database? (in under 3 minutes)](https://www.youtube.com/watch?v=GxjKsKSOWJk) — CockroachDB (tiếng Anh). Tóm tắt cơ chế co giãn dung lượng của cơ sở dữ liệu serverless.
- [What is Serverless?](https://www.youtube.com/watch?v=vxJobGtqKVM) — IBM Technology (tiếng Anh). Bổ sung góc nhìn tổng thể về mô hình serverless khi áp dụng cho cả tầng dữ liệu.

## Bài tập

1. Chọn một hệ thống quy mô trường đại học (thư viện, hệ thống quản lý học tập LMS hoặc bãi giữ xe). Đề xuất họ cơ sở dữ liệu và sản phẩm cụ thể phù hợp, đồng thời chỉ rõ đánh đổi CAP mà bạn chấp nhận.
2. Phác thảo kế hoạch ba bước để chuyển đổi từ "lưu toàn bộ embedding trong cột JSON" sang dùng `pgvector` có áp dụng lọc theo `tenant_id`.
3. Giải thích vì sao khi kết hợp hàm serverless với cơ sở dữ liệu serverless, hệ thống vẫn cần một bộ gom kết nối (connection pooler).

## Tài liệu tham khảo

1. Hướng dẫn Amazon Aurora Serverless v2 (co giãn theo ACU).
2. Dự án `pgvector`: [github.com/pgvector/pgvector](https://github.com/pgvector/pgvector).
3. Google, *Spanner* (OSDI 2012).
4. Tài liệu MongoDB Atlas Vector Search; Pinecone / Weaviate (so sánh ANN kết hợp lọc metadata).
5. NIST SP 800-145 — đặc trưng đàn hồi và đo lường áp dụng cho cả tầng *dung lượng cơ sở dữ liệu*.
