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

Bài bắt buộc phủ CAP, BASE vs ACID, và bốn họ NoSQL. Ghi chú tùy chọn thêm các kho xuất hiện cạnh các họ đó trong 2022–2026: CSDL vận hành **serverless**, **SQL phân tán** muốn vừa ACID vừa scale, và **chỉ mục vector** cho ứng dụng RAG/AI.

> Lý thuyết bắt buộc đang ở bản tiếng Anh. Đây là phần ứng dụng tùy chọn bằng tiếng Việt.

## Mục tiêu học tập

- Đặt sản phẩm kiểu Aurora/Cosmos/Firestore serverless lên trục IaaS–PaaS.
- Đối chiếu “NoSQL để scale” với SQL phân tán (hạng Spanner).
- Giải thích chỉ mục vector như *đường truy cập mới*, không thay hệ thống ghi sổ.

## 1. Serverless là đàn hồi áp vào cơ sở dữ liệu

Đàn hồi nhanh NIST từng nghĩa “thêm VM.” CSDL serverless scale **dung lượng theo kết nối và lưu trữ**, thường có tạm dừng scale-to-zero:

- Amazon Aurora Serverless v2 (scale ACU), Cloud Spanner / AlloyDB, Azure Cosmos DB serverless, Neon/PlanetScale-style nhánh Postgres.
- Đánh đổi CAP không biến mất: bạn vẫn chọn nhất quán vs. độ trễ khi một vùng chết.

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

Cold start chuyển từ hàm (Chương 08) sang **câu truy vấn đầu sau khi tạm dừng**. Timeout và pooling nay thuộc hợp đồng SOA truy cập dữ liệu.

## 2. SQL phân tán: ACID không cần một primary (khi làm được)

CockroachDB, Yugabyte, Google Spanner và hệ tương tự quảng bá SQL **serializable** (hoặc mạnh) trên shard. Chúng không bãi bỏ CAP: dùng đồng thuận (Paxos/Raft) và đồng hồ (TrueTime hoặc HLC) để đường *thường gặp* trông như một Postgres. Thắng lợi ứng dụng: ít shard tự chế hơn playbook MongoDB 2015.

Dùng khi cần **ghi đa vùng** và join SQL. Đừng dùng như cache rẻ.

## 3. CSDL vector: đường truy cập AI

RAG và tìm embedding thêm hình thứ năm cạnh KV/tài liệu/cột/đồ thị: **láng giềng gần đúng (ANN)** trên vector float (`pgvector`, Pinecone, Weaviate, Milvus, MongoDB Atlas Vector Search).

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

**Quy tắc ứng dụng:** kho vector là *dịch vụ truy xuất*. Hệ thống ghi sổ (đơn hàng, điểm, thanh toán) ở kho ACID hoặc NoSQL chọn kỹ. Trộn chúng trong một chỉ mục không xác thực là cách RAG rò tenant.

<div class="content-box warning-box">
<p><strong>Lọc rồi ANN, hay ANN rồi lọc?</strong> Bộ lọc trước (tenant, ACL) phải nằm trên đường chỉ mục, nếu không bạn lấy láng giềng rồi bỏ — hoặc tệ hơn, trả về. Đây là ủy quyền, không chỉ recall@k.</p>
</div>

## 4. Ánh xạ lại phân loại bắt buộc

| Khối lượng | Mặc định những năm 2020 | Ghi chú CAP |
| --- | --- | --- |
| Phiên / giỏ | KV hoặc Redis | Cache nghiêng AP |
| Danh mục | Document | Lược đồ linh hoạt |
| Sổ cái / ghi danh | SQL phân tán hoặc RDBMS | Nghiêng CP |
| Tìm ngữ nghĩa | Vector + lọc metadata | Xấp xỉ |

## Thách thức

- Hóa đơn *min capacity* serverless khiến nhóm tưởng “scale to zero là miễn phí.”
- Xây lại chỉ mục vector sau khi đổi mô hình embedding (là di trú, không trivia reindex).
- Nhân bản đa đám mây cả SQL lẫn vector (hai câu chuyện nhất quán).

## Bài tập

1. Chọn một hệ thống campus (thư viện, LMS, gửi xe). Họ nào + sản phẩm những năm 2020 nào, và đau CAP nào bạn chấp nhận?
2. Viết kế hoạch ba bước từ “mọi embedding trong cột JSON” sang `pgvector` có lọc tenant.
3. Giải thích vì sao CSDL serverless cộng hàm serverless vẫn cần pooler kết nối.

## Tài liệu tham khảo

1. Hướng dẫn Amazon Aurora Serverless v2 (scale ACU).
2. Dự án `pgvector`: [github.com/pgvector/pgvector](https://github.com/pgvector/pgvector).
3. Google, *Spanner* (OSDI 2012).
4. Tài liệu MongoDB Atlas Vector Search; Pinecone / Weaviate (so ANN + lọc metadata).
5. NIST SP 800-145 — đàn hồi và dịch vụ đo lường vẫn áp cho *dung lượng CSDL*, không chỉ VM.
