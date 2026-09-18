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

**Hình thức:** lab checklist (không tính điểm trừ khi giảng viên nói khác).  
**Khung giờ:** khoảng 2–3 giờ tập trung cho một cặp.  
**Đám mây:** không. Nếu chưa có Docker, dừng lại và cài Docker Engine *hoặc* Docker Desktop; đừng thay bằng VM công cộng.

Lab nguyên tác IUH. **Không** phải đề Stanford CS 40.

## Mục đích

Dựng **Bảng tin học phần ba tiến trình** trên laptop: cạnh trình duyệt, API, và kho dữ liệu. Chỉ được từng tiến trình và nói nó tương ứng tầng nào trong bài 01-07 (cạnh / ứng dụng / dữ liệu).

## Chuẩn đầu ra

- File Compose với **hai mạng do người dùng định nghĩa** (hoặc một frontend + một backend) để cổng kho dữ liệu không mở ra LAN máy chủ.
- `/healthz` trên API thất bại nếu kho dữ liệu tắt.
- Log stdout có `request_id` (có `trace_id` càng tốt).
- Ghi chú nửa trang kèm sơ đồ.

## Ứng dụng gợi ý (tự viết mã)

**Bảng tin IUH (local):** liệt kê và tạo *bài nhờ hỗ trợ học phần* (mã học phần, tiêu đề, nội dung). Không thông tin sinh viên thật. Tên tổng hợp (`sv0001`, không phải MSSV bạn cùng lớp).

| Phương thức | Đường | Hành vi |
| --- | --- | --- |
| GET | `/healthz` | 200 nếu ping DB được; 503 nếu không |
| GET | `/api/posts` | danh sách JSON |
| POST | `/api/posts` | tạo JSON; từ chối tiêu đề rỗng |

Cạnh có thể là Caddy, nginx, hoặc UI tĩnh do API phục vụ. Ghim thẻ ảnh (không `latest`).

## Checklist

### 0. Vệ sinh

- [ ] Tên cặp và ngày trên đầu `NOTES.md`.
- [ ] `.gitignore` loại `.env`, `*.pem`, `__pycache__`.
- [ ] Chạy được `docker version` và `docker compose version`.

### 1. Mạng, không chỉ cổng

- [ ] `docker-compose.yml` có mạng kiểu `edge` và `persist`.
- [ ] Dịch vụ kho dữ liệu: **không** `ports: ["5432:5432"]` (hoặc Redis `6379`) trên `0.0.0.0`. Chỉ API vào mạng persist.
- [ ] Cạnh publish `127.0.0.1:8080:80` nếu Compose cho phép, thay vì `8080:80` trần.

### 2. Sức khỏe và sự cố

- [ ] `healthcheck` Compose trên kho dữ liệu *và* API.
- [ ] `docker compose ps` healthy trước khi gọi API.
- [ ] **Tắt kho dữ liệu**, xác nhận `/healthz` là 503, lưu log API. Rồi bật lại.

### 3. Quan sát (tối thiểu)

- [ ] API log một dòng JSON mỗi request: `method`, `path`, `status`, `duration_ms`, `request_id`.
- [ ] Không log thân request có thể chứa dữ liệu cá nhân.

### 4. Biếm họa đàn hồi

- [ ] `docker compose up --scale api=2` chạy *hoặc* bạn viết vì sao cạnh chưa cân bằng hai API.
- [ ] Nếu scale-out thất bại (trạng thái trong bộ nhớ), ghi thành phát hiện.

### 5. Xong khi

- [ ] Trình duyệt trên **máy bạn** vào `http://127.0.0.1:8080` và tạo được bài.
- [ ] `NOTES.md` có sơ đồ ba tầng và hai mạng.
- [ ] `docker compose down -v` được ghi chú. Bạn biết volume sẽ mất.

## Kéo giãn (vẫn local)

- TLS trên cạnh bằng `mkcert` cho `courseboard.localhost`.
- `Makefile` với `make up`, `make test`, `make down`.
- Smoke test bài 13-04 trên `/healthz`.

## Không làm

- Không mở kho dữ liệu ra Wi-Fi nhà trường.
- Không đẩy ảnh chứa `.env`.
- Không lấy `network_mode: host` từ blog ngẫu nhiên làm lối tắt.

## Ghi chú giảng viên

Minh chứng: `NOTES.md` + Compose + ảnh/log 503. Không dùng rubrics trường khác.
