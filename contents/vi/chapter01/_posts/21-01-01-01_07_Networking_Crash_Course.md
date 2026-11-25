---
layout: post
title: 01-07 Nhập môn mạng đám mây
chapter: '01'
order: 8
owner: Nguyen Le Linh
lang: vi
categories:
- chapter01
lesson_type: optional
---

Khi ứng dụng chạy trên máy tính cá nhân, `localhost:8080` nối thẳng tới `localhost:5432` dễ đến mức ta gần như quên mất mạng máy tính tồn tại. Nhưng ngay phút đầu tiên đưa ứng dụng lên đám mây công cộng, mọi thứ đổi khác: đột nhiên trình duyệt báo lỗi chứng chỉ đỏ lòm, dịch vụ API không kết nối được tới PostgreSQL dù cả hai đều đang chạy, hoặc tệ hơn — một cổng cơ sở dữ liệu mở nhầm `0.0.0.0/0` bị bot quét toàn cầu phát hiện chỉ sau vài phút.

Đặc trưng *truy cập mạng rộng* của NIST dễ bị hiểu một cách chung chung cho đến khi bạn phải thiết kế để một gói tin đi tới đúng container — và *chỉ* container đó. Bài tùy chọn này tổng hợp ngắn gọn năm khái niệm mạng cốt lõi bạn sẽ gặp xuyên suốt từ Chương 08 trở đi: **VPC, subnet, security group, DNS và TLS**. Thay vì hướng dẫn thao tác từng nút bấm trên giao diện web, bài học tập trung xây dựng hệ thuật ngữ chung để khi làm việc với Docker, Kubernetes và Terraform, các khái niệm mạng trở nên rõ ràng và liên kết với nhau.

Bạn nên đọc bài này sau khi hoàn thành các bài bắt buộc của Chương 01, và xem lại trước khi thực hành [lab Lộ trình triển khai]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_01_Lab_Docker_Local %}).

## Kiến thức cần nhớ: Năm lớp trụ cột của mạng đám mây

Để không bao giờ nhầm lẫn giữa "không tìm thấy đường đi" và "bị tường lửa chặn", hãy đặt năm khái niệm mạng đám mây vào đúng vị trí của chúng trên đường đi của một gói tin:

1. **VPC & Phân hoạch CIDR (Không gian địa chỉ cô lập)**:
   Thay vì cắm dây vào một bộ chuyển mạch vật lý dùng chung, đám mây định nghĩa một mạng riêng ảo (SDN) với dải IP nội bộ riêng (ví dụ $$10.20.0.0/16$$). Đây là "ranh giới đất" cô lập hoàn toàn dải IP của bạn với hàng nghìn khách hàng khác trên cùng trung tâm dữ liệu.
2. **Subnet & Bảng định tuyến (Route Table — Đường đi của gói tin)**:
   Subnet cắt nhỏ VPC theo từng vùng sẵn sàng (AZ). Bảng định tuyến quyết định *hướng đi* của gói tin (ra Internet qua Internet Gateway, đi nhờ ra ngoài qua NAT Gateway, hay chỉ ở trong nội bộ VPC). Tính "công khai" hay "riêng tư" nằm ở bảng định tuyến, không nằm ở dải số IP.
3. **Security Group & Nguyên tắc đặc quyền tối thiểu (Tường lửa có trạng thái)**:
   Gắn trực tiếp lên giao diện mạng (NIC) của từng máy chủ hoặc dịch vụ, hoạt động theo mô hình *danh sách cho phép (allow-list) có trạng thái*. Nó trả lời câu hỏi: "Gói tin đến đúng địa chỉ rồi, nhưng có được phép bước vào cổng này không?"
4. **DNS (Hệ thống định vị theo tên)**:
   Ánh xạ tên miền dễ nhớ (`courseboard.example.edu`) hoặc tên dịch vụ nội bộ (`postgres.courseboard.svc`) sang địa chỉ IP động vốn thay đổi liên tục khi hệ thống tự động co giãn (autoscaling).
