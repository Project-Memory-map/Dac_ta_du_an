[← Mục lục SRS](README.md)

# 7. Yêu cầu phi chức năng

Cột **Mức**: Bắt buộc / Mục tiêu (con số nhóm đặt ra cho phạm vi đồ án). Cột **Hiện trạng** ghi cách bản cài đặt đáp ứng và đã kiểm chứng hay chưa.

## 7.1 Hiệu năng

| **Mã** | **Yêu cầu** | **Mức** | **Hiện trạng** |
|---|---|---|---|
| NFR-P1 | Trang chính tải trong ≤ 3 giây trên mạng thông thường. | Mục tiêu | ⏳ Chưa đo bằng công cụ. Trang không dùng framework, JS/CSS tách nhỏ; ảnh đại diện cache 1 ngày. |
| NFR-P2 | Hỗ trợ tối thiểu 100 người dùng đồng thời, thời gian phản hồi API < 1 giây. | Mục tiêu | ⏳ Chưa kiểm thử tải. Kết nối CSDL dùng pool (`pool_pre_ping`, `pool_recycle=3600`); truy vấn công khai có chỉ mục `idx_mem_pub`, `idx_mem_public_geo`. |
| NFR-P3 | Bản đồ chỉ tải ghim trong vùng đang xem, tối đa 500 ghim công khai mỗi lần. | Bắt buộc | ✅ `/api/explore/pins` lọc theo khung và giới hạn 500; tải lại sau 250 ms khi ngừng kéo. Ghim của tôi tối đa 1000. |
| NFR-P4 | Danh sách phân trang; ảnh trong danh sách dùng bản thu nhỏ, tải ảnh gốc khi xem chi tiết. | Bắt buộc | ⚠️ Phân trang đạt (12/20 mục, "Tải thêm"). **Chưa tạo ảnh thu nhỏ**: `thumb_path` luôn rỗng, danh sách dùng ảnh gốc (≤ 5 MB). |
| NFR-P5 | Không làm máy chủ treo khi dịch vụ ngoài chậm. | Bắt buộc | ✅ Thời gian chờ: Nominatim 5 giây, Google/Facebook 5 giây, Gemini 8 giây; email gửi nền; Nominatim xếp hàng tối đa 5 giây. |
| NFR-P6 | Video phát được ngay, không phải tải hết. | Bắt buộc | ✅ Hỗ trợ `Range` (206); ký ức công khai phát thẳng từ URL. |

## 7.2 Bảo mật

| **Mã** | **Yêu cầu** | **Mức** | **Hiện trạng** |
|---|---|---|---|
| NFR-S1 | Mật khẩu băm bằng bcrypt (salt riêng), không lưu dạng thô; giới hạn 72 byte. | Bắt buộc | ✅ |
| NFR-S2 | Ảnh/video không truy cập được qua đường dẫn tĩnh; chỉ qua URL có chữ ký HMAC-SHA256 kèm hạn dùng, và mỗi lần tải vẫn kiểm quyền xem (BR-17). | Bắt buộc | ✅ Thư mục `uploads/` không được phục vụ tĩnh. Hạn URL **1 giờ** (`media_url_expire_seconds = 3600`), dài hơn mức "vài phút" của bản 0.3 để video dài không đứt giữa chừng. |
| NFR-S3 | Phiên đăng nhập bằng JWT có hạn; thu hồi ngay bằng `token_version` khi đổi/đặt lại mật khẩu hoặc bị khóa (BR-20). | Bắt buộc | ✅ JWT HS256, hạn mặc định 1440 phút (24 giờ). ⚠️ Đăng xuất không thu hồi JWT ở máy chủ (rủi ro L11, mục 11.3). |
| NFR-S4 | Dùng HTTPS khi triển khai thật. | Bắt buộc | ⏳ Phụ thuộc cách triển khai. Khi `APP_ENV` khác dev, cookie `state` OAuth bật cờ `Secure`. |
| NFR-S5 | Kiểm loại và dung lượng tệp ở máy chủ. | Bắt buộc | ✅ Kiểm đuôi, MIME và chữ ký nhị phân; dừng đọc ngay khi vượt dung lượng; trigger `trg_media_bi` chặn thêm ở CSDL. |
| NFR-S6 | Chống SQL injection (ORM, truy vấn tham số hóa) và XSS (BR-21). | Bắt buộc | ✅ Chỉ dùng SQLAlchemy; `LIKE` trong trang quản trị có thoát ký tự `%`, `_`; giao diện không dùng `innerHTML` với dữ liệu người dùng. |
| NFR-S7 | Khóa bí mật không đưa lên kho mã nguồn. | Bắt buộc | ✅ `.env` và `uploads/` nằm trong `.gitignore`; có `.env.example`. |
| NFR-S8 | *(mới ở 1.0)* Header bảo mật cho mọi phản hồi. | Bắt buộc | ✅ `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: strict-origin-when-cross-origin`; trang HTML `Cache-Control: no-cache`; API đăng nhập `no-store`. CORS chỉ cho `localhost:8000` và `127.0.0.1:8000`. |
| NFR-S9 | *(mới ở 1.0)* Chống giả mạo callback OAuth. | Bắt buộc | ✅ Tham số `state` ngẫu nhiên lưu trong cookie HttpOnly, SameSite=Lax, sống 10 phút, so khớp bằng phép so sánh thời gian hằng. |
| NFR-S10 | *(mới ở 1.0)* Không để lộ email nào đã đăng ký. | Bắt buộc | ✅ Đăng nhập sai và quên mật khẩu trả cùng thông báo cho email có/không tồn tại; email không tồn tại vẫn chạy bcrypt với bản băm giả. |
| NFR-S11 | *(mới ở 1.0)* Không đọc/ghi tệp ngoài thư mục `uploads/`. | Bắt buộc | ✅ Mọi đường dẫn tệp đi qua `resolve_upload_path()`, từ chối đường dẫn thoát ra ngoài thư mục gốc; tên tệp lưu là chuỗi ngẫu nhiên, không dùng tên người dùng gửi. |
| NFR-S12 | *(mới ở 1.0)* Không lộ vị trí qua ảnh (BR-22). | Bắt buộc | ✅ Xóa EXIF/GPS/XMP khỏi ảnh ký ức và ảnh đại diện trước khi lưu. |
| NFR-S13 | *(mới ở 1.0)* Lỗi không lộ chi tiết kỹ thuật. | Bắt buộc | ✅ Lỗi 500 chỉ trả câu chung; chi tiết ghi log máy chủ. Log gửi email không ghi token hay mật khẩu SMTP. |

