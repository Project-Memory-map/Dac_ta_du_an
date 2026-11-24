[← Mục lục SRS](README.md)

# 11. Quyết định, điểm lệch so với bản 0.3 và vấn đề còn mở

## 11.1 Quyết định đã chốt

1. Phạm vi GĐ1 không gồm tương tác cộng đồng (thích, bình luận, "Muốn đi", theo dõi, báo cáo); FR-4.1 đến FR-4.5, FR-4.7 thuộc GĐ2.
2. Gợi ý hashtag bằng AI (FR-3.6) đã làm trong GĐ1, dùng Google Gemini, có phương án dự phòng theo từ khóa khi không có khóa hoặc Gemini lỗi.
3. "Khu vực" là cấp tỉnh/thành, danh mục cố định 34 tỉnh/thành sau sắp xếp năm 2025. Ánh xạ ký ức vào tỉnh/thành theo tọa độ và ranh giới GeoJSON, không theo tên Nominatim trả về (BR-25).
4. Giới hạn: ảnh tối đa 10/ký ức, ≤ 5 MB, .jpg/.jpeg/.png/.webp/.gif; video tối đa 1/ký ức, ≤ 50 MB, .mp4/.mov; hashtag tối đa 10/ký ức, ≤ 30 ký tự, nhận chữ có dấu (BR-06).
5. Chỉ hai mức riêng tư: riêng tư và công khai.
6. Một ký ức thuộc được nhiều chuyến đi. Chuyến đi không có chế độ chia sẻ công khai ở GĐ1.
7. Đăng ký bằng email chưa bắt buộc xác thực email. Tài khoản chỉ đăng nhập mạng xã hội có tùy chọn "Tạo mật khẩu" trong hồ sơ.
8. Tài khoản bị khóa: mọi ký ức công khai của người đó ẩn khỏi cộng đồng tới khi mở khóa; trạng thái từng ký ức không đổi (BR-29).
9. Tìm địa điểm: Nominatim qua máy chủ, có cache và giới hạn 1 yêu cầu/giây toàn hệ thống (BR-19). Email: Gmail SMTP (App Password) cho môi trường thật, Mailtrap khi kiểm thử.
10. Lưu ảnh/video: thư mục cục bộ `uploads/` cho bản demo; triển khai thật dự kiến Cloudflare R2.
11. Chỉ tiêu hiệu năng cho đồ án: 100 người dùng đồng thời, API < 1 giây, trang chính ≤ 3 giây (chưa đo, mục 11.3).
12. Quy tắc nghiệp vụ quan trọng được chặn hai lớp: ở ứng dụng và bằng trigger MySQL.

## 11.2 Điểm lệch giữa bản 0.3 và bản cài đặt (đã chấp nhận ở bản 1.0)

