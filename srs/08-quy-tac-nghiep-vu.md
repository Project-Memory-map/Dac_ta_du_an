[← Mục lục SRS](README.md)

# 8. Quy tắc nghiệp vụ

Cột **Thực thi**: *App* = kiểm ở máy chủ FastAPI; *UI* = kiểm thêm ở giao diện cho phản hồi nhanh; *DB* = trigger, ràng buộc hoặc collation của MySQL. Quy tắc từ BR-23 trở đi là mới ở bản 1.0, ghi lại hành vi đã có trong sản phẩm.

| **Mã** | **Quy tắc** | **Nguồn** | **Thực thi** | **Trạng thái** |
|---|---|---|---|---|
| BR-01 | Ký ức mới tạo mặc định ở chế độ riêng tư. | FR-2.6 | App, UI, DB (DEFAULT) | ✅ |
| BR-02 | Chỉ chủ sở hữu được sửa, xóa, đổi chế độ, gắn hashtag, đưa ký ức vào chuyến đi. Quản trị viên chỉ ẩn/bỏ ẩn/xóa theo FR-4.6. Không được đổi chủ sở hữu của ký ức. | FR-2.5, FR-4.6 | App; DB (`trg_memories_bu` chặn đổi `user_id`) | ✅ |
| BR-03 | Ký ức hiện với người không phải chủ khi và chỉ khi: công khai, không bị ẩn, chưa xóa, **và** chủ ký ức đang hoạt động (không bị khóa). Quy tắc áp cho bản đồ, danh sách, tìm kiếm, chi tiết, ảnh, video, hashtag, thống kê công khai. | FR-2.1 | App (`public_query`, kiểm ở từng API); DB (view `v_public_memories`) | ✅ |
| BR-04 | Ký ức riêng tư chỉ chủ xem được. Người khác mở chi tiết nhận 404 "Không tìm thấy ký ức" (không tiết lộ ký ức có tồn tại). Người khác mở URL ảnh/video của ký ức riêng tư nhận 403 "Bạn không có quyền truy cập tệp này". | NFR-S2 | App | ✅ |
| BR-05 | Ngày của ký ức bắt buộc và không được ở tương lai (so với ngày của máy chủ). | FR-2.3 | UI, App; DB (`trg_memories_bi`, `trg_memories_bu`) | ✅ |
| BR-06 | Mỗi ký ức tối đa **10 ảnh** (.jpg, .jpeg, .png, .webp, .gif; ≤ 5 MB mỗi ảnh), **1 video** (.mp4, .mov; ≤ 50 MB), **10 hashtag**. Hashtag: bỏ "#" đầu, chữ thường, chỉ gồm chữ cái (kể cả có dấu tiếng Việt), chữ số, "_"; không khoảng trắng; 1–30 ký tự. | FR-2.4, FR-2.9 | UI, App; DB (`trg_media_bi`, `trg_hashtags_bi`, `trg_memory_hashtags_bi`) | ✅ |
| BR-07 | Mật khẩu ≥ 8 ký tự, có ít nhất một chữ cái và một chữ số, ≤ 72 byte UTF-8; chỉ lưu bản băm bcrypt. | FR-1.1 | App | ✅ |
| BR-08 | Email là duy nhất, lưu chữ thường, so khớp không phân biệt hoa thường. Đăng nhập mạng xã hội có email trùng tài khoản đã có thì liên kết vào tài khoản đó theo BR-18, không tạo tài khoản thứ hai. | FR-1.9 | App; DB (UNIQUE, `trg_users_bi`/`trg_users_bu` chuẩn hóa email) | ✅ |
| BR-09 | Chuyến đi chỉ chứa ký ức của chính chủ chuyến đi. Ký ức trong chuyến đi xếp theo ngày ký ức tăng dần; trùng ngày thì theo thời điểm tạo, rồi theo id. Một ký ức thuộc được nhiều chuyến đi. | FR-2.7 | App; DB (`trg_trip_memories_bi`) | ✅ |
| BR-10 | Thao tác ẩn, xóa ký ức và khóa tài khoản của quản trị viên phải có lý do (≤ 500 ký tự). Mọi thao tác quản trị (kể cả bỏ ẩn, mở khóa) được ghi nhật ký: người thao tác, thời điểm, loại thao tác, đối tượng, lý do. | FR-4.9 | App; DB (`trg_admin_actions_bi`) | ✅ |
| BR-11 | Quản trị viên không được khóa chính tài khoản của mình. | FR-4.6 | App; DB (`trg_admin_actions_bi`) | ✅ |
| BR-12 | Tác giả của ký ức công khai hiển thị bằng tên hiển thị và ảnh đại diện; không bao giờ trả email của tác giả. | FR-2.8 | App | ✅ |
| BR-13 | Xóa ký ức là xóa mềm: biến mất ngay khỏi mọi màn hình, tệp đính kèm không truy cập được nữa. Tệp vật lý được dọn định kỳ mỗi tháng một lần, xóa tệp của ký ức đã xóa mềm quá 30 ngày. | FR-2.5 | App | ⚠️ Xóa mềm đạt; **script dọn tệp định kỳ chưa làm** (mục 11.3) |
| BR-14 | Liên kết đặt lại mật khẩu có hiệu lực 30 phút và chỉ dùng một lần; đặt lại thành công thì mọi liên kết chưa dùng khác của tài khoản cũng mất hiệu lực. | FR-1.5 | App | ✅ |
| BR-15 | Nhập sai mật khẩu 5 lần trong vòng 15 phút (tính từ lần sai đầu) thì tạm khóa đăng nhập bằng mật khẩu 15 phút; đăng nhập đúng hoặc đặt lại mật khẩu thì xóa bộ đếm. | FR-1.2 | App | ✅ |
| BR-16 | Lọc theo nhiều hashtag: hiện ký ức có ít nhất một hashtag đã chọn (OR). Kết hợp với lọc khu vực, từ khóa, vùng bản đồ: thỏa tất cả (AND). | FR-3.1 | App | ✅ |
| BR-17 | Ảnh/video chỉ truy cập qua URL có chữ ký HMAC-SHA256 kèm hạn dùng 1 giờ (`/api/media/{id}?exp=&sig=` cho ảnh, `/api/media/{id}/stream?exp=&sig=` cho video, chữ ký video gồm cả đường dẫn tệp). Mỗi lần tải, máy chủ vẫn kiểm lại quyền xem theo BR-03, BR-04 và trạng thái xóa. Thư mục `uploads/` không được phục vụ tĩnh. | NFR-S2 | App | ✅ |
| BR-18 | Liên kết tài khoản mạng xã hội theo email trùng (BR-08): **Google** chỉ liên kết khi Google xác nhận email đã xác thực (`email_verified`); nếu không, từ chối với "Email Google chưa được xác thực, hãy đăng nhập bằng mật khẩu" và không tạo tài khoản mới. **Facebook**: theo bản 0.3, không tự liên kết theo email. | FR-1.9 | App | ✅ Google · ⚠️ Facebook hiện vẫn liên kết theo email (mục 11.3) |
| BR-19 | Gọi Nominatim: chỉ khi người dùng nhấn Enter hoặc nút "Tìm" (không gọi khi đang gõ); luôn qua máy chủ (`/api/geocode`, `/api/geocode/reverse`); tối đa 1 yêu cầu/giây cho **toàn hệ thống**, yêu cầu sau chờ tới lượt tối đa 5 giây rồi mới báo bận; header User-Agent ghi tên ứng dụng; cache kết quả tìm tên 24 giờ (≤ 1000 mục), cache reverse theo tọa độ làm tròn 4 chữ số (≤ 512 mục). | NFR-P5 | UI, App | ✅ |
| BR-20 | `users.token_version` tăng 1 khi đổi mật khẩu, tạo mật khẩu, đặt lại mật khẩu, hoặc bị quản trị viên khóa. JWT mang `token_version` lúc cấp; mỗi yêu cầu so với giá trị trong CSDL, khác thì JWT hết hiệu lực ngay. Đăng xuất không tăng `token_version`. | NFR-S3 | App; DB (`trg_users_bu`) | ✅ |
| BR-21 | Dữ liệu người dùng nhập (tiêu đề, nội dung, tên hiển thị, hashtag, tên địa điểm, lý do...) chỉ được đưa vào giao diện bằng `textContent` / `UI.el()`, kể cả trong popup Leaflet; không dùng `innerHTML` hoặc `bindPopup` với chuỗi HTML dựng từ dữ liệu người dùng. | NFR-S6 | UI | ✅ |
| BR-22 | Ảnh tải lên (ảnh ký ức, ảnh đại diện) được xóa siêu dữ liệu EXIF/GPS/XMP trước khi lưu, bằng bộ lọc Python thuần không nén lại ảnh (JPEG, PNG, WebP; GIF giữ nguyên vì không mang EXIF). | NFR-S12 | App | ✅ |
| BR-23 | Quên mật khẩu: mỗi tài khoản nhận tối đa 1 liên kết mỗi 60 giây; tài khoản bị khóa không được cấp liên kết; mọi trường hợp đều trả cùng một thông báo. | FR-1.5 | App | ✅ |
| BR-24 | Chủ xóa ký ức thì ký ức được gỡ khỏi mọi chuyến đi; xóa chuyến đi hoặc bỏ ký ức khỏi chuyến đi không xóa ký ức. | FR-2.5, FR-2.7 | App; DB (khóa ngoại CASCADE) | ✅ |
| BR-25 | Khu vực của ký ức xác định theo tọa độ đối chiếu ranh giới 34 tỉnh/thành, không theo tên địa điểm. Điểm nằm ngoài mọi tỉnh nhưng cách đường biên gần nhất không quá 0,05° (khoảng 5 km) thì gán tỉnh đó; xa hơn (ví dụ nước ngoài) thì không gắn khu vực. Đổi tọa độ thì tính lại. | FR-2.13, FR-3.1 | App | ✅ |
| BR-26 | Nhật ký quản trị không sửa, không xóa được. | FR-4.9 | DB (`trg_admin_actions_bu`, `trg_admin_actions_bd`; khóa ngoại RESTRICT với người thao tác) | ✅ |
| BR-27 | Quản trị viên chỉ xem và thao tác trên ký ức **công khai** chưa xóa; ký ức riêng tư không hiện ở trang quản trị. | FR-4.8 | App | ✅ |
| BR-28 | Hashtag so khớp không phân biệt hoa thường nhưng phân biệt dấu ("DaLat" = "dalat", nhưng "đàlạt" ≠ "dalat"). Mỗi liên kết ký ức–hashtag ghi nguồn `manual` hoặc `ai`. | FR-2.9, FR-3.6 | App; DB (collation `utf8mb4_0900_as_ci`) | ✅ |
| BR-29 | Khóa tài khoản không đổi trạng thái từng ký ức; ký ức công khai của người bị khóa tự ẩn khỏi cộng đồng nhờ BR-03, và hiện lại nguyên trạng khi mở khóa. | FR-4.6 | App | ✅ |
| BR-30 | Đổi mật khẩu: tài khoản đã có mật khẩu phải nhập đúng mật khẩu hiện tại và mật khẩu mới phải khác mật khẩu cũ; tài khoản chỉ đăng nhập mạng xã hội được tạo mật khẩu mà không cần mật khẩu hiện tại. | FR-1.6 | App | ✅ |
