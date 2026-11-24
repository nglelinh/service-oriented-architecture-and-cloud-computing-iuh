---
layout: post
title: 01-05 Trung tâm dữ liệu và mở rộng quy mô
chapter: '01'
order: 6
owner: Nguyen Le Linh
lang: vi
categories:
- chapter01
lesson_type: optional
---

Vào những năm 1990, nếu một cơ sở dữ liệu chạy chậm, giải pháp quen thuộc của các kỹ sư là gọi hãng phần cứng mua một "siêu máy chủ" (mainframe) đắt tiền hơn với CPU nhanh hơn và nhiều RAM hơn (**Scale-Up**). Nhưng khi Google phải lập chỉ mục hàng tỷ trang web và Facebook phục vụ hàng tỷ người dùng đồng thời, họ đụng phải bức tường vật lý: không một chiếc máy chủ đơn lẻ nào trên Trái Đất đủ lớn — và giá của một siêu máy chủ tăng theo hàm mũ so với hiệu năng!

Đột phá của điện toán đám mây hiện đại là coi **cả một trung tâm dữ liệu gồm hàng chục nghìn máy chủ phổ thông (commodity servers) là MỘT chiếc máy tính khổng lồ duy nhất** (*Warehouse-Scale Computer*). Bài học này giải thích toán học đứng sau mở rộng dọc (Scale-Up) so với mở rộng ngang (Scale-Out), định luật Amdahl giới hạn tốc độ tăng tốc, và hiệu suất năng lượng PUE của các trung tâm dữ liệu hiện đại.

---

## Bản đồ kiến thức nền: Kiến trúc Trung tâm Dữ liệu & Luật Mở rộng

### 1. Mở rộng dọc (Scale-Up) vs. Mở rộng ngang (Scale-Out)
- **Scale-Up:** Nâng cấp CPU/RAM trên 1 máy. Lập trình cực dễ (không cần phân tán trạng thái), nhưng có **trần vật lý cứng**, chi phí phi tuyến và luôn là một điểm lỗi đơn (**Single Point of Failure**).
- **Scale-Out:** Thêm nhiều máy chủ ngang hàng sau bộ cân bằng tải. Lý thuyết mở rộng không giới hạn và chịu lỗi nút tốt — nhưng đòi hỏi ứng dụng **phi trạng thái (stateless)** và phải giải bài toán đồng bộ dữ liệu phân tán.
- **Câu hỏi lớn:** Phần nào của ứng dụng dễ Scale-Out (tầng web/API) và phần nào khó nhất (tầng cơ sở dữ liệu ghi)?
- **Ứng dụng thực tế:** Cân bằng tải L7, phân mảnh cơ sở dữ liệu (sharding), cụm Kubernetes.

### 2. Định luật Amdahl & Điểm nghẽn tuần tự (Serial Bottlenecks)
Khi bạn tăng số máy chủ lên $$N$$ lần, tốc độ hệ thống **không** tự động tăng $$N$$ lần nếu vẫn còn tỷ lệ công việc tuần tự $$s$$ (ví dụ: khóa ghi chung vào 1 bảng CSDL duy nhất):
$$S(N) = rac{1}{s + rac{1 - s}{N}}$$
- **Câu hỏi lớn:** Nếu 10% thời gian xử lý bị kẹt ở một khóa tuần tự duy nhất, mua thêm 1.000 máy chủ giúp tăng tốc tối đa bao nhiêu lần?
- **Ứng dụng thực tế:** Tìm điểm nghẽn (bottleneck profiling), tách đọc/ghi (CQRS), giảm tranh chấp khóa.

### 3. Hiệu suất năng lượng (PUE) & Thiết kế Warehouse-Scale
Trung tâm dữ liệu hiện đại tiêu thụ hàng chục megawatt điện, trong đó phần lớn năng lượng bị đốt cho làm mát và chuyển đổi điện áp. Chỉ số **PUE (Power Usage Effectiveness)** đo tổng điện năng toàn tòa nhà chia cho điện năng thực sự cấp cho máy chủ IT (lý tưởng tiến gần $$1.0$$).
- **Câu hỏi lớn:** Làm sao các hyperscaler (AWS, Google, Azure) kéo PUE từ 2.0 xuống dưới 1.15?
- **Ứng dụng thực tế:** Làm mát bằng không khí tự nhiên/chất lỏng, thiết kế mô-đun và tính toán xanh (Green Cloud).

---

