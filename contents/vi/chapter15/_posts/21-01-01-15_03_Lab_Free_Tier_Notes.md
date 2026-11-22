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

**Hình thức:** Đọc tài liệu kết hợp thực hiện checklist.  
**Thời lượng dự kiến:** 60–90 phút nếu thực hành trên giấy; lâu hơn nếu lớp được cấp tài khoản đám mây thật.  
**Mặc định:** Bạn vẫn có thể hoàn thành trọn vẹn Lộ trình triển khai mà **không** cần tạo tài khoản đám mây công cộng.

Chính sách free-tier của các nhà cung cấp thay đổi thường xuyên, vì vậy mọi danh mục dịch vụ cần được **kiểm tra trực tiếp trên trang chính thức tại thời điểm thực hành**. Bài này tập trung vào nguyên tắc vận hành an toàn tại IUH, không phải hướng dẫn săn mã ưu đãi và không sao chép từ CS 40.

## Mục đích

Hiểu rõ phạm vi thực sự của các gói "miễn phí", nhận diện những khoản mục dễ phát sinh chi phí ngầm, và nắm đúng **trình tự thao tác an toàn** trước khi triển khai ra ngoài `localhost`.

## Phân loại các hình thức free-tier phổ biến

| Loại hình | Phạm vi điển hình | Rủi ro thường gặp ở sinh viên |
| --- | --- | --- |
| Hạn mức miễn phí dài hạn (always-free) | Giới hạn sử dụng nhỏ mỗi tháng | Vượt hạn mức âm thầm mà không để ý |
| Tín dụng dùng thử có thời hạn | Số tiền hoặc giờ sử dụng có ngày hết hạn | Tín dụng hết hạn trong khi tài nguyên vẫn đang chạy |
| Ưu đãi 12 tháng cho tài khoản mới | Một số SKU miễn phí trong năm đầu | Các mục phụ trợ đi kèm (IPv4 công khai, NAT, snapshot) vẫn tính phí |
| Gói sinh viên / giáo dục | Cấp thêm hạn mức tín dụng học tập | Bị ràng buộc bởi quy định tổ chức dùng chung, không thể dùng tùy ý như tài khoản cá nhân |

**Những hạng mục thường phát sinh chi phí ngay cả trên tài khoản mới:** NAT Gateway để quên, bộ cân bằng tải chạy không tải, địa chỉ IPv4 công khai gắn trên nhiều giao diện mạng, triển khai đa vùng sẵn sàng (multi-AZ) không cần thiết, máy ảo GPU và lưu lượng egress lớn khi chia sẻ cho cả lớp.

Hãy đọc trước bài [12-03 FinOps và đọc hóa đơn đám mây]({{ site.baseurl }}{% multilang_post_url contents/chapter12/21-01-01-12_03_FinOps_Billing_Literacy %}) trước khi thao tác trên bảng điều khiển web.

## Trình tự thao tác an toàn tại IUH

Ghi lại vào `NOTES.md` và đánh dấu kiểm tra nếu bạn có sử dụng đám mây:

1. [ ] Giảng viên xác nhận rõ **loại tài khoản** được phép sử dụng.
2. [ ] Truy cập được trang **Billing** và xác nhận đơn vị tiền tệ hiển thị.
3. [ ] Thiết lập **ngân sách (budget) và cảnh báo** ở mức trần thấp, gửi email đồng thời tới **cả hai** thành viên.
4. [ ] Chọn cố định một vùng địa lý (region) mặc định; không tạo tài nguyên rải rác ở vùng khác.
5. [ ] Bật xác thực đa yếu tố (MFA) cho tài khoản đăng nhập.
6. [ ] Không chia sẻ khóa truy cập dài ngày qua nhóm chat; ưu tiên thao tác an toàn và chuyển sang OIDC khi tích hợp CI (bài 13-04).
7. [ ] Gắn đầy đủ thẻ `project`, `pair`, `env`, `owner` cho mọi tài nguyên.
8. [ ] Chuẩn bị sẵn kịch bản `destroy` *trước* khi khởi tạo tài nguyên (xem Lab B).
9. [ ] Cuối buổi thực hành, rà soát toàn bộ vùng để chắc chắn không còn tài nguyên tồn đọng (máy ảo, ổ đĩa, IP công khai, snapshot, log group, container registry).
10. [ ] Lưu ảnh chụp màn hình **chi phí bằng 0 hoặc gần 0** kèm ngày tháng rõ ràng.

