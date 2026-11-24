[← Mục lục SRS](README.md)

# 6. Yêu cầu giao diện ngoài

## 6.1 Giao diện người dùng

### 6.1.1 Nguyên tắc chung

- Toàn bộ giao diện, thông báo lỗi, email bằng tiếng Việt.
- Dùng được trên máy tính và điện thoại, từ độ rộng 360 px, không có trang nào tràn ngang (đã kiểm ở cỡ 1366×800 và Pixel 7 giả lập).
- Thông báo ngắn hiện ở góc màn hình (toast); thao tác nguy hiểm (xóa, công khai, khóa) qua hộp xác nhận; thao tác cần lý do qua hộp nhập.
- Dữ liệu người dùng nhập chỉ được đưa vào trang bằng `textContent` hoặc hàm dựng phần tử `UI.el()` (BR-21).
- Mỗi trang là một file HTML riêng; CSS và JS tách file theo module (NFR-M1).

### 6.1.2 Danh mục trang

| Trang | Địa chỉ | Ai dùng | Nội dung chính |
|---|---|---|---|
| Trang mở đầu | `/` | Mọi người | Giới thiệu, nút "Ghim ký ức đầu tiên" và "Xem bản đồ cộng đồng", 3 bước sử dụng, giới thiệu AI gợi ý hashtag, 3 ký ức công khai mới nhất, số liệu cộng đồng |
| Bản đồ | `/ban-do` | Mọi người | Bản đồ toàn màn hình, thanh tìm địa điểm, 3 tab thanh bên (🌏 Khám phá, 📍 Của tôi, 🧳 Chuyến đi), ngăn kéo chi tiết/form ký ức, nút định vị, bật/tắt lớp ghim, menu tài khoản |
| Đăng nhập | `/accounts/login.html` | Khách | Email, mật khẩu, nút Google/Facebook (làm mờ khi chưa cấu hình), liên kết quên mật khẩu, đăng ký; hiện lỗi từ tham số `?error=`; quay lại trang `?next=` sau khi đăng nhập |
| Đăng ký | `/accounts/register.html` | Khách | Tên hiển thị, email, mật khẩu, nút Google/Facebook |
| Quên mật khẩu | `/accounts/forgot-password.html` | Khách | Nhập email; ở môi trường dev hiện liên kết thử khi chưa có SMTP |
| Đặt lại mật khẩu | `/accounts/reset-password.html?token=` | Khách | Kiểm liên kết ngay khi mở; nhập mật khẩu mới |
| Hồ sơ | `/accounts/profile.html` | Người dùng | Ảnh đại diện, tên, email, giới thiệu, thống kê, đổi/tạo mật khẩu, ký ức theo năm, chuyến đi; nhắc bổ sung email khi thiếu |
| Hoàn tất đăng nhập mạng xã hội | `/accounts/oauth-callback.html` | Khách | Nhận JWT từ phần `#` của địa chỉ, lưu lại rồi chuyển tới bản đồ hoặc hồ sơ |
| Quản trị | `/admin/` | Quản trị viên | Thống kê; tab Ký ức công khai, Tài khoản, Nhật ký thao tác; tìm, lọc, phân trang, thao tác kèm lý do |

### 6.1.3 Bản đồ

| Thành phần | Mô tả |
|---|---|
| Nền | Ô bản đồ OpenStreetMap, ghi nguồn "© OpenStreetMap" |
| Ghim | Xanh `#2563eb`: ký ức của tôi công khai · Xanh đậm `#1e3a8a`: của tôi riêng tư · Xám `#9ca3af`: của tôi bị quản trị viên ẩn · Đỏ `#e11d48`: công khai của cộng đồng · Tím `#7c3aed`: vị trí đang chọn (nảy lên khi xuất hiện, kéo được) |
| Popup ghim | Tiêu đề, ngày, địa điểm, tác giả, nút mở chi tiết; dựng bằng DOM, không bằng chuỗi HTML |
| Tạo ghim | Nhấp đúp hoặc nhấp chuột phải (máy tính), nhấn giữ 600 ms (điện thoại), nút "🎯 Chọn trên bản đồ" hoặc "+ Ghim mới" |
| Lộ trình | Đường nét đứt tím nối ký ức của chuyến đi đang mở |
| Điều khiển | Thu phóng, về vị trí hiện tại (Geolocation API), bật/tắt "Ký ức của tôi" và "Ký ức cộng đồng" |

### 6.1.4 Form ký ức

Trường theo thứ tự: Vị trí (tìm tên hoặc chọn trên bản đồ; tên địa điểm sửa được) · Tiêu đề · Ngày · Câu chuyện · Hashtag (nút "✨ Gợi ý hashtag") · Ảnh (đếm x/10, xem trước, bỏ, đổi thứ tự) · Video (1 tệp) · Chế độ 🔒 Riêng tư / 🌏 Công khai · Chuyến đi (ô chọn) · nút "Lưu ký ức" / "Lưu thay đổi" kèm thanh tiến trình tải tệp.

## 6.2 Giao diện phần mềm: REST API

### 6.2.1 Quy ước