5. **TLS (Hợp đồng xác thực danh tính và mã hóa)**:
   Đảm bảo trình duyệt đang nói chuyện với *đúng* máy chủ sở hữu tên miền đó (qua chứng chỉ số do CA ký) và toàn bộ dữ liệu trên đường truyền được mã hóa chống nghe lén.

## Mục tiêu học tập

- Phác thảo một mạng ảo cô lập gồm subnet công khai và subnet riêng tư, giải thích rõ mục đích của từng loại subnet.
- Phân biệt **security group** (danh sách cho phép có trạng thái gắn trên giao diện mạng NIC) với **bảng định tuyến** (quyết định bước chuyển tiếp tiếp theo của gói tin).
- Giải thích vì sao **DNS** và **TLS** là một phần trong hợp đồng giao tiếp ở tầng ứng dụng.
- Ánh xạ cùng một thiết kế mạng sang hệ thuật ngữ tương ứng của AWS, Azure và GCP.

## 1. Vì sao mạng là thành phần hạ tầng cốt lõi trên đám mây

Trên máy tính cá nhân, `localhost` che giấu hầu hết chi tiết về đánh địa chỉ, định tuyến và phân giải tên miền. Trên đám mây công cộng, mọi tài nguyên đều nằm trong một **VPC (Virtual Private Cloud)**: mạng ảo định nghĩa bằng phần mềm với dải địa chỉ CIDR do bạn chọn (ví dụ $$10.0.0.0/16$$). Khách hàng khác không thể nhìn thấy dải IP riêng của bạn; dù vẫn chia sẻ hạ tầng mạng vật lý của nhà cung cấp (*resource pooling*), toàn bộ chính sách định tuyến và tường lửa trong VPC thuộc quyền kiểm soát của bạn.

Xét ví dụ ứng dụng **Bảng tin học phần** tại IUH: tầng API cần nhận yêu cầu HTTPS từ sinh viên ngoài Internet, giao tiếp với cơ sở dữ liệu *không* kết nối trực tiếp Internet, và chặn toàn bộ các luồng truy cập trái phép khác. Yêu cầu đó bản chất là một thiết kế VPC, không chỉ là một tham số cấu hình Docker.

```text
Internet
   │
   ▼
[Internet gateway] ── subnet công ── cân bằng tải / proxy ngược (80/443)
                           │
                     [NAT, nếu cần]
                           │
                     subnet riêng ── API + worker
                           │
                     subnet riêng ── cơ sở dữ liệu (không IP công)
```

**Câu hỏi thiết kế:** Nếu bạn cấp địa chỉ IP công khai cho cơ sở dữ liệu với lý do "chỉ dùng trong tuần demo", bạn không còn duy trì ranh giới subnet riêng tư nữa mà đang tạo ra một rủi ro bảo mật nghiêm trọng.

## 2. Subnet, định tuyến và khái niệm công khai / riêng tư

**Subnet** là một dải con CIDR tách ra từ VPC (ví dụ $$10.0.1.0/24$$), thường gắn với một vùng sẵn sàng (Availability Zone). Tính chất "công khai" (public) hay "riêng tư" (private) không nằm ở bản thân dải CIDR mà được quyết định bởi *cấu hình định tuyến*:

| Loại | Tuyến điển hình | Gán IP công khai |
| --- | --- | --- |
| Công khai | tuyến mặc định $$0.0.0.0/0$$ → Internet Gateway | Có (hoặc đứng sau bộ cân bằng tải công khai) |
| Riêng tư | tuyến mặc định → NAT Gateway, hoặc *không* có tuyến mặc định ra ngoài | Không |

**Bảng định tuyến (route table)** trả lời câu hỏi "gói tin sẽ đi tiếp tới đâu?", còn **security group** trả lời câu hỏi "gói tin có được phép đi qua giao diện mạng này hay không?". Người mới thường nhầm lẫn hai khái niệm này vì cả hai đều có thể khiến kết nối Internet bị chặn, nhưng chúng hoạt động ở hai tầng khác nhau.