## 7.3 Tương thích và dễ dùng

| **Mã** | **Yêu cầu** | **Mức** | **Hiện trạng** |
|---|---|---|---|
| NFR-U1 | Chạy được trên Chrome, Edge, Firefox, Safari trên máy tính và điện thoại. | Bắt buộc | ⚠️ Kiểm thử tích hợp chạy trên Edge (máy tính 1366×800 và Pixel 7 giả lập): 46/46 ca giao diện đạt, không lỗi JavaScript. Các trình duyệt khác và điện thoại thật chưa kiểm có hệ thống. |
| NFR-U2 | Hiển thị đúng trên màn hình từ 360 px. | Bắt buộc | ✅ Không trang nào tràn ngang ở cỡ điện thoại. |
| NFR-U3 | Giao diện và thông báo lỗi bằng tiếng Việt. | Bắt buộc | ✅ Kể cả lỗi kiểu dữ liệu của FastAPI được dịch sang câu tiếng Việt ("Vui lòng nhập trường này", "Ngày không hợp lệ"...). |
| NFR-U4 | Ghim một ký ức hoàn thành trong vài bước. | Mục tiêu | ✅ Nhấp đúp → điền tiêu đề, ngày → Lưu (tên địa điểm tự điền). |
| NFR-U5 | *(mới ở 1.0)* Khi dịch vụ ngoài lỗi, người dùng vẫn có cách khác. | Bắt buộc | ✅ Tìm địa điểm lỗi → chọn trên bản đồ; reverse geocode lỗi → tự nhập tên; Gemini lỗi → gợi ý theo từ khóa; thiếu khóa OAuth → nút làm mờ. |

## 7.4 Độ tin cậy, bảo trì và kiểm thử

| **Mã** | **Yêu cầu** | **Mức** | **Hiện trạng** |
|---|---|---|---|
| NFR-M1 | Mã nguồn chia module rõ ràng, tách HTML, CSS, JS; mỗi thư mục có README mô tả từng file. | Bắt buộc | ✅ 4 module + lõi; mỗi API chỉ khai báo ở một file. |
| NFR-M2 | Mỗi tiêu chí nghiệm thu chuyển được thành ca kiểm thử. | Bắt buộc | ✅ 238 test pytest (khoảng 20 giây, SQLite trong bộ nhớ) + kiểm thử tích hợp 108 AC trên MySQL thật và trình duyệt. |
| NFR-M3 | *(mới ở 1.0)* Quy tắc nghiệp vụ quan trọng được chặn ở cả ứng dụng và CSDL. | Bắt buộc | ✅ 11 trigger MySQL (BR-02, 05, 06, 08, 09, 10, 11, 20; nhật ký không sửa/xóa được). Lỗi trigger trả 422 kèm câu của trigger. |
| NFR-M4 | *(mới ở 1.0)* Mốc thời gian bản ghi dùng một đồng hồ. | Bắt buộc | ✅ `created_at` do MySQL ghi; `updated_at`, `deleted_at` gán bằng `func.now()` (giờ CSDL), không dùng giờ UTC của Python (lỗi L14 đã sửa). |
| NFR-R1 | Sao lưu CSDL hàng tuần bằng `mysqldump`, lưu ngoài thư mục mã nguồn. | Bắt buộc khi vận hành | ⏳ Quy trình vận hành, chưa có script hay lịch tự động trong repo. |
| NFR-R2 | *(mới ở 1.0)* Kiểm tra sức khỏe hệ thống. | Bắt buộc | ✅ `GET /api/health` trả 200 khi kết nối được CSDL, 503 khi không. |
| NFR-R3 | *(mới ở 1.0)* Ghi log. | Bắt buộc | ✅ Log mức INFO có thời gian, mức độ, nguồn; ghi lỗi CSDL, lỗi không lường trước, thao tác bị trigger chặn, lỗi gửi email, lỗi gọi Gemini. |
| NFR-R4 | *(mới ở 1.0)* Thao tác nhiều bước không để dữ liệu dở dang. | Bắt buộc | ✅ Ghi CSDL trong giao dịch, lỗi thì hoàn tác và xóa các tệp vừa ghi; khóa dòng khi đếm sai mật khẩu, sửa ảnh, thao tác quản trị. |
| NFR-R5 | *(mới ở 1.0)* Tạo lại CSDL dễ dàng, an toàn. | Bắt buộc | ✅ `scripts/init_db.py` chạy lại không mất dữ liệu (`--xoa-cu` để tạo lại từ đầu); `seed_demo.py` tạo dữ liệu mẫu. |