Đằng sau khái niệm trừu tượng "đám mây" là một hệ thống hạ tầng vật lý quy mô lớn: các trung tâm dữ liệu (data center). Hiểu cách năng lực tính toán mở rộng từ một máy tính cá nhân thành một hệ thống máy tính quy mô nhà kho (warehouse-scale computer) là kiến thức nền tảng của kỹ thuật đám mây.

## Nhu cầu về quy mô

Các dịch vụ web toàn cầu hiện nay vận hành với khối lượng dữ liệu và lưu lượng truy cập vượt xa khả năng đáp ứng của một máy chủ đơn lẻ. Nhiều hệ thống xử lý từ hàng **petabyte (PB)** đến **exabyte (EB)** dữ liệu (để tham chiếu, 1 zettabyte tương đương 1 nghìn tỷ gigabyte). Các nền tảng như Facebook hay YouTube phục vụ hàng tỷ lượt truy cập mỗi ngày, đòi hỏi các cụm máy chủ lớn phối hợp xử lý song song để duy trì độ trễ thấp.

## Hai hướng tiếp cận mở rộng quy mô

Khi một máy chủ chạm ngưỡng giới hạn hiệu năng, kiến trúc sư hệ thống có hai chiến lược chính để nâng công suất: mở rộng dọc và mở rộng ngang.

### 1. Mở rộng dọc — Scale Up (Vertical Scaling)

Mở rộng dọc ("scale up") là nâng cấp phần cứng (CPU mạnh hơn, nhiều RAM hơn hoặc ổ lưu trữ tốc độ cao hơn) trên cùng một máy chủ hiện có. Hướng tiếp cận này thể hiện rõ trong tiến trình phát triển từ máy tính cá nhân lên máy trạm, máy chủ chuyên dụng và cuối cùng là **máy tính lớn (mainframe)**.

Về mặt khái niệm, cách này đơn giản vì không cần thay đổi nhiều trong kiến trúc phần mềm, nhưng gặp ba giới hạn lớn. Thứ nhất là **giới hạn vật lý của phần cứng**: tốc độ CPU và dung lượng RAM tối đa của bo mạch chủ đều có trần cứng. Thứ hai, **chi phí tăng phi tuyến**: phần cứng cao cấp nhất thường đắt hơn nhiều lần so với phần cứng phổ thông nhưng mức tăng hiệu năng không tương xứng. Thứ ba, và đặc biệt quan trọng đối với độ tin cậy trên đám mây, một máy chủ đơn lẻ luôn là **điểm lỗi đơn (single point of failure)** — nếu máy đó gặp sự cố, toàn bộ hệ thống ngừng hoạt động.

### 2. Mở rộng ngang — Scale Out (Horizontal Scaling)

Mở rộng ngang ("scale out") là bổ sung thêm nhiều máy chủ vào hệ thống thay vì cố gắng nâng cấp một máy duy nhất. Đây là nguyên lý kiến trúc nền tảng của Internet hiện đại: thay cho một mainframe đơn lẻ, hệ thống được tổ chức thành một **cụm (cluster)** gồm nhiều máy chủ tiêu chuẩn, mở rộng thành trung tâm dữ liệu và mạng lưới trung tâm dữ liệu toàn cầu.

Hướng tiếp cận này mang lại nhiều ưu điểm quan trọng. Hệ thống có thể sử dụng **phần cứng phổ thông (commodity hardware)** với giá thành thấp hơn nhiều so với mainframe; chi phí mở rộng tăng gần như tuyến tính theo nhu cầu công suất. Quan trọng hơn, mở rộng ngang mang lại **khả năng chịu lỗi cao**: trong một cụm 1.000 máy chủ, nếu một máy hỏng, 999 máy còn lại vẫn tiếp tục gánh tải mà không gây gián đoạn dịch vụ. Đổi lại, **độ phức tạp phần mềm** tăng lên đáng kể do phải quản lý trạng thái phân tán và tính nhất quán dữ liệu trên hàng nghìn nút.

## Trung tâm dữ liệu dưới góc nhìn một hệ thống máy tính

Trung tâm dữ liệu hiện đại không chỉ là phòng đặt máy chủ mà được thiết kế như một "máy tính quy mô nhà kho" thống nhất, tối ưu đồng thời cho hiệu suất, độ tin cậy và quy mô.

### Kiến trúc vật lý

Đơn vị cơ bản của trung tâm dữ liệu là **máy chủ (server)**. Hàng chục máy chủ được lắp vào một khung tiêu chuẩn gọi là **tủ rack** (ví dụ khoảng 40 máy chủ mỗi rack). Mỗi rack có một bộ chuyển mạch **Top of Rack (ToR) switch** kết nối các máy chủ bên trong vào mạng chung. Hàng trăm đến hàng nghìn tủ rack được ghép nối thành các **cụm (cluster)** vận hành như một thực thể tính toán lớn.

