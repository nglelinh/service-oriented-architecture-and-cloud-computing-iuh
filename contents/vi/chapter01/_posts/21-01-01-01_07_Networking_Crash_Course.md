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

Đặc trưng NIST *truy cập mạng theo nhu cầu* dễ trở thành khẩu hiệu cho đến khi một gói tin phải tới đúng container — và *chỉ* container đó. Bài tùy chọn này là bản đồ ngắn, viết cho IUH, về năm đối tượng bạn sẽ gặp từ Chương 08 trở đi: **VPC, subnet, security group, DNS và TLS**. Đây không phải hướng dẫn bấm console. Mục tiêu là có ngôn ngữ chung để Docker, Kubernetes và Terraform không còn là chuỗi từ viết tắt.

Đọc sau các bài bắt buộc Chương 01, rồi đọc lại trước [lab Lộ trình triển khai]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_01_Lab_Docker_Local %}).

## Mục tiêu học tập

- Phác một mạng ảo cô lập với subnet công khai / riêng và nói *subnet đó để làm gì*.
- Phân biệt **security group** (danh sách cho phép có trạng thái trên NIC) với bảng định tuyến (gói đi đâu tiếp theo).
- Giải thích vì sao **DNS** và **TLS** là hợp đồng phía ứng dụng, không phải trang trí.
- Ánh xạ cùng một thiết kế sang tên gọi AWS, Azure, GCP mà không thuộc lòng mọi SKU.

## 1. Vì sao “mạng” là đối tượng đám mây hạng nhất

Trên laptop, `localhost` giấu địa chỉ, định tuyến và phân giải tên. Trên đám mây công cộng, mọi tài nguyên nằm trong **VPC**: mạng định nghĩa bằng phần mềm, với khối CIDR bạn chọn (ví dụ $$10.0.0.0/16$$). Khách hàng khác không thấy IP riêng của bạn. Bạn vẫn chia sẻ fabric vật lý của nhà cung cấp — đó là *resource pooling* — nhưng chính sách định tuyến và tường lửa là của bạn.

Hình dung IUH: API **Bảng tin học phần** nhận HTTPS từ sinh viên trên internet, nói chuyện với cơ sở dữ liệu *không* nằm trên internet, và từ chối mọi đường vào khác. Đó đã là thiết kế VPC, không phải một cờ Docker.

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

**Câu hỏi thiết kế.** Nếu cơ sở dữ liệu có IP công “chỉ trong tuần demo”, bạn không có subnet riêng. Bạn có một quả bom hẹn giờ.

## 2. Subnet, tuyến, và “công / riêng”

**Subnet** là lát CIDR của VPC (ví dụ $$10.0.1.0/24$$) gắn một vùng sẵn sàng. “Công” và “riêng” không phải kiểu ma thuật trong CIDR. Chúng là *sự thật định tuyến*:

| Loại | Tuyến điển hình | Ai giữ IP công |
| --- | --- | --- |
| Công | mặc định $$0.0.0.0/0$$ → internet gateway | có (hoặc cân bằng tải công phía trước) |
| Riêng | mặc định → NAT, hoặc *không* có tuyến mặc định | không |

**Bảng định tuyến** trả lời “gói đi đâu tiếp?”. **Security group** trả lời “gói này có được phép tại NIC này không?”. Sinh viên hay lẫn vì cả hai đều có thể “chặn internet” — ở các tầng khác nhau.

Ba quy tắc thực dụng:

1. Đặt **kho trạng thái** vào subnet riêng. Đừng để PostgreSQL, Redis hay MinIO lắng nghe `0.0.0.0/0` “cho nhanh”.
2. Đặt **lối vào** (cân bằng tải hoặc một proxy) ở subnet công.
3. Nếu subnet riêng cần tải gói, đó là việc của NAT — và NAT **có tính tiền**. Lab Docker local không cần NAT đám mây.

Azure gọi VPC là **Virtual Network (VNet)**. GCP gọi là **VPC network**, thường dùng subnet theo vùng. Sơ đồ trên vẫn đúng.

## 3. Security group là allow-list, không phải “sản phẩm firewall”

**Security group (SG)** là tập luật *có trạng thái* gắn instance, ENI, hoặc nhóm NIC. Có trạng thái nghĩa là: cho TCP 443 vào thì gói trả lời được ra mà không cần luật ra khớp. **NSG** (Azure) và **VPC firewall rules** (GCP) cùng vai trò, mặc định khác nhau.

Viết luật SG theo *ý định*:

```text
sg-edge:    vào 443 từ 0.0.0.0/0 ; ra tới sg-api cổng 8080
sg-api:     vào 8080 chỉ từ sg-edge
sg-db:      vào 5432 chỉ từ sg-api
```

Tham chiếu **security group khác** tốt hơn dán IP riêng hiện tại của API. IP đổi; danh tính “tầng API” không nên đổi.

<div class="content-box warning-box">
<p><strong>Mở hết là mùi lab.</strong> <code>0.0.0.0/0</code> trên SSH (22) hoặc cổng cơ sở dữ liệu là tai nạn free-tier phổ biến nhất. Ưu tiên không mở SSH (console / SSM / IAP) hoặc SSH từ IP của bạn vài giờ rồi đóng.</p>
</div>

NACL (nếu gặp) là lọc subnet *không trạng thái*. Trong khóa này, làm đúng ý định SG trước.

## 4. DNS là cách người và dịch vụ tìm VIP

