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

Mười năm trước, câu hỏi quen thuộc nhất khi thiết kế tầng dữ liệu là một lựa chọn nhị phân: *"Dùng SQL (PostgreSQL/MySQL) để có giao dịch ACID chặt chẽ, hay dùng NoSQL (Cassandra/MongoDB) để mở rộng ngang?"* Nhưng bước vào giai đoạn 2022–2026, ranh giới đó không còn đơn giản như vậy nữa. Các cơ sở dữ liệu **serverless** tự co giãn năng lực theo từng giây tải; **SQL phân tán** (Spanner, CockroachDB) mang giao dịch ACID lên nhiều vùng địa lý; còn làn sóng AI/RAG lại đưa thêm một thành viên hoàn toàn mới vào kiến trúc: **cơ sở dữ liệu vector** để tìm kiếm theo ngữ nghĩa.

Phần lý thuyết bắt buộc trình bày định lý CAP, so sánh BASE với ACID và bốn họ cơ sở dữ liệu NoSQL. Bài tùy chọn này bổ sung ba dòng hệ quản trị dữ liệu nổi bật giai đoạn 2022–2026: cơ sở dữ liệu vận hành theo mô hình **serverless**, **SQL phân tán** kết hợp giao dịch ACID với khả năng mở rộng ngang, và **chỉ mục vector** phục vụ ứng dụng tìm kiếm ngữ nghĩa và RAG/AI.

> Lý thuyết bắt buộc hiện ở bản tiếng Anh. Đây là bài ứng dụng tùy chọn bằng tiếng Việt.

Một ứng dụng khởi nghiệp xây hệ thống hỏi đáp tài liệu nội bộ (RAG) gặp cùng lúc hai cơn đau đầu về cơ sở dữ liệu: ban đêm chẳng có ai truy cập nhưng cụm PostgreSQL truyền thống vẫn đốt tiền điện toán suốt 24/7; còn ban ngày, khi người dùng tìm kiếm theo ngữ nghĩa (*"chính sách nghỉ phép năm"* khớp với *"quy định ngày nghỉ hưởng lương"*), câu lệnh `LIKE '%nghỉ phép%'` của SQL truyền thống hoàn toàn bó tay!

Bài học này khám phá ba làn sóng cơ sở dữ liệu đám mây giai đoạn 2022–2026: **CSDL Serverless** tách rời tính toán khỏi lưu trữ (scale-to-zero), **SQL phân tán (Distributed SQL)** mở rộng ngang mà vẫn giữ giao dịch ACID toàn cục, và **Cơ sở dữ liệu Vector** (`pgvector`, Qdrant, Milvus) dùng chỉ mục đồ thị **HNSW** để tìm hàng xóm gần nhất trong hàng triệu vector nhúng chỉ trong vài mili-giây.

---

## Kiến thức cần nhớ: Ba cuộc cách mạng Cơ sở dữ liệu Đám mây hiện đại

1. **CSDL Serverless (Tách Compute–Storage & Scale-to-Zero):** Các hệ thống như Neon, Aurora Serverless v2 lưu trang dữ liệu trên tầng object/page storage dùng chung, cho phép bật/tắt hoặc co giãn nút tính toán trong vài trăm mili-giây — thậm chí **tạo nhánh CSDL (database branching)** tức thì giống Git cho môi trường kiểm thử!
2. **SQL Phân tán (Distributed SQL — CockroachDB, Spanner, TiDB):** Kết hợp khả năng chia mảnh ngang (sharding) của NoSQL với tính toàn vẹn giao dịch ACID của SQL thông qua giao thức đồng thuận **Raft/Paxos** (cần đa số $$\lfloor N/2
floor + 1$$ nút xác nhận).
3. **Cơ sở dữ liệu Vector & Tìm kiếm gần đúng (ANN / HNSW):** Biểu diễn văn bản/hình ảnh thành vector $$d$$ chiều (ví dụ $$d = 1536$$) và tìm top-$$k$$ tài liệu gần nhất theo **Cosine Similarity** hoặc khoảng cách Euclid. Thay vì quét vét cạn $$O(N\cdot d)$$, chỉ mục **HNSW** (Hierarchical Navigable Small World) nhảy qua đồ thị phân tầng với tốc độ $$O(\log N)$$!

---

## Kiến thức cần nhớ: Bốn trụ cột của tầng dữ liệu đám mây hiện đại

