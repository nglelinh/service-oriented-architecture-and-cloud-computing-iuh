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

Chữ **"Free Tier" (Gói miễn phí)** trên trang chủ các nhà cung cấp đám mây là một trong những nguồn hiểu lầm tốn tiền nhất đối với sinh viên mới học Cloud. Miễn phí trên đám mây **không** có nghĩa là "cài gì cũng không tốn tiền"—nó là một hệ thống hạn mức đo lường chính xác theo giờ, theo GB lưu trữ, theo lưu lượng băng thông ra ngoài và theo thời hạn 12 tháng hoặc tín dụng giới hạn.

Bài thực hành này trang bị cho bạn bộ kỹ năng phòng vệ tài chính tối thiểu trước khi tạo bất kỳ tài nguyên đám mây thật nào: phân biệt các loại free-tier, thiết lập cảnh báo ngân sách, khóa quyền tài khoản gốc (root MFA), và xây dựng thói quen dọn dẹp tài nguyên sạch sẽ sau mỗi buổi học.

## Bản đồ kiến thức nền: Bốn nguyên tắc an toàn khi dùng Cloud Free Tier

1. **Ba loại hình Free Tier hoàn toàn khác nhau**:
   - **Always Free (Miễn phí vĩnh viễn theo hạn mức tháng)**: áp dụng lâu dài nhưng rất nhỏ (ví dụ 1 VM siêu nhỏ, hạn mức serverless hàng tháng).
   - **12-Month Free (Miễn phí 12 tháng đầu)**: tự động chuyển sang tính tiền đầy đủ ngay ngày thứ 366 nếu bạn quên tắt tài nguyên!
   - **Trial Credit (Tín dụng dùng thử, ví dụ `$100–$300` trong 90 ngày)**: hết tiền hoặc hết hạn là khóa dịch vụ hoặc trừ thẻ nếu đã nâng cấp tài khoản trả phí.

2. **Hạn mức tính theo *tổng giờ cộng dồn*, không phải "số máy"**:
   Hạn mức `750 giờ/tháng` của một dòng máy ảo nhỏ nghĩa là tổng thời gian chạy của *tất cả* các máy ảo thuộc dòng đó cộng lại. Chạy 1 máy suốt 31 ngày = `744 giờ` (vừa khít miễn phí); chạy **2 máy song song** sẽ đốt hết `750 giờ` chỉ trong hơn 15 ngày!

3. **Bảo vệ tài khoản trước kẻ tấn công đào tiền ảo (Account Hijacking)**:
   Tài khoản đám mây không bật **MFA (xác thực đa yếu tố)** trên tài khoản Root hoặc bị lộ Access Key trên GitHub công khai thường bị bot tự động quét thấy chỉ sau **vài phút** và bật hàng chục máy GPU đào coin gây thiệt hại hàng nghìn USD.

4. **Vệ sinh hạ tầng (Teardown Discipline)**:
   Mọi tài nguyên thử nghiệm đều phải có ngày hết hạn, được tạo bằng IaC hoặc danh sách kiểm kê rõ ràng, và kết thúc buổi thực hành bằng bước kiểm tra bảng tài nguyên ở *tất cả* các Region đã dùng.

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

### Khung giải thích (nên đọc chậm): Tính tay 2 cái bẫy Free Tier phổ biến nhất khiến sinh viên bị trừ tiền oan

#### Bẫy 1: Hạn mức `750 giờ/tháng` khi bật nhầm 2 máy ảo song song

Giả sử gói 12-Month Free Tier cho phép **`750 giờ/tháng`** máy ảo `t2.micro` / `t3.micro` (giá ngoài hạn mức `$0,012/giờ`). Một sinh viên tạo 1 máy ở buổi học tuần 1, sang tuần 2 tạo thêm máy thứ 2 để thử nghiệm và quên tắt máy thứ 1 trong cả tháng 30 ngày (`720 giờ`):
- **Tổng số giờ sử dụng thực tế**: $2\text{ máy} \times 720\text{ giờ} = \mathbf{1.440\text{ giờ}}$.
- **Số giờ vượt hạn mức Free Tier**: $1.440 - 750 = \mathbf{690\text{ giờ bị tính phí}}$!
- Nếu máy đó lại gắn thêm **IPv4 công khai** (`$0,005/giờ/IP`) hoặc ổ đĩa vượt `30 GB`:
  $$\text{Phí vượt mức} = (690 \times \$0{,}012) + (1.440 \times \$0{,}005) = \$8{,}28 + \$7{,}20 = \mathbf{\$15{,}48}$$
  dù sinh viên tin chắc mình "chỉ dùng dòng máy Free Tier"!

#### Bẫy 2: Quên xóa tài nguyên phụ thuộc sau khi xóa máy ảo

| Hành động dọn dẹp | Tài nguyên chính đã xóa? | Tài nguyên "mồ côi" vẫn âm thầm tính tiền | Cách phòng tránh |
| :--- | :---: | :--- | :--- |
| Chỉ bấm **Terminate VM** trên Console | Có | Ổ đĩa rời (EBS volume không tích *Delete on termination*), Elastic IP chưa gắn máy (`$3,60/tháng`), NAT Gateway (`$33/tháng`), Snapshot | Tạo mọi thứ bằng OpenTofu rồi chạy **`tofu destroy`**, và kiểm tra lại mục **Billing → Cost Explorer** sau 24 giờ |

Chỉ cần nhớ phép tính `2 máy × 720 giờ = 1.440 giờ > 750 giờ` và thói quen đặt **Budget Alert `$1`** ngay ngày đầu, bạn sẽ không bao giờ bị bất ngờ bởi hóa đơn đám mây.

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
