---
layout: post
title: 15-01 Lab A — Ứng dụng đa tầng local với Docker Compose
chapter: '15'
order: 2
owner: Nguyen Le Linh
lang: vi
categories:
- chapter15
lesson_type: optional
---

**Hình thức:** Bài thực hành dạng checklist (không tính điểm trừ khi giảng viên quy định khác).  
**Thời lượng dự kiến:** Khoảng 2–3 giờ tập trung cho một nhóm hai sinh viên.  
**Yêu cầu đám mây:** Không. Nếu máy chưa cài Docker, hãy cài Docker Engine *hoặc* Docker Desktop; không thay thế bằng máy ảo đám mây công cộng.

Bài thực hành này được biên soạn riêng cho IUH, **không** sử dụng đề bài của Stanford CS 40.

Trước khi đụng tới Kubernetes hay một tài khoản đám mây có tính phí, mọi kỹ sư hệ thống đều cần làm chủ hoàn toàn "phòng thí nghiệm thu nhỏ" ngay trên laptop cá nhân: đóng gói ứng dụng thành ảnh container tinh gọn, nối nhiều dịch vụ qua mạng riêng Docker Compose, kiểm tra sức khỏe bằng `healthcheck`, và bảo toàn dữ liệu bền vững qua Volume.

## Bản đồ kiến thức nền: Bốn kỹ năng cốt lõi của thực hành Container cục bộ

1. **Multi-Stage Build (Đóng gói đa tầng tối ưu dung lượng & bảo mật)**:
   Giai đoạn biên dịch (build) cần trình biên dịch (`gcc`, `cargo`, `npm`), nhưng giai đoạn chạy thật (runtime) hoàn toàn không cần chúng. **Multi-stage Dockerfile** biên dịch ở tầng `builder` rồi chỉ copy đúng tệp nhị phân/mã chạy sang ảnh nền siêu nhẹ (`slim` hoặc `distroless`), giảm tới $80\%\text{–}95\%$ dung lượng và thu nhỏ bề mặt tấn công.

2. **Mạng cầu nối riêng (User-Defined Bridge Network) & DNS nội bộ**:
   Trong `docker compose`, mọi dịch vụ trong cùng một mạng tự động tìm thấy nhau bằng **tên dịch vụ** (ví dụ API kết nối Postgres qua chuỗi `postgres://db:5432/app` thay vì `localhost:5432`—vì `localhost` bên trong container API là chính container API, không phải container DB!).

3. **Tính phù du của Container vs. Volume bền vững**:
   Mọi tệp ghi vào lớp ghi mặc định của container sẽ biến mất ngay khi chạy `docker compose down`. Dữ liệu cơ sở dữ liệu bắt buộc phải gắn vào **Named Volume** để sống sót qua mọi lần nâng cấp ảnh.

4. **Nguyên tắc bảo mật tối thiểu ngay ở Local (Non-root & `.env`)**:
   Chạy tiến trình trong container bằng tài khoản thường (`USER appuser`) thay vì `root`, và không bao giờ đưa tệp `.env` chứa mật khẩu vào Git hoặc `COPY . .` vào trong ảnh (luôn liệt kê `.env` trong `.dockerignore`).

## Mục đích

Xây dựng ứng dụng **Bảng tin học phần gồm ba tầng** chạy cục bộ trên máy tính cá nhân: lớp biên (edge/trình duyệt), lớp API và lớp cơ sở dữ liệu. Sau khi chạy thành công, bạn cần chỉ rõ từng tiến trình tương ứng với tầng kiến trúc nào trong bài 01-07 (biên / ứng dụng / dữ liệu).

## Chuẩn đầu ra

- Một tệp `docker-compose.yml` định nghĩa **hai mạng tách biệt** (ví dụ `edge` và `persist`) sao cho cổng cơ sở dữ liệu không mở ra mạng LAN của máy chủ.
- Endpoint `/healthz` trên API trả về lỗi khi container cơ sở dữ liệu bị dừng.
- Nhật ký (log) xuất ra `stdout` có đính kèm `request_id` (khuyến khích bổ sung `trace_id`).
- Bản ghi chú `NOTES.md` dài khoảng nửa trang kèm sơ đồ kiến trúc.