| # | Mục | Bản 0.3 ghi | Bản cài đặt | Lý do / ghi chú |
|---|---|---|---|---|
| 1 | Định dạng ảnh (BR-06, US-MEM-003) | .jpg, .jpeg, .png, .heic; .heic tự chuyển sang .jpg bằng pillow-heif | .jpg, .jpeg, .png, .webp, .gif; không nhận .heic | Đa số trình duyệt không hiển thị HEIC; ảnh đại diện thống nhất cùng danh sách (lỗi L18). Hỗ trợ HEIC đưa vào mục 10. |
| 2 | Xóa EXIF (BR-22) | Dùng Pillow | Bộ lọc Python thuần theo cấu trúc tệp, không nén lại ảnh | Không thêm thư viện, giữ nguyên chất lượng ảnh (lỗi L13). |
| 3 | Gán khu vực (US-EXP-003) | Thư viện shapely | Thuật toán ray casting tự viết + dung sai ~5 km cho ghim ven biển | Không thêm thư viện; xử lý ghim sát bờ biển lệch ra ngoài đường biên đã rút gọn (lỗi L05). |
| 4 | Thời hạn JWT (US-ACC-008) | Ví dụ 60 phút | Mặc định 1440 phút (24 giờ), cấu hình bằng `ACCESS_TOKEN_MINUTES` | Chưa có refresh token nên chọn hạn dài để người dùng không phải đăng nhập lại liên tục. Rủi ro ở 11.3 #4. |
| 5 | Hạn URL media (NFR-S2) | Vài phút | 3600 giây (1 giờ) | Video dài và trang mở lâu không bị đứt link; quyền xem vẫn được kiểm lại mỗi lần tải. |
| 6 | Liên kết Google khi email trùng (BR-08) | Nếu tài khoản cũ chưa xác thực email thì xóa `password_hash` cũ và thu hồi mọi phiên rồi liên kết | Chỉ liên kết khi Google báo email đã xác thực; không xóa mật khẩu cũ. Nếu Google báo chưa xác thực thì từ chối | Đơn giản hơn và vẫn chặn chiếm tài khoản qua email Google chưa xác thực. |
| 7 | API tải media (Hình 7) | `POST /memories/{id}/media` | Tách `POST /api/memories/{id}/images` và `POST /api/memories/{id}/video` | Ảnh và video có quy tắc khác nhau (số lượng, thêm/bỏ/sắp xếp, phát theo Range). |
| 8 | API ghim công khai (Hình 9) | `GET /explore/memories` | `GET /api/explore/pins` cho ghim (≤ 500, dữ liệu gọn); `GET /api/explore/memories` cho danh sách có phân trang | Ghim cần ít trường và nhiều dòng; danh sách cần đủ trường và phân trang. |
| 9 | Phân trang (NFR-P4) | 20 mục/trang | Khám phá 12, Của tôi 20, Quản trị 20 | Thẻ ký ức công khai có ảnh, chiếm chỗ hơn. |
| 10 | Kiến trúc trong module (Hình 2, 3) | Router → Service → Repository; `core/deps.py`, `core/storage.py` | Phần lớn logic nằm trong router; `get_current_user` ở `accounts/deps.py`; dịch vụ tệp ở `memories/services/`. `core/deps.py`, `core/storage.py`, `explore/service.py`, `explore/repository.py`, `explore/schemas.py` để trống | Đủ cho quy mô đồ án; khung file giữ lại để tách sau. |
| 11 | FR-3.6 | Tùy chọn, chỉ làm nếu còn thời gian | Đã làm (T05B590) | — |
| 12 | Chọn vị trí trên bản đồ (US-MEM-002) | Nhấp đúp / nhấn giữ | Thêm nhấp chuột phải và nút "🎯 Chọn trên bản đồ"; ghim tím kéo được | Dễ dùng hơn trên máy tính bàn di chuột. |
| 13 | Ghi chú thời hạn dọn tệp (US-MEM-007) | Cron/APScheduler 1 lần/tháng | Chưa làm | Xem 11.3 #2. |

## 11.3 Vấn đề còn mở