### Các đặc điểm hạ tầng then chốt

Để vận hành ổn định ở quy mô lớn, trung tâm dữ liệu đòi hỏi **hạ tầng mạng băng thông cao, độ trễ thấp** (network fabric) liên kết toàn bộ các nút. **Cơ chế dự phòng (redundancy)** được thiết kế ở mọi tầng — từ bộ lưu điện (UPS), máy phát điện diesel, hệ thống làm mát dự phòng đến nhiều tuyến mạng độc lập — nhằm đảm bảo hoạt động liên tục. Bên cạnh đó là **an ninh vật lý nhiều lớp** như kiểm soát ra vào nghiêm ngặt, xác thực sinh trắc học và cửa ngăn hai lớp (man trap).

### Điện năng và tác động môi trường

Trung tâm dữ liệu tiêu thụ lượng điện năng rất lớn: một tủ rack có thể tiêu thụ hơn 4 kW, còn một trung tâm dữ liệu siêu quy mô (hyperscale) sử dụng lượng điện tương đương một đô thị nhỏ. Hầu hết điện năng này chuyển hóa thành nhiệt và phải được tản ra ngoài để bảo vệ thiết bị; do đó **hệ thống làm mát** thường chiếm 30–50% tổng điện năng tiêu thụ của cơ sở. Vì lý do này, nhiều trung tâm dữ liệu được đặt gần nguồn năng lượng tái tạo giá rẻ — chẳng hạn các nhà máy thủy điện dọc lưu vực sông Columbia — nhằm giảm chi phí vận hành và hạn chế tác động môi trường.

## Trung tâm dữ liệu mô-đun và phân tán

### Trung tâm dữ liệu mô-đun (Modular Data Center)

Để rút ngắn thời gian xây dựng và mở rộng công suất, nhiều đơn vị áp dụng mô hình **trung tâm dữ liệu mô-đun**, trong đó máy chủ, thiết bị mạng và hệ thống làm mát được lắp ráp sẵn trong các container tiêu chuẩn. Khi cần mở rộng, chỉ cần vận chuyển container tới địa điểm, đấu nối nguồn điện, hệ thống làm mát và đường truyền mạng. Cách tiếp cận dạng mô-đun lắp đặt nhanh này giúp tối ưu luồng khí làm mát và triển khai linh hoạt.

### Trung tâm dữ liệu phân tán

Với các dịch vụ phục vụ người dùng toàn cầu, một trung tâm dữ liệu đơn lẻ không thể đáp ứng yêu cầu về độ trễ và độ tin cậy do các giới hạn vật lý. **Độ trễ mạng (latency)** bị giới hạn bởi tốc độ truyền tín hiệu ánh sáng: người dùng tại châu Á truy cập máy chủ đặt tại Mỹ sẽ cảm nhận độ trễ rõ rệt. Hơn nữa, tập trung toàn bộ hệ thống tại một địa điểm tiềm ẩn rủi ro lớn khi xảy ra thiên tai. Để khắc phục, các nhà cung cấp xây dựng **mạng lưới trung tâm dữ liệu phân tán toàn cầu**, nhân bản dữ liệu giữa nhiều khu vực và định tuyến yêu cầu tới điểm gần người dùng nhất (Edge), vừa cải thiện hiệu năng vừa tăng khả năng chống chịu sự cố.

---

## Ví dụ cầm tay chỉ việc & Khung giải thích

### Ví dụ cầm tay chỉ việc 1: Tính trần tăng tốc theo Định luật Amdahl khi mở rộng ngang

Giả sử một API xử lý yêu cầu có **$$s = 10\%$$ (0.10)** công việc buộc phải chạy tuần tự (ví dụ: xin cấp phát số hóa đơn tăng dần từ 1 bảng khóa chung), và **90% (0.90)** còn lại song song hoàn toàn trên $$N$$ máy chủ:

| Số máy chủ $$N$$ | Công thức Amdahl $$S(N) = rac{1}{0.10 + 0.90/N}$$ | Mức tăng tốc thực tế | Hiệu suất sử dụng phần cứng ($$S(N)/N$$) |
|---|---|---|---|
| $$N = 1$$ | $$1 / (0.10 + 0.90)$$ | **1.00×** | 100% |
| $$N = 10$$ | $$1 / (0.10 + 0.09) = 1 / 0.19$$ | **5.26×** | 52.6% |
| $$N = 100$$ | $$1 / (0.10 + 0.009) = 1 / 0.109$$ | **9.17×** | 9.2% |
| $$N 	o \infty$$ | $$1 / 0.10$$ | **Tối đa 10.0×!** | Tiến về 0%! |