## Ứng dụng gợi ý (tự lập trình)

**Bảng tin học phần IUH (bản cục bộ):** Cho phép xem danh sách và đăng *bài nhờ hỗ trợ học phần* (gồm mã học phần, tiêu đề và nội dung). Tuyệt đối không dùng thông tin sinh viên thật; chỉ dùng dữ liệu giả lập (như `sv0001`).

| Phương thức | Đường dẫn | Hành vi mong đợi |
| --- | --- | --- |
| GET | `/healthz` | Trả về 200 nếu kết nối cơ sở dữ liệu thành công; trả về 503 nếu thất bại |
| GET | `/api/posts` | Trả về danh sách bài đăng dạng JSON |
| POST | `/api/posts` | Tạo bài đăng mới từ JSON; từ chối nếu tiêu đề rỗng |

Lớp biên có thể dùng Caddy, nginx hoặc giao diện tĩnh phục vụ qua API. Cần cố định phiên bản thẻ ảnh container cụ thể (không dùng thẻ `latest`).

## Danh sách kiểm tra (Checklist)

### 0. Chuẩn bị môi trường

- [ ] Ghi tên hai thành viên và ngày thực hành ở đầu tệp `NOTES.md`.
- [ ] Cấu hình `.gitignore` loại trừ `.env`, `*.pem` và `__pycache__`.
- [ ] Xác nhận hai lệnh `docker version` và `docker compose version` hoạt động bình thường.

### 1. Phân tách mạng thay vì mở cổng tràn lan

- [ ] `docker-compose.yml` khai báo hai mạng riêng biệt kiểu `edge` và `persist`.
- [ ] Container cơ sở dữ liệu **không** khai báo `ports: ["5432:5432"]` (hoặc Redis `6379`) trên `0.0.0.0`; chỉ dịch vụ API kết nối vào mạng `persist`.
- [ ] Dịch vụ lớp biên ánh xạ cổng `127.0.0.1:8080:80` thay vì `8080:80` mở công khai trên mọi giao diện mạng.

### 2. Kiểm tra trạng thái hoạt động và mô phỏng sự cố

- [ ] Cấu hình `healthcheck` trong Compose cho cả cơ sở dữ liệu *và* API.
- [ ] Xác nhận `docker compose ps` hiển thị trạng thái `healthy` trước khi gửi yêu cầu thử nghiệm.
- [ ] **Chủ động dừng container cơ sở dữ liệu**, xác nhận `/healthz` trả về mã 503 và lưu lại đoạn log của API, sau đó khởi động lại.

### 3. Khả năng quan sát tối thiểu

- [ ] API ghi một dòng log JSON cho mỗi yêu cầu gồm: `method`, `path`, `status`, `duration_ms`, `request_id`.
- [ ] Không ghi nội dung thân yêu cầu (request body) có thể chứa thông tin cá nhân vào log.

### 4. Mô phỏng mở rộng ngang (elasticity)

- [ ] Chạy thử `docker compose up --scale api=2` *hoặc* giải thích rõ trong ghi chú lý do lớp biên hiện tại chưa cân bằng tải được giữa hai phiên bản API.
- [ ] Nếu việc mở rộng ngang gặp lỗi (ví dụ do lưu trạng thái trong bộ nhớ tiến trình), hãy ghi nhận lại như một phát hiện kỹ thuật.

### 5. Tiêu chí hoàn thành

- [ ] Truy cập `http://127.0.0.1:8080` từ trình duyệt trên **máy của bạn** và tạo được bài đăng mới.
- [ ] `NOTES.md` có sơ đồ thể hiện rõ ba tầng dịch vụ và hai mạng.
- [ ] Ghi chú rõ lệnh `docker compose down -v` và hiểu rằng cờ `-v` sẽ xóa dữ liệu volume.

## Mục tiêu mở rộng (vẫn chạy cục bộ)