| # | Mức | Vấn đề | Ảnh hưởng | Hướng xử lý đề nghị |
|---|---|---|---|---|
| 1 | **Cao** | **Form ký ức không lưu hashtag và chuyến đi đã chọn.** `memories.js` gửi `hashtags`, `ai_hashtags`, `trip_ids` kèm `POST /api/memories` và `PATCH /api/memories/{id}`, nhưng hai API này không khai báo các trường đó nên bỏ qua; giao diện cũng không gọi `POST /api/memories/{id}/hashtags` hay `POST /api/trips/{id}/memories` sau khi lưu. Phát hiện khi soạn bản 1.0 bằng cách đối chiếu code; kiểm thử tích hợp không bắt được vì tạo hashtag bằng API trực tiếp. | Người dùng không gắn được hashtag nào qua giao diện (US-MEM-010, FR-2.9); gợi ý AI chọn xong cũng mất; lọc hashtag (US-EXP-002) chỉ có dữ liệu từ seed. Ô chọn chuyến đi trong form vô tác dụng (thêm qua panel Chuyến đi vẫn được). | Cách 1 (ít sửa nhất): trong `memories.js`, sau khi lưu ký ức gọi `PUT /api/memories/{id}/hashtags` với `{hashtags, ai_hashtags}`, và gọi API thêm/bỏ ký ức của từng chuyến đi đã thay đổi. Cách 2: cho `create_memory` / `update_memory` nhận 3 trường này và gọi `sync_memory_hashtags`, `_attach_memories_to_trip` trong cùng giao dịch. Thêm một ca kiểm thử giao diện tạo ký ức có hashtag qua form. |
| 2 | Trung bình | Chưa có script dọn tệp của ký ức đã xóa mềm (BR-13). | Tệp không truy cập được nhưng vẫn chiếm ổ đĩa; ảnh/video bị bỏ khi sửa ký ức thì đã xóa ngay. | Viết `scripts/don_tep.py` xóa tệp media của ký ức có `deleted_at` quá 30 ngày, chạy bằng lịch của hệ điều hành ngày 1 hàng tháng. |
| 3 | Trung bình | Facebook liên kết vào tài khoản có cùng email mà không kiểm email đã xác thực, khác BR-18 bản 0.3. | Nếu Facebook trả một email chưa được chủ thật xác nhận, người khác có thể vào tài khoản email/mật khẩu trùng email đó. | Theo BR-18: không tự liên kết theo email với Facebook; tạo tài khoản mới, hoặc yêu cầu đăng nhập bằng mật khẩu trước rồi mới cho liên kết. |
| 4 | Thấp–TB | JWT còn dùng được tới 24 giờ sau khi đăng xuất (lỗi L11, ghi nhận theo thiết kế BR-20). | Token bị lộ trước khi đăng xuất vẫn gọi được API. | Rút `ACCESS_TOKEN_MINUTES` (ví dụ 60) kèm refresh token, hoặc tăng `token_version` khi đăng xuất (đăng xuất mọi thiết bị). |
| 5 | Thấp | Chưa có ảnh thu nhỏ (NFR-P4). | Danh sách tải ảnh gốc tới 5 MB mỗi ảnh, chậm trên điện thoại. | Tạo bản thu nhỏ (ví dụ cạnh dài 400 px) khi tải lên, lưu vào `media.thumb_path`, trả qua `thumb_url`. |
| 6 | Thấp | Nhãn đếm ảnh trong `map.html` còn ghi "0/10 · .jpg .png .heic ≤ 5 MB", trong khi hệ thống không nhận .heic (ô chọn tệp và máy chủ đã đúng). | Người dùng iPhone có thể hiểu nhầm. | Sửa nhãn thành ".jpg .png .webp .gif ≤ 5 MB". |
| 7 | Thấp | Quản trị viên khóa được quản trị viên khác. | Một quản trị viên có thể khóa toàn bộ quản trị viên còn lại. | Nhóm quyết định: chặn khóa tài khoản `role = 'admin'`, hoặc chấp nhận và ghi thành quy tắc. |
| 8 | Thấp | Quản trị viên xóa ký ức không gỡ dòng `trip_memories` như khi chủ xóa (BR-24). | Không hiện ra ngoài (truy vấn chuyến đi bỏ qua ký ức đã xóa) nhưng dữ liệu không nhất quán. | Gỡ `trip_memories` trong `admin_delete_memory`, hoặc ghi rõ đây là chủ ý để giữ dấu vết. |
| 9 | Môi trường | Chưa chạy luồng Google/Facebook thật vì chưa có khóa OAuth (lỗi L12, 5 AC "cần cấu hình"). | Chưa nghiệm thu được US-ACC-003 AC1/2/4, US-ACC-004 AC1/2 trên dịch vụ thật. | Điền khóa theo `.env.example`, đăng nhập thử bằng một tài khoản Google và một tài khoản Facebook. |
| 10 | Chưa đo | Chưa kiểm thử tải (NFR-P1, NFR-P2), chưa kiểm trên Chrome, Firefox, Safari và điện thoại thật (NFR-U1). | Chưa có số liệu chứng minh chỉ tiêu hiệu năng và tương thích. | Chạy công cụ kiểm thử tải (ví dụ Locust/k6) với 100 người dùng; thử nhanh trên 1 Android và 1 iPhone thật trước khi demo. |
| 11 | Thấp | Cột `users.email_verified_at` có sẵn nhưng chưa ghi; đăng ký email không xác thực email. | Không phân biệt được email đã xác thực của tài khoản đăng ký thường. | Làm cùng chức năng gửi email xác thực (mục 10.1). |
| 12 | Vận hành | Chưa có sao lưu CSDL tự động (NFR-R1), chưa cấu hình HTTPS (NFR-S4). | Rủi ro mất dữ liệu và lộ phiên khi đưa lên Internet. | Thiết lập khi triển khai thật. |