Ba nguyên tắc thực hành cần ghi nhớ:

1. Đặt mọi **thành phần lưu trữ trạng thái** (PostgreSQL, Redis, MinIO...) trong subnet riêng tư; không mở lắng nghe `0.0.0.0/0` ra Internet chỉ để tiện thử nghiệm.
2. Đặt **điểm tiếp nhận lưu lượng** (bộ cân bằng tải hoặc reverse proxy) trong subnet công khai.
3. Khi máy chủ trong subnet riêng tư cần truy cập ra ngoài để tải gói phần mềm, lưu lượng phải đi qua NAT Gateway — lưu ý rằng NAT Gateway **có phát sinh chi phí**. Trong bài lab Docker chạy cục bộ, bạn không cần sử dụng NAT đám mây.

Về tên gọi, Azure gọi VPC là **Virtual Network (VNet)**, còn GCP gọi là **VPC network** với các subnet trải theo từng vùng (regional subnet), nhưng nguyên lý kiến trúc hoàn toàn tương đương.

## 3. Security group là danh sách cho phép (allow-list), không phải tường lửa truyền thống

**Security group (SG)** là tập hợp các quy tắc lọc lưu lượng *có trạng thái (stateful)* gắn vào máy ảo, ENI hoặc nhóm giao diện mạng. "Có trạng thái" nghĩa là khi bạn cho phép lưu lượng TCP cổng 443 đi vào, gói tin phản hồi tương ứng sẽ tự động được phép đi ra mà không cần khai báo quy tắc chiều ra. **NSG** trên Azure và **VPC firewall rules** trên GCP đảm nhận vai trò tương đương, dù giá trị mặc định có đôi chút khác biệt.

Nên viết quy tắc security group dựa trên *vai trò logic* của từng tầng:

```text
sg-edge:    vào 443 từ 0.0.0.0/0 ; ra tới sg-api cổng 8080
sg-api:     vào 8080 chỉ từ sg-edge
sg-db:      vào 5432 chỉ từ sg-api
```

Tham chiếu trực tiếp đến **security group nguồn** an toàn và bền vững hơn nhiều so với việc ghi cứng địa chỉ IP riêng của máy chủ API, vì IP có thể thay đổi khi mở rộng hoặc khởi tạo lại, còn vai trò "tầng API" thì không đổi.

<div class="content-box warning-box">
<p><strong>Tránh mở toàn bộ cổng như môi trường thử nghiệm.</strong> Cấu hình <code>0.0.0.0/0</code> cho cổng SSH (22) hoặc cổng cơ sở dữ liệu là lỗi phổ biến nhất trên tài khoản free-tier. Nên tránh mở SSH công khai (ưu tiên truy cập qua console, AWS SSM hoặc GCP IAP), hoặc chỉ giới hạn đúng địa chỉ IP của bạn trong thời gian ngắn rồi đóng lại.</p>
</div>

Nếu gặp khái niệm **NACL (Network ACL)**, hãy nhớ đây là lớp lọc *không trạng thái (stateless)* hoạt động ở cấp subnet. Trong phạm vi khóa học này, ưu tiên hàng đầu là thiết lập chính xác security group theo đúng ý định kiến trúc.

## 4. DNS là cơ chế giúp người dùng và dịch vụ định vị nhau

**DNS** ánh xạ tên miền sang các bản ghi tương ứng (A/AAAA, CNAME, MX, TXT) trên hai mặt phẳng chính:

- **DNS công khai:** Ví dụ `courseboard.example.edu` trỏ về bộ cân bằng tải công khai.
- **DNS riêng tư:** Ví dụ `api.internal.courseboard` hoặc `postgres.courseboard.svc` dùng để phân giải nội bộ mà không cần tồn tại trên Internet.

