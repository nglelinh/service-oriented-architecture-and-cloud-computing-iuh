---
layout: post
title: 15-03 Lab C — Ghi chú free-tier và vệ sinh đám mây IUH
chapter: '15'
order: 4
owner: Nguyen Le Linh
lang: vi
categories:
- chapter15
lesson_type: optional
---

**Hình thức:** đọc + checklist.  
**Khung giờ:** 60–90 phút nếu ở trên giấy; lâu hơn nếu giảng viên mở tài khoản thật.  
**Mặc định:** có thể xong Lộ trình triển khai **không** cần tài khoản đám mây công cộng.

Danh sách free-tier đổi. Mọi SKU dưới đây là **“kiểm trang nhà cung cấp hôm nay.”** Đây là vệ sinh IUH, không phải hướng dẫn săn coupon và không phải lab CS 40.

## Mục đích

Biết “miễn phí” thường nghĩa là gì, chưa từng nghĩa là gì, và **thứ tự thao tác** nếu rời localhost.

## Free-tier thường là gì

| Dáng | Nghĩa điển hình | Rủi ro sinh viên |
| --- | --- | --- |
| Hạn mức always-free | Trần nhỏ mỗi tháng | Vượt trần im lặng |
| Tín dụng dùng thử theo thời gian | Tiền hoặc giờ hết hạn | Hết tín dụng, tài nguyên vẫn chạy |
| Ưu đãi tài khoản mới 12 tháng | Một số SKU gần miễn phí một năm | Đồng hồ kèm theo (IPv4, NAT, snapshot) vẫn tính |
| Gói sinh viên / giáo dục | Tín dụng thêm | Luật org dùng chung; đừng coi như AWS cá nhân |

**Không bao giờ “miễn phí trong tinh thần”:** NAT để quên, cân bằng tải không instance, IPv4 công trên mọi NIC, hai AZ “vì production,” VM GPU, egress cho cả lớp.

Đọc [12-03 FinOps]({{ site.baseurl }}{% multilang_post_url contents/chapter12/21-01-01-12_03_FinOps_Billing_Literacy %}) trước khi bấm console.

## Thứ tự thao tác IUH

Ghi vào `NOTES.md` và đánh dấu nếu dùng đám mây:

1. [ ] Giảng viên xác nhận **loại tài khoản**.
2. [ ] Mở được **Billing** và thấy loại tiền.
3. [ ] **Budget + cảnh báo** trần thấp. Email tới **cả hai** thành viên.
4. [ ] Vùng mặc định đã chọn; không tạo “thêm một cái” ở vùng thứ hai.
5. [ ] MFA trên login thật sự dùng.
6. [ ] Không khóa dài ngày trên chat. Ưu tiên console + OIDC sau (13-04).
7. [ ] Thẻ `project`, `pair`, `env`, `owner`.
8. [ ] Kế hoạch destroy viết *trước* khi tạo (Lab B).
9. [ ] Sau buổi: quét vùng trống (máy, đĩa, IP, snapshot, log, registry).
10. [ ] Ảnh **chi phí = 0 hoặc gần 0** trong ngày, có ngày tháng.

Nếu bước 3 không làm được, **dừng**. Về Lab A.

## “Cạnh hello” tùy chọn (chỉ sau các dấu tick)

Đủ cho khóa này là **một** trong:

- Một VM rất nhỏ hoặc dịch vụ kiểu Cloud Run / Container Apps với TLS do nhà cung cấp quản lý, **hoặc**
- DNS + TLS trên hostname miễn phí bạn đã có, trỏ tới tunnel local mà giảng viên duyệt.

Không cần dựng VPC ba tầng trên tài khoản cá nhân. Sơ đồ + plan Lab B đủ cho năng lực mạng.

- [ ] URL công dùng **HTTPS**.
- [ ] Gốc là dịch vụ scale-to-zero hoặc VM bạn sẽ **tắt và hủy** trong ngày.
- [ ] Firewall: 443 từ internet, **không** 22 từ `0.0.0.0/0`.
- [ ] Lệnh/bấm destroy đã viết và đã chạy.

## Đạo đức lớp

- Không host nội dung vi phạm, máy quét, hay đào coin.
- Không lưu hồ sơ học vụ thật trên project free-tier cá nhân.
- Không chia sẻ mật khẩu root cho cả lớp.
- Nếu tạo nhầm đồng hồ tính tiền, báo giảng viên trong ngày.

## Xong khi

Bạn giải thích được, không cần slide, khác biệt “always free,” “tín dụng,” và “tôi để NAT bật.” Nếu không mở tài khoản, `NOTES.md` vẫn liệt kê mười tick và **tick nào chặn bạn** — đó là Lab C hợp lệ.

## Tài liệu

1. Trang AWS Free Tier, Azure free, GCP free (mở đúng ngày lab).
2. GitHub Student Developer Pack (nếu đủ điều kiện) — vẫn không cho phép bỏ budget.
3. [infracourse.cloud](https://infracourse.cloud/) — cảm hứng coi trọng DNS/TLS và chi phí. Quy trình trên là nguyên tác IUH.