Nếu không thể hoàn thành Bước 3, **dừng lại ngay** và quay về thực hành cục bộ với Lab A.

## Thử nghiệm điểm biên công khai (tùy chọn, chỉ sau khi hoàn tất 10 bước trên)

Ở phạm vi môn học, nhóm chỉ cần chọn **một** trong hai phương án tối giản sau:

- Một máy ảo cỡ rất nhỏ hoặc dịch vụ quản lý kiểu Cloud Run / Container Apps có sẵn TLS do nhà cung cấp quản lý, **hoặc**
- Gắn DNS + TLS trên một tên miền miễn phí sẵn có, trỏ qua đường hầm (tunnel) cục bộ được giảng viên cho phép.

Không cần tự dựng VPC ba tầng hoàn chỉnh trên tài khoản cá nhân; sơ đồ mạng và tệp `plan` ở Lab B đã đủ để đánh giá năng lực thiết kế mạng.

- [ ] Địa chỉ truy cập công khai sử dụng **HTTPS**.
- [ ] Dịch vụ nền có khả năng `scale-to-zero` hoặc máy ảo sẽ được **tắt và xóa hẳn** ngay trong ngày.
- [ ] Quy tắc tường lửa chỉ mở cổng `443` từ Internet, **tuyệt đối không** mở cổng `22` từ `0.0.0.0/0`.
- [ ] Kịch bản `destroy` đã được ghi nhận và thực thi đầy đủ.

## Quy tắc đạo đức sử dụng tài nguyên lớp học

- Không lưu trữ nội dung vi phạm, không chạy công cụ quét mạng trái phép và không khai thác tiền mã hóa.
- Không đưa dữ liệu học vụ thật của sinh viên lên dự án free-tier cá nhân.
- Không chia sẻ mật khẩu tài khoản root cho nhiều người.
- Nếu vô tình khởi tạo tài nguyên phát sinh chi phí ngoài dự kiến, cần báo ngay cho giảng viên trong ngày để xử lý kịp thời.

## Tiêu chí hoàn thành

Bạn có thể giải thích mạch lạc (không cần nhìn slide) sự khác biệt giữa "always free", "tín dụng dùng thử" và tình huống "quên tắt NAT Gateway". Ngay cả khi không mở tài khoản đám mây, tệp `NOTES.md` vẫn cần liệt kê đủ 10 bước kiểm tra và nêu rõ **bước nào khiến nhóm quyết định dừng lại ở máy cục bộ** — đó vẫn là một bài Lab C hợp lệ.

## Video tham khảo

- **Xem trước:** [Introducing the new AWS Free Tier Program \| Amazon Web Services](https://www.youtube.com/watch?v=pisDsIgXGUI) — Amazon Web Services (tiếng Anh). Thông báo chính thức của AWS về chương trình free-tier hiện hành; luôn đối chiếu lại trên trang tài liệu mới nhất.
- [AWS Free Tier and the New AWS Experience](https://www.youtube.com/watch?v=7aOb0vZHT5k) — AWS Developers (tiếng Anh). Giải thích chi tiết hơn về hạn mức free-tier và cách theo dõi trên giao diện AWS.
- [AWS Cost Optimisation Series: Cost Explorer \| Amazon Web Services](https://www.youtube.com/watch?v=oE8TNKGmc40) — Amazon Web Services (tiếng Anh). Hướng dẫn dùng Cost Explorer để phát hiện sớm các mục tính phí ẩn trên tài khoản miễn phí.

## Tài liệu tham khảo

1. Trang chính sách miễn phí chính thức của AWS, Azure và GCP (kiểm tra tại ngày làm bài lab).
2. GitHub Student Developer Pack (nếu đủ điều kiện) — lưu ý vẫn bắt buộc thiết lập cảnh báo ngân sách.
3. [infracourse.cloud](https://infracourse.cloud/) — nguồn tham khảo tư duy coi trọng DNS/TLS và kiểm soát chi phí; quy trình trên được biên soạn độc lập cho IUH.