Cơ chế Service DNS trong Kubernetes (`*.svc.cluster.local`, Chương 09) áp dụng cùng nguyên lý này trong phạm vi cụm, còn **Private Hosted Zone** áp dụng cho máy ảo và cơ sở dữ liệu được quản lý.

```python
import socket

def resolve(name: str) -> str:
    return socket.getaddrinfo(name, 443, type=socket.SOCK_STREAM)[0][4][0]

# Phân giải thành công không có nghĩa TCP đã thông.
print(resolve("example.com"))
```

**TTL (Time to Live)** là thông số cần lưu ý khi vận hành: một bản ghi thử nghiệm có TTL 5 giây và một tên miền chính thức có TTL 24 giờ sẽ có tốc độ cập nhật và hoàn tác (rollback) rất khác nhau.

## 5. TLS là hợp đồng xác thực và mã hóa gắn với tên miền

**TLS** thực hiện đồng thời hai nhiệm vụ: xác thực danh tính *máy chủ* với máy khách và mã hóa dữ liệu trên đường truyền. Bốn điểm thực hành quan trọng cần nắm:

1. **Chứng chỉ số** gắn **khóa công khai** với một hoặc nhiều **tên miền** (trường SAN), được ký bởi một tổ chức cấp phát chứng chỉ (CA) mà trình duyệt tin cậy.
2. Trình duyệt đối chiếu **tên miền người dùng nhập** với chứng chỉ; truy cập trực tiếp qua địa chỉ IP như `https://10.0.1.23` sẽ gây lỗi xác thực chứng chỉ.
3. Trong môi trường sản xuất, **HTTP cổng 80** chỉ nên dùng để chuyển hướng sang HTTPS; cổng phục vụ chính là **443**.
4. Giao tiếp nội bộ giữa các dịch vụ vẫn cần mã hóa bằng TLS hoặc cơ chế định danh của service mesh (Chương 09); "nằm trong mạng IP riêng" không đồng nghĩa với việc dữ liệu đã được mã hóa.

Các dịch vụ như Let’s Encrypt và trình quản lý chứng chỉ của nhà cung cấp đám mây giúp tự động hóa cấp phát và gia hạn mà không cần tự vận hành CA. Trong môi trường thử nghiệm cục bộ, có thể dùng `mkcert`, Caddy hoặc nginx, và tuyệt đối không đưa khóa riêng (private key) lên Git.

```python
import ssl
import socket

def peer_name(host: str) -> str:
    ctx = ssl.create_default_context()
    with socket.create_connection((host, 443), timeout=5) as raw:
        with ctx.wrap_socket(raw, server_hostname=host) as tls:
            return tls.getpeercert()["subject"]

print(peer_name("example.com"))
```

<div class="content-box insight-box">
<p><strong>DNS và TLS gắn liền với trải nghiệm người dùng cuối.</strong> Một hệ thống VPC cấu hình chuẩn nhưng dùng chứng chỉ tự ký trên địa chỉ IP trần vẫn sẽ hiển thị cảnh báo lỗi trên điện thoại người dùng. Hãy gán tên miền cho điểm biên (edge) và thực hiện kết thúc phiên TLS (TLS termination) trên chính tên miền đó.</p>
</div>

## 6. Ánh xạ thuật ngữ giữa ba nhà cung cấp đám mây lớn

| Khái niệm | AWS | Azure | GCP |
| --- | --- | --- | --- |
| Mạng cô lập | VPC | Virtual Network | VPC network |
| Phân đoạn mạng theo AZ | Subnet | Subnet | Subnet |
| Allow-list có trạng thái | Security group | NSG | VPC firewall rule |
| Kết nối công khai | IGW + subnet công khai | Public IP / LB | Cloud NAT / LB + subnet |
| Phân giải tên công khai | Route 53 | Azure DNS | Cloud DNS |
| Quản lý chứng chỉ TLS | ACM | Key Vault / Front Door | Certificate Manager |

## 7. Ví dụ thiết kế quy mô trường học

Kiến trúc mạng gợi ý cho ứng dụng **Bảng tin học phần IUH**:

1. Tạo một VPC với dải $$10.20.0.0/16$$ trong một vùng (region); chỉ mở rộng sang hai vùng sẵn sàng (AZ) khi yêu cầu bài toán đặt nặng tính dự phòng cao (hai AZ sẽ làm tăng chi phí — xem thêm bài FinOps).
2. Subnet công khai chỉ đặt bộ cân bằng tải.
3. Subnet riêng tư đặt máy chủ API và cơ sở dữ liệu.
4. `sg-db` chỉ cho phép cổng `5432` đi vào từ `sg-api`.
5. DNS công khai trỏ về bộ cân bằng tải.
6. Cấp chứng chỉ TLS cho tên miền đó và cấu hình chuyển hướng HTTP sang HTTPS.

Khi thực hành với Docker Compose trên máy cá nhân (Lab A), bạn sẽ mô phỏng *cùng cấu trúc phân tầng* này thông qua mạng Docker do người dùng tự định nghĩa.

### Giải thích: Tính tay quy hoạch CIDR và lần dấu từng bước một gói tin HTTPS

Hãy làm thử hai bài tập nhỏ bằng tay để biến lý thuyết mạng thành phản xạ thực chiến.

#### Bước 1: Tính số địa chỉ IP khả dụng khi chia nhỏ VPC $$10.20.0.0/16$$

Một địa chỉ IPv4 có đúng **32 bit**. Phần tiền tố `/16` giữ cố định 16 bit đầu (`10.20.*.*`), để lại $$32 - 16 = 16$$ bit cho máy chủ:
$$\text{Tổng IP của VPC } 10.20.0.0/16 = 2^{32-16} = 2^{16} = \mathbf{65.536\text{ địa chỉ}}.$$

Bây giờ ta tách ra các subnet `/24` (ví dụ `10.20.1.0/24` cho tầng công khai, `10.20.2.0/24` cho tầng API và `10.20.3.0/24` cho tầng CSDL):
- Mỗi subnet `/24` có $$2^{32-24} = 2^8 = \mathbf{256\text{ địa chỉ}}$$.
- Trên AWS VPC, nhà cung cấp luôn giữ lại **5 địa chỉ** mỗi subnet (địa chỉ mạng `.0`, router VPC `.1`, DNS `.2`, dự phòng `.3`, và broadcast `.255`), nên mỗi subnet `/24` cấp thực tế được $$256 - 5 = \mathbf{251\text{ máy/Pod}}$$ — quá dư dả cho dự án sinh viên nhưng lại vừa vặn để dễ nhớ theo từng nhóm số `10.20.1.x`, `10.20.2.x`, `10.20.3.x`.

#### Bước 2: Lần dấu gói tin khi sinh viên mở `https://courseboard.iuh.edu.vn/courses`

| Chặng | Cửa kiểm soát đi qua | Điều gì xảy ra nếu cấu hình sai? |
| :--- | :--- | :--- |
| 1. Phân giải tên | DNS công khai trả về IP của Load Balancer (ví dụ `203.0.113.10`) | Báo lỗi `NXDOMAIN` — gói tin chưa kịp rời khỏi trình duyệt |
| 2. Vào biên VPC | Bảng định tuyến subnet công khai có tuyến `0.0.0.0/0 → IGW`; `sg-edge` mở inbound `443` | Treo kết nối (`Connection timed out`) vì gói tin bị drop lặng lẽ |
| 3. Bắt tay TLS | Load Balancer trình chứng chỉ SAN khớp `courseboard.iuh.edu.vn` | Trình duyệt chặn với cảnh báo `NET::ERR_CERT_COMMON_NAME_INVALID` |
| 4. Gọi tầng API | Load Balancer chuyển tiếp tới `10.20.2.15:8080`; `sg-api` chỉ nhận cổng `8080` từ `sg-edge` | Trả về `502 Bad Gateway` hoặc `504 Gateway Timeout` |
| 5. Truy vấn CSDL | API gọi `10.20.3.20:5432`; `sg-db` chỉ nhận cổng `5432` từ `sg-api` | API ghi log `psycopg2.OperationalError: timeout`, trong khi kẻ tấn công từ Internet hoàn toàn không chạm được tới `10.20.3.20` |