- Mọi API nằm dưới `/api`, trao đổi JSON (riêng tạo/sửa ký ức, ảnh, video, ảnh đại diện dùng `multipart/form-data`). Tài liệu tương tác tự sinh ở `/docs` (OpenAPI).
- Xác thực: header `Authorization: Bearer <JWT>`. API không bắt buộc đăng nhập vẫn đọc JWT nếu có (để biết người xem có phải chủ ký ức).
- Mọi phản hồi, kể cả lỗi, có cùng khung:

```json
{
  "statusCode": 200,
  "message": "Thành công",
  "error": null,
  "data": { },
  "path": "/api/...",
  "timestamp": "2026-11-21T10:00:00+00:00"
}
```

- Danh sách phân trang trả `data = {items, total, page, page_size, pages}`.
- Ảnh, video, ảnh đại diện không bao giờ trả đường dẫn lưu trữ; chỉ trả URL có chữ ký hoặc `/api/users/{id}/avatar?v=…`.
- Đường dẫn `/api/...` không tồn tại trả 404 "Chức năng này chưa được xây dựng".

### 6.2.2 Mã lỗi

| HTTP | `error` | Khi nào |
|---|---|---|
| 400 | `BAD_REQUEST` | Yêu cầu sai nghiệp vụ (tệp sai định dạng, sai mật khẩu hiện tại, liên kết đặt lại không hợp lệ...) |
| 401 | `UNAUTHORIZED` | Thiếu JWT, JWT sai, hết hạn hoặc đã bị thu hồi ("Bạn cần đăng nhập để tiếp tục"); sai email/mật khẩu |
| 403 | `FORBIDDEN` | Không có quyền (không phải chủ, không phải quản trị viên, tài khoản bị khóa) |
| 404 | `NOT_FOUND` | Không có dữ liệu, hoặc có nhưng người xem không được biết (ký ức riêng tư của người khác) |
| 409 | `CONFLICT` | Trùng dữ liệu (email đã đăng ký), trạng thái đã đúng như yêu cầu (ẩn ký ức đang ẩn) |
| 413 | `PAYLOAD_TOO_LARGE` | Dự phòng cho tệp quá lớn (hiện các kiểm tra dung lượng trả 400 kèm câu thông báo cụ thể) |
| 422 | `VALIDATION_ERROR` | Dữ liệu sai kiểu/định dạng; `data` là danh sách `{field, message}` |
| 422 | `BUSINESS_RULE` | Trigger MySQL chặn (`SIGNAL 45000`); `message` là câu của trigger, ví dụ "BR-06: Mỗi ký ức tối đa 10 ảnh" |
| 429 | `TOO_MANY_REQUESTS` | Đang tạm khóa đăng nhập do sai mật khẩu nhiều lần |
| 503 | `SERVICE_UNAVAILABLE` | Nominatim không phản hồi, quá tải; CSDL không kết nối được (`/api/health`) |
| 500 | `INTERNAL_ERROR` | Lỗi không lường trước: "Lỗi hệ thống, vui lòng thử lại sau" (chi tiết chỉ ghi log) |

### 6.2.3 Danh mục API

Bảng đầy đủ 65 khai báo API (tính cả các cặp PUT/PATCH cùng chức năng), chia theo module, ở [Phụ lục B](phu-luc-b-danh-muc-api.md).

## 6.3 Dịch vụ bên ngoài

| Dịch vụ | Dùng cho | Cách gọi | Khi lỗi hoặc chưa cấu hình |
|---|---|---|---|
| OpenStreetMap tile | Nền bản đồ | Trình duyệt tải ô bản đồ trực tiếp | Bản đồ trống nền, các chức năng khác vẫn chạy |
| Nominatim `/search`, `/reverse` | Tìm địa điểm, đổi tọa độ thành tên | Máy chủ gọi qua `/api/geocode`; 1 yêu cầu/giây toàn hệ thống, User-Agent riêng, `accept-language=vi`, cache, chờ tối đa 5 giây | Báo 503 kèm gợi ý chọn trực tiếp trên bản đồ / tự nhập tên |
| Google OAuth 2.0 | Đăng nhập Google | Authorization code ở máy chủ, scope `openid email profile` | Nút làm mờ; lỗi quay về trang đăng nhập |
| Facebook Graph API | Đăng nhập Facebook | Authorization code ở máy chủ, scope `email`, trường `id,name,email` | Nút làm mờ; lỗi quay về trang đăng nhập |
| Google Gemini | Gợi ý hashtag | Máy chủ gọi `generateContent` (mặc định `gemini-2.5-flash`, nhiệt độ 0,4, trả JSON), chờ tối đa 8 giây | Tự chuyển sang gợi ý theo từ khóa; trả `engine = "local"` |
| SMTP (Gmail / Mailtrap) | Email đặt lại mật khẩu | `smtplib`, STARTTLS hoặc SSL, chạy nền | Ghi log; ở dev trả liên kết thử qua API |

## 6.4 Giao diện phần cứng

Không có yêu cầu phần cứng riêng. Trên điện thoại cần màn hình cảm ứng (nhấn giữ để ghim). Nút "về vị trí hiện tại" dùng định vị của thiết bị khi người dùng cho phép. Chọn ảnh/video dùng hộp chọn tệp của hệ điều hành (trên điện thoại có thể chọn từ máy ảnh hoặc thư viện).
