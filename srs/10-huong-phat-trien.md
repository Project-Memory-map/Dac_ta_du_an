[← Mục lục SRS](README.md)

# 10. Hướng phát triển

## 10.1 Việc nên làm trước (nợ kỹ thuật của GĐ1)

Các việc dưới đây hoàn thiện yêu cầu đã có, nên làm trước khi mở rộng tính năng. Chi tiết và lý do ở mục 11.3.

| Việc | Yêu cầu liên quan | Độ khó |
|---|---|---|
| Sửa form ký ức để lưu hashtag và chuyến đi đã chọn | FR-2.7, FR-2.9, US-MEM-010 | Thấp |
| Viết script dọn tệp của ký ức đã xóa mềm quá 30 ngày, chạy hàng tháng | BR-13 | Thấp |
| Tạo ảnh thu nhỏ khi tải ảnh lên, dùng trong danh sách | NFR-P4 | Trung bình |
| Thống nhất cách liên kết tài khoản Facebook theo email với BR-18 | BR-18, FR-1.9 | Thấp |
| Rút ngắn thời hạn JWT hoặc thêm cơ chế thu hồi khi đăng xuất | NFR-S3, L11 | Trung bình |
| Kiểm thử tải 100 người dùng đồng thời; kiểm trên Chrome, Firefox, Safari và điện thoại thật | NFR-P1, NFR-P2, NFR-U1 | Trung bình |
| Lưu ảnh/video trên dịch vụ lưu trữ đối tượng (Cloudflare R2, presigned URL) khi triển khai thật | BR-17 | Trung bình |
| Sao lưu CSDL tự động hàng tuần | NFR-R1 | Thấp |
| Hỗ trợ ảnh .heic từ iPhone bằng cách chuyển sang .jpg khi tải lên | FR-2.4 | Trung bình |
| Gửi email xác thực khi đăng ký, ghi `email_verified_at` | FR-1.1, BR-18 | Trung bình |

## 10.2 Giai đoạn 2: tương tác cộng đồng

FR-4.1 đến FR-4.5 và FR-4.7 (thích, bình luận, "Muốn đi", theo dõi, báo cáo vi phạm và hàng đợi xử lý). Thiết kế bảng đã chuẩn bị sẵn ở cuối [schema.sql](../schema.sql) (phần chú thích "GIAI ĐOẠN 2"): `memory_likes`, `comments`, `wishlist_items`, `follows`, `reports`. Khi làm cần thêm loại thao tác vào `admin_actions.action` cho xử lý báo cáo và xóa bình luận.

## 10.3 Ý tưởng mở rộng

Tham khảo từ tài liệu góp ý nhóm thu thập; chọn ý nào thì mới chuyển thành yêu cầu và user story.

| **Ý tưởng** | **Mô tả ngắn** | **Độ khó** | **Tác động thiết kế** |
|---|---|---|---|
| Thanh trượt thời gian | Kéo qua các năm để ghim hiện/ẩn trên bản đồ | Thấp | Dùng `memory_date` và `/api/memories/mine/years` đã có |
| Xuất dữ liệu (.zip, .pdf) | Tải toàn bộ ảnh, bài viết về máy | Thấp–TB | Thêm API xuất, không đổi CSDL |
| Trang chi tiết kiểu nhật ký | Ảnh toàn màn hình, chọn font chữ | Trung bình | Có thể thêm cột `font_style` cho ký ức |
| "Ngày này năm xưa" | Nhắc lại ký ức cùng ngày các năm trước | Trung bình | Truy vấn theo ngày/tháng; gửi email cần bảng thông báo |
| Hộp viên nang thời gian | Khóa ký ức tới ngày chỉ định | Trung bình | Thêm cột `unlock_at` |
| Nhạc nền theo ký ức | Gắn liên kết Spotify/YouTube | Trung bình | Thêm cột `music_url`, chỉ nhúng liên kết |
| Ghi âm giọng nói | Đính kèm bản ghi âm | TB–Cao | Thêm `media_type = 'audio'`, giới hạn dung lượng |
| Gắn thẻ người tham gia | Tìm ký ức theo người cùng đi | Cao | Thêm bảng người tham gia, cơ chế đồng ý |
| Không gian ký ức dùng chung | Bản đồ nhóm nhiều người cùng ghim | Cao | Thêm bảng nhóm, thành viên, phân quyền |
| Chia sẻ chuyến đi công khai | Gửi liên kết xem lộ trình cho người khác | Trung bình | Thêm cột `visibility` cho `trips` |
| AI viết lại câu chuyện, gợi ý tiêu đề | Mở rộng tích hợp Gemini đã có | Thấp–TB | Dùng lại cấu hình `GEMINI_*` |