Nhìn vào bảng 5 chặng trên, mỗi lần gặp lỗi mạng trên đám mây, bạn chỉ cần kiểm tra đúng từ trên xuống dưới (DNS → Route Table → Security Group → TLS → Cổng ứng dụng) thay vì đoán mò.

## Thách thức thường gặp

- Xung đột dải CIDR khi về sau cần kết nối liên VPC (VPC peering) hoặc nối VPN về mạng nội bộ của tổ chức.
- Quên tắt NAT Gateway và bộ cân bằng tải sau buổi demo khiến chi phí tiếp tục phát sinh.
- Quy tắc `0.0.0.0/0` mở tạm lúc kiểm thử vô tình bị giữ nguyên khi đưa lên môi trường sản xuất.
- Chứng chỉ TLS hết hạn vào cuối tuần do không có cơ chế gia hạn tự động hoặc không phân công người theo dõi.

## Video tham khảo

- **Xem trước:** [AWS VPC & Subnets For Beginners](https://www.youtube.com/watch?v=TUTqYEZZUdc) — Sam Meech-Ward (tiếng Anh). Giải thích trực quan về VPC và subnet — các thành phần mạng nền tảng của bài.
- [How DNS Works - Computerphile](https://www.youtube.com/watch?v=uOfonONtIuk) — Computerphile (tiếng Anh). Trình bày cơ chế phân giải DNS giúp máy khách định vị dịch vụ.
- [SSL, TLS, HTTPS Explained](https://www.youtube.com/watch?v=j9QmMEWmcfo) — ByteByteGo (tiếng Anh). Minh họa vai trò của TLS và HTTPS trong việc xác thực tên miền và mã hóa kết nối.

## Bài tập

1. Vẽ sơ đồ mạng cho ứng dụng Bảng tin học phần gồm ba security group, *không* cấp IP công khai cho cơ sở dữ liệu, và ghi rõ tuyến định tuyến làm nên tính chất công khai của subnet công khai.
2. Khi một sinh viên khác dùng `dig` phân giải được tên miền nhưng trình duyệt vẫn bị treo kết nối, hãy liệt kê ba tầng cần kiểm tra (DNS, Security Group, TLS) và nội dung cụ thể cần xác minh ở mỗi tầng.
3. Viết lại hai quy tắc bảo mật chưa an toàn sau theo nguyên tắc đặc quyền tối thiểu: `sg-api vào 0.0.0.0/0 1-65535`, `sg-db vào 0.0.0.0/0 5432`.
4. Chỉ dựa vào bảng ở Mục 6, gọi tên các dịch vụ tương ứng trên Azure và GCP cho cấu hình "subnet riêng tư + danh sách cho phép kiểu security group + DNS công khai".

## Đọc tiếp

- [08-03 Docker]({{ site.baseurl }}{% multilang_post_url contents/chapter08/21-01-01-08_03_Docker %})
- [09-03 Service]({{ site.baseurl }}{% multilang_post_url contents/chapter09/21-01-01-09_03_Services_Deployments %})
- [13-04 CI/CD]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_04_CICD_Cloud_Apps %})
- [Hub Lộ trình triển khai]({{ site.baseurl }}/contents/vi/chapter15/)

## Tài liệu tham khảo

1. NIST SP 800-145.
2. Tài liệu chính thức về VPC/VNet của AWS, Azure và GCP.
3. RFC 5280 và RFC 8446 — tập trung vào nguyên lý *ràng buộc tên miền với chứng chỉ*.
4. [infracourse.cloud](https://infracourse.cloud/) (Stanford CS 40) — tham khảo cách tiếp cận giới thiệu mạng đám mây trước khi khởi chạy máy ảo; phần trình bày cho IUH được biên soạn độc lập.