**DNS** ánh xạ tên tới bản ghi (A/AAAA, CNAME, MX, TXT). Có hai mặt phẳng:

- **DNS công** — `courseboard.example.edu` → cân bằng tải.
- **DNS riêng** — `api.internal.courseboard` hoặc `postgres.courseboard.svc` → địa chỉ không cần tồn tại trên internet công.

DNS Service của Kubernetes (`*.svc.cluster.local`) là cùng ý tưởng trong cụm (Chương 09). **Private hosted zone** là cùng ý tưởng cho VM và cơ sở dữ liệu quản lý.

```python
import socket

def resolve(name: str) -> str:
    return socket.getaddrinfo(name, 443, type=socket.SOCK_STREAM)[0][4][0]

# Phân giải thành công không có nghĩa TCP đã thông.
print(resolve("example.com"))
```

**TTL** là chi tiết triển khai: tên lab năm giây và tên trường 24 giờ không rollback giống nhau.

## 5. TLS là hợp đồng trên tên đó

**TLS** xác thực *máy chủ* với máy khách và mã hóa byte. Bốn sự thật cho bài tập:

1. **Chứng chỉ** gắn **khóa công** với một **tên** (SAN), do CA mà trình duyệt đã tin ký.
2. Trình duyệt khớp **tên người dùng gõ** với chứng chỉ. `https://10.0.1.23` sẽ chống lại bạn.
3. **HTTP 80** trong thiết kế production chỉ là chuyển hướng. Cổng thật là **443**.
4. Dịch vụ riêng vẫn cần TLS *hoặc* danh tính mesh (Chương 09). “Có IP riêng” không phải mã hóa.

Let’s Encrypt và chứng chỉ do nhà cung cấp quản lý tồn tại để bạn không tự làm CA. Lab local: `mkcert` hoặc Caddy/nginx. Đừng commit khóa riêng.

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
<p><strong>DNS + TLS là một câu chuyện người dùng.</strong> VPC hoàn hảo với chứng chỉ tự ký trên IP thô vẫn “hỏng” trên điện thoại bạn cùng lớp. Đặt tên cho cạnh, rồi chấm dứt TLS trên tên đó.</p>
</div>

## 6. Cùng thiết kế, ba bộ từ vựng

| Ý niệm | AWS | Azure | GCP |
| --- | --- | --- | --- |
| Mạng cô lập | VPC | Virtual Network | VPC network |
| Lát + AZ | Subnet | Subnet | Subnet |
| Allow-list có trạng thái | Security group | NSG | VPC firewall rule |
| Đường công | IGW + subnet công | Public IP / LB | Cloud NAT / LB + subnet |
| Tên công | Route 53 | Azure DNS | Cloud DNS |
| Chứng chỉ quản lý | ACM | Key Vault / Front Door | Certificate Manager |

## 7. Ví dụ kích thước campus

**Bảng tin học phần IUH:**

1. VPC $$10.20.0.0/16$$, một vùng; hai AZ chỉ khi giảng viên yêu cầu phác độ bền (hai AZ đắt hơn — xem bài FinOps).
2. Subnet công chỉ giữ cân bằng tải.
3. Subnet riêng giữ API và cơ sở dữ liệu.
4. `sg-db` nhận 5432 chỉ từ `sg-api`.
5. DNS công → cân bằng tải.
6. Chứng chỉ TLS cho tên đó; HTTP chuyển HTTPS.

Compose trên laptop hiện *cùng các tầng* bằng mạng do người dùng định nghĩa (Lab A).

## Thách thức

- CIDR chồng khi sau này peering VPC hoặc VPN nhà trường.
- Giờ NAT và cân bằng tải sống lâu hơn buổi demo.
- Luật `0.0.0.0/0` “tạm” đi vào ảnh chụp rồi vào production.
- Chứng chỉ hết hạn Chủ nhật vì không ai sở hữu gia hạn.

## Bài tập

1. Vẽ Bảng tin với ba security group, *không* IP công trên cơ sở dữ liệu. Ghi tuyến khiến subnet công là công.
2. Bạn cùng lớp `dig` được tên nhưng trình duyệt treo. Liệt kê ba tầng (DNS, SG, TLS) và thứ sẽ kiểm.
3. Viết lại luật xấu: `sg-api vào 0.0.0.0/0 1-65535`, `sg-db vào 0.0.0.0/0 5432`.
4. Chỉ dùng bảng §6, nêu tên Azure và GCP cho “subnet riêng + allow-list kiểu SG + DNS công.”

## Tiếp theo

- [08-03 Docker]({{ site.baseurl }}{% multilang_post_url contents/chapter08/21-01-01-08_03_Docker %})
- [09-03 Service]({{ site.baseurl }}{% multilang_post_url contents/chapter09/21-01-01-09_03_Services_Deployments %})
- [13-04 CI/CD]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_04_CICD_Cloud_Apps %})
- [Hub Lộ trình triển khai]({{ site.baseurl }}/contents/vi/chapter15/)

## Tài liệu

1. NIST SP 800-145.
2. Tài liệu VPC/VNet của AWS, Azure, GCP.
3. RFC 5280 và RFC 8446 — ý *ràng buộc tên*, không phải cài stack.
4. [infracourse.cloud](https://infracourse.cloud/) (Stanford CS 40) — cảm hứng đặt mạng trước “cứ chạy một VM.” Bài IUH là nguyên tác.