1. **Đàn hồi tầng dữ liệu (Serverless Database & Connection Pooling)**:
   Đưa đặc trưng *rapid elasticity* của NIST xuống tận động cơ lưu trữ: tách tầng tính toán (compute) khỏi tầng lưu trữ (storage) để tăng giảm đơn vị năng lực (như ACU) theo tải thực tế, nhưng phải kiểm soát **cold start** và **bão kết nối** bằng connection pooler.
2. **Đồng thuận phân tán & SQL phân tán (Raft / Paxos + HLC / TrueTime)**:
   Thay vì từ bỏ SQL khi dữ liệu vượt quá 1 máy chủ, SQL phân tán chia bảng thành nhiều mảnh (*range/shard*) và dùng giao thức đồng thuận **Raft/Paxos** để đảm bảo giao dịch ACID đa nút, chấp nhận thêm một vòng trễ mạng (RTT) giữa các vùng sẵn sàng.
3. **Tìm kiếm ngữ nghĩa & Chỉ mục láng giềng gần đúng (ANN / HNSW / IVF)**:
   Khác với chỉ mục B-Tree tìm khớp chính xác (`id = 42`), chỉ mục vector đo độ tương đồng hình học (khoảng cách Euclid hoặc Cosine) trong không gian hàng nghìn chiều để tìm đoạn văn bản có *ý nghĩa* gần nhất với câu hỏi của người dùng.
4. **Phân quyền đa thuê bao trong hệ RAG (Pre-filtering vs. Post-filtering)**:
   Kho vector là tầng *truy xuất phụ trợ*, không thay thế CSDL ghi sổ chính (system of record). Ràng buộc quyền truy cập (`tenant_id`, ACL) phải gắn chặt vào bước duyệt chỉ mục để tránh rò rỉ dữ liệu giữa các khách hàng.

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

### Giải thích: Tính tay dung lượng RAM cho 1 triệu vector và độ trễ đồng thuận Raft

#### Bước 1: Ước lượng bộ nhớ cho bảng `docs` chứa **1.000.000 đoạn tài liệu** với `vector(1536)`

Mỗi chiều của vector dùng kiểu số thực `float32` chiếm **4 byte**. Do đó:
- Kích thước 1 vector: $$1.536 \times 4\text{ byte} = \mathbf{6.144\text{ byte}} = \mathbf{6\text{ KB}}.$$
- Tổng dung lượng thô của 1.000.000 vector:
  $$10^6 \times 6\text{ KB} = \mathbf{6\text{ GB}}.$$
- Khi xây chỉ mục đồ thị **HNSW** (giữ thêm danh sách cạnh láng giềng trong RAM, hệ số phình thường từ $$1{,}5\times$$ đến $$2\times$$):
  $$\text{RAM cần cho chỉ mục HNSW} \approx 6\text{ GB} \times 1{,}8 \approx \mathbf{10{,}8\text{ GB RAM}}.$$

Con số này giải thích ngay vì sao chỉ mục vector tốn RAM nhanh đến thế, và vì sao kỹ thuật nén lượng tử hóa (`halfvec` float16 giảm một nửa xuống ~5,4 GB, hay binary quantization giảm $$32\times$$) là chủ đề nóng trong thực tế.

#### Bước 2: Cái giá độ trễ vật lý của giao dịch ACID trên SQL phân tán đa vùng

Giả sử cụm SQL phân tán đặt **3 bản sao Raft** ở 3 vùng: **Singapore** (Leader), **Tokyo** (RTT = 70 ms) và **Sydney** (RTT = 90 ms). Để cam kết một lệnh `INSERT` an toàn theo đa số (quorum $$2/3$$), Leader tại Singapore phải nhận xác nhận từ ít nhất **1 bản sao nữa** (nhanh nhất là Tokyo):
- Độ trễ ghi tối thiểu = $$1 \times \text{RTT}_{\text{Singapore–Tokyo}} + \text{ghi đĩa cục bộ} \approx 70 + 2 = \mathbf{72\text{ ms}}.$$
- Trong khi đó, ghi trên 1 cụm PostgreSQL đơn vùng (trong cùng 1 Region, RTT giữa các AZ chỉ ~1,5 ms) chỉ mất khoảng **3–4 ms**!

Chính vì vậy, đừng chọn SQL phân tán đa vùng theo phong trào: hãy chỉ trả cái giá **72 ms/giao dịch** khi bài toán thực sự cần chịu mất nguyên một vùng địa lý mà không mất dữ liệu ghi.

## Thách thức