- Cấu hình TLS cục bộ tại lớp biên bằng `mkcert` cho tên miền `courseboard.localhost`.
- Viết tệp `Makefile` với các lệnh `make up`, `make test` và `make down`.
- Tích hợp kiểm thử nhanh (smoke test) từ bài 13-04 cho endpoint `/healthz`.

### Khung giải thích (nên đọc chậm): So sánh ảnh Single-Stage (`920 MB`) vs. Multi-Stage (`28 MB`) và trace lỗi `localhost` kinh điển trong Docker Compose

#### 1. Tại sao Multi-Stage Build thu nhỏ ảnh hơn $30\times$?

Giả sử bạn đóng gói một dịch vụ API nhỏ (viết bằng Go/Rust hoặc Python/Node). So sánh hai cách viết `Dockerfile`:

| Cách đóng gói | Thành phần nằm bên trong ảnh cuối cùng | Dung lượng ước lượng | Số lỗ hổng CVE tiềm ẩn |
| :--- | :--- | :---: | :---: |
| **Single-stage ngây thơ** (`FROM golang:1.22` hoặc `node:20`) | Toàn bộ OS Debian đầy đủ + trình biên dịch + công cụ git/curl + mã nguồn + cache build | **`~850–920 MB`** | Cao (hàng chục gói OS dư thừa) |
| **Multi-stage chuẩn** (`builder` → `distroless`/`alpine`/`slim`) | Chỉ đúng tệp thực thi đã biên dịch + chứng chỉ CA gốc + `USER nonroot` | **`~18–28 MB`** | Gần như bằng **0** (không có cả shell `/bin/sh` cho kẻ tấn công!) |

#### 2. Trace lỗi kết nối DB phổ biến nhất ở bài lab (`Connection refused` trên `localhost:5432`)

Khi sinh viên chạy `docker compose up` với 2 dịch vụ `api` và `db`:
- **Cấu hình sai**: `DATABASE_URL=postgres://user:pass@localhost:5432/board`.
  - Container `api` có **Network Namespace riêng** (`172.20.0.3`). Gọi `localhost:5432` bên trong `api` nghĩa là hỏi cổng `5432` trên chính `172.20.0.3` → **không có gì lắng nghe → `Connection refused`!**
- **Cấu hình đúng**: `DATABASE_URL=postgres://user:pass@db:5432/board`.
  - Container `api` hỏi DNS nội bộ của Docker (`127.0.0.11`) địa chỉ tên miền `db` → Docker DNS trả về IP của container Postgres (`172.20.0.2`) → kết nối TCP `172.20.0.3 → 172.20.0.2:5432` thành công ngay lập tức!

## Những điều không nên làm

- Không mở cổng cơ sở dữ liệu ra mạng Wi-Fi của trường.
- Không đẩy ảnh container có chứa tệp `.env` lên registry.
- Không dùng `network_mode: host` như một đường tắt để bỏ qua bước cấu hình mạng.

## Minh chứng gợi ý cho giảng viên

Bộ minh chứng gồm tệp `NOTES.md`, tệp `docker-compose.yml` và ảnh chụp hoặc log khi hệ thống trả về mã 503 trong kịch bản tắt cơ sở dữ liệu.

## Video tham khảo

- **Xem trước:** [Tự học Docker siêu tốc trong 10 phút (Có code mẫu và ví dụ) - Code Dạo Dễ Òm](https://www.youtube.com/watch?v=1k8pox8mkxc) — Phạm Huy Hoàng (tiếng Việt). Ôn nhanh các lệnh Docker cơ bản bằng tiếng Việt trước khi ghép nhiều dịch vụ bằng Compose.
- [What is Docker Compose? \| Docker Concepts](https://www.youtube.com/watch?v=f3xb7kt_dH4) — Docker (tiếng Anh). Giải thích mục đích của Docker Compose trong bài toán chạy ứng dụng đa tầng cục bộ.
- [Docker Compose in 12 Minutes](https://www.youtube.com/watch?v=Qw9zlE3t8Ko) — Jake Wright (tiếng Anh). Minh họa tệp Compose gồm nhiều dịch vụ, gần với cấu trúc biên – ứng dụng – dữ liệu của bài lab.