**Bài học xương máu:** Nếu không xóa bỏ điểm nghẽn tuần tự 10% (ví dụ chuyển sang sinh mã UUID/Snowflake phân tán không cần khóa chung), dù bạn ném tiền mua **1.000 máy chủ** thì hệ thống vẫn không bao giờ vượt qua ngưỡng **10×**!

### Ví dụ cầm tay chỉ việc 2: Tính chi phí điện năng thực tế theo chỉ số PUE

Một cụm máy chủ IT tiêu thụ thực tế **1.0 MW** điện liên tục suốt 1 năm (8.760 giờ), giá điện công nghiệp **$$0.10/	ext{kWh}$$**:
- **Trung tâm dữ liệu doanh nghiệp cũ (PUE = 2.0):** Tổng điện tiêu thụ $$= 2.0	ext{ MW} \implies 17{,}520{,}000	ext{ kWh/năm} \implies$$ tiền điện **$$1{,}752{,}000$$ USD/năm** (nửa hóa đơn chỉ để chạy điều hòa và tổn hao điện!).
- **Trung tâm dữ liệu Hyperscaler hiện đại (PUE = 1.12):** Tổng điện tiêu thụ $$= 1.12	ext{ MW} \implies$$ tiền điện **$$981{,}120$$ USD/năm** — tiết kiệm tới **$$770{,}880$$ USD mỗi năm** cho cùng một năng lực tính toán!

### Khung giải thích: 4 bước biến ứng dụng đơn máy thành hệ thống Scale-Out thực thụ (nên đọc chậm)

#### Bước 1: Đẩy toàn bộ trạng thái phiên (Session/File tạm) ra khỏi máy chủ ứng dụng
Nếu người dùng đăng nhập được lưu trong RAM của Máy A, bộ cân bằng tải không thể chuyển yêu cầu kế tiếp sang Máy B. Hãy chuyển phiên đăng nhập sang **Redis** hoặc **JWT**, còn ảnh tải lên đẩy thẳng vào **Object Storage (S3)** để tầng web hoàn toàn *stateless*.

#### Bước 2: Dùng luật hàng đợi (Little’s Law: $$L = \lambda W$$) để ước lượng sức chứa
Nếu lưu lượng đến là $$\lambda = 200	ext{ req/s}$$ và thời gian xử lý trung bình là $$W = 0.05	ext{ s}$$ (50 ms), số yêu cầu đang xử lý đồng thời trung bình là $$L = 200 	imes 0.05 = 10$$. Chỉ cần thời gian phản hồi CSDL chậm lên $$0.5	ext{ s}$$, số luồng kẹt đồng thời lập tức vọt lên **100** và làm cạn connection pool!

#### Bước 3: Mở rộng dọc đến ngưỡng hợp lý trước khi nhảy vào phức tạp của mở rộng ngang
Một máy CSDL PostgreSQL hiện đại 64 lõi + NVMe có thể gánh hàng chục nghìn truy vấn/giây. Hãy tối ưu chỉ mục (index), thêm bản sao đọc (read replicas) và cache trước khi vội vàng phân mảnh (sharding) phức tạp.

#### Bước 4: Thiết kế cho cái chết bình thường của phần cứng ("Cattle, not Pets")
Trong trung tâm dữ liệu 50.000 máy, mỗi ngày đều có hàng chục ổ đĩa và thanh RAM hỏng. Máy chủ phải được tạo tự động từ image chuẩn để khi một nút chết, bộ điều phối chỉ việc giết nó đi và dựng nút mới thay thế trong vài giây.

## Video tham khảo

- **Xem trước:** [Infra Coffee Time #30: Trung tâm dữ liệu là gì? Các loại TTDL phổ biến](https://www.youtube.com/watch?v=oDUHJxuVLiE) — Viettel IDC (tiếng Việt). Giới thiệu khái niệm trung tâm dữ liệu và các loại hình phổ biến bằng tiếng Việt.
- [Google Data Center 360° Tour](https://www.youtube.com/watch?v=zDAYZU4A3w0) — Google Cloud Tech (tiếng Anh). Tham quan thực tế không gian tủ rack, hệ thống điện và làm mát trong trung tâm dữ liệu.
- [Vertical Vs Horizontal Scaling: Key Differences You Should Know](https://www.youtube.com/watch?v=dvRFHG2-uYs) — ByteByteGo (tiếng Anh). So sánh trực quan mở rộng dọc (`scale up`) và mở rộng ngang (`scale out`).