- Chi phí dung lượng tối thiểu (`min capacity`) của cơ sở dữ liệu serverless dễ khiến đội ngũ nhầm lẫn rằng `scale-to-zero` hoàn toàn miễn phí.
- Thay đổi mô hình sinh embedding đồng nghĩa với việc phải tính toán lại toàn bộ vector và xây dựng lại chỉ mục — đây là một đợt di chuyển dữ liệu thực thụ chứ không chỉ là thao tác reindex đơn giản.
- Nhân bản đồng bộ trên đa đám mây cho cả cơ sở dữ liệu SQL và kho vector tạo ra hai bài toán nhất quán song song.

---

### Ví dụ thực tế làm từng bước: Tính chi phí Serverless Scale-to-Zero và chấm điểm Cosine Similarity bằng tay

**Phần A — Bài toán chi phí CSDL Serverless vs. Máy chủ cố định (24/7):**
Một môi trường phát triển/staging chỉ thực sự có truy vấn **3 giờ mỗi ngày** (90 giờ/tháng) với tải trung bình 2 vCPU:
- **RDS cố định 24/7 (720 giờ/tháng):** Trả tiền cho đủ $$720 	imes 2 = 1{,}440	ext{ vCPU-giờ/tháng}$$.
- **CSDL Serverless (scale-to-zero khi nhàn rỗi):** Chỉ trả tiền cho $$90 	imes 2 = \mathbf{180	ext{ vCPU-giờ/tháng}}$$ (cộng phí lưu trữ trang dữ liệu rất nhỏ) — tiết kiệm **87,5%** chi phí tính toán!

**Phần B — Tính tay độ tương đồng Cosine trong tìm kiếm Vector (RAG):**
Giả sử câu hỏi của người dùng có vector chuẩn hóa $$\mathbf{q} = [0{,}6,\; 0{,}8,\; 0{,}0]$$ ($$\|\mathbf{q}\| = \sqrt{0{,}36 + 0{,}64} = 1$$), so với hai đoạn tài liệu đã chuẩn hóa:
- Tài liệu 1 (cùng chủ đề): $$\mathbf{d}_1 = [0{,}8,\; 0{,}6,\; 0{,}0] \implies \cos(\mathbf{q}, \mathbf{d}_1) = 0{,}6(0{,}8) + 0{,}8(0{,}6) + 0 = \mathbf{0{,}96}$$ (rất gần!).
- Tài liệu 2 (khác chủ đề): $$\mathbf{d}_2 = [0{,}0,\; 0{,}0,\; 1{,}0] \implies \cos(\mathbf{q}, \mathbf{d}_2) = \mathbf{0{,}00}$$ (vuông góc, không liên quan!).

### Giải thích: Bốn nguyên tắc chọn đúng Cơ sở dữ liệu cho hệ thống Cloud/AI

1. **Đừng dựng ngay một CSDL Vector chuyên biệt nếu dữ liệu còn dưới vài triệu vector:** Nếu bạn đã dùng PostgreSQL, cài thêm mở rộng **`pgvector` (với chỉ mục HNSW)** cho phép lọc siêu dữ liệu SQL (`WHERE tenant_id = 42`) ngay chung một giao dịch ACID với tìm kiếm vector!
2. **Hiểu cái giá của Scale-to-Zero (Cold Start):** Nút tính toán ngủ đông cần 300 ms–1 giây để đánh thức ở truy vấn đầu tiên — rất tuyệt cho staging và ứng dụng tải theo đợt, nhưng với API thanh toán yêu cầu p99 < 20 ms, hãy đặt mức tối thiểu (`min_capacity > 0`).
3. **Đánh đổi Recall vs. Độ trễ trong chỉ mục HNSW:** Tham số `ef_search` càng lớn, độ chính xác tìm đúng top-$$k$$ (recall) càng gần 100% nhưng độ trễ truy vấn càng tăng — luôn đo đường cong Recall@10 vs. Latency trên dữ liệu thật của bạn.
4. **Tính toán độ trễ đồng thuận Raft trong Distributed SQL:** Ghi dữ liệu đồng bộ xuyên lục địa (Mỹ – Âu – Á) bắt buộc tốn ít nhất 1 vòng RTT mạng (~150 ms); hãy gom cụm bản sao theo vùng địa lý của người dùng.

**Kiểm tra nhanh.** Một cụm Distributed SQL dùng giao thức đồng thuận Raft gồm $$N = 5$$ nút. Khi có **2 nút** bị mất điện cùng lúc, cụm có còn đủ đa số (quorum) để tiếp tục ghi giao dịch ACID không — còn nếu mất **3 nút** thì sao?

---

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
