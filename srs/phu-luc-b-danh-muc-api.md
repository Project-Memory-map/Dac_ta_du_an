[← Mục lục SRS](README.md)

# Phụ lục B: Danh mục API

Quy ước chung (khung phản hồi, mã lỗi, xác thực) ở [mục 6.2](06-yeu-cau-giao-dien-ngoai.md#62-giao-diện-phần-mềm-rest-api). Cột **Quyền**: *Công khai* = không cần đăng nhập; *Tùy chọn* = không bắt buộc, có JWT thì chủ ký ức thấy thêm dữ liệu của mình; *Đăng nhập* = cần JWT hợp lệ và tài khoản không bị khóa; *Chủ* = phải là chủ của đối tượng; *Admin* = `role = 'admin'`. Tài liệu tương tác: `/docs` khi chạy máy chủ.

## B.1 Lõi

| Phương thức | Đường dẫn | Quyền | Mô tả |
|---|---|---|---|
| GET | `/api/health` | Công khai | Kiểm tra kết nối CSDL: 200 hoặc 503 |

## B.2 M1 Tài khoản

| Phương thức | Đường dẫn | Quyền | Mô tả | Story |
|---|---|---|---|---|
| POST | `/api/auth/register` | Công khai | Đăng ký `{display_name, email, password}` → 201 + JWT | US-ACC-001 |
| POST | `/api/auth/login` | Công khai | Đăng nhập `{email, password}` → JWT | US-ACC-002 |
| GET | `/api/auth/me` | Đăng nhập | Thông tin tài khoản (`has_password`, `providers`, `avatar_url`) | US-ACC-002 |
| GET | `/api/auth/providers` | Công khai | `{google, facebook}`: đã cấu hình đủ khóa chưa | US-ACC-003/004 |
| GET | `/api/auth/google/login` | Công khai | Chuyển hướng sang Google | US-ACC-003 |
| GET | `/api/auth/google/callback` | Công khai | Nhận kết quả Google → `/accounts/oauth-callback.html#token=…` hoặc trang đăng nhập kèm lỗi | US-ACC-003 |
| GET | `/api/auth/facebook/login` | Công khai | Chuyển hướng sang Facebook | US-ACC-004 |
| GET | `/api/auth/facebook/callback` | Công khai | Nhận kết quả Facebook (như Google) | US-ACC-004 |
| POST | `/api/auth/forgot-password` | Công khai | Gửi liên kết đặt lại `{email}`; dev trả `dev_reset_url` | US-ACC-005 |
| POST | `/api/auth/validate-reset-token` | Công khai | Kiểm liên kết `{token}` | US-ACC-005 |
| POST | `/api/auth/reset-password` | Công khai | Đặt mật khẩu mới `{token, new_password}` | US-ACC-005 |
| POST | `/api/auth/change-password` | Đăng nhập | Đổi/tạo mật khẩu `{current_password?, new_password}` → JWT mới | US-ACC-007 |
| POST | `/api/auth/logout` | Đăng nhập | Đăng xuất (phía trình duyệt) | US-ACC-008 |
| PATCH | `/api/users/me` | Đăng nhập | Sửa `{display_name?, email?, bio?}` | US-ACC-006 |
| POST | `/api/users/me/avatar` | Đăng nhập | Tải ảnh đại diện (multipart `avatar`) | US-ACC-006 |
| DELETE | `/api/users/me/avatar` | Đăng nhập | Gỡ ảnh đại diện | US-ACC-006 |
| GET | `/api/users/{id}/avatar` | Công khai | Tệp ảnh đại diện (cache 1 ngày) | US-ACC-006 |
| GET | `/api/users/me/stats` | Đăng nhập | `{memories, public_memories, trips, places}` | US-ACC-006 |

## B.3 M2 Ký ức, chuyến đi, bản đồ

| Phương thức | Đường dẫn | Quyền | Mô tả | Story |
|---|---|---|---|---|
| GET | `/api/geocode?q=` | Công khai | Tìm địa điểm theo tên (≤ 5 kết quả) | US-MEM-001 |
| GET | `/api/geocode/reverse?lat=&lng=` | Công khai | Tọa độ → tên địa điểm | US-MEM-002 |
| POST | `/api/memories` | Đăng nhập | Tạo ký ức (multipart) → 201 | US-MEM-001/002 |
| GET | `/api/memories/{id}` | Tùy chọn | Chi tiết ký ức theo quyền xem | US-MEM-005 |
| PATCH | `/api/memories/{id}` | Chủ | Sửa ký ức (multipart, trường tùy chọn) | US-MEM-006 |
| DELETE | `/api/memories/{id}` | Chủ | Xóa mềm, gỡ khỏi chuyến đi | US-MEM-007 |
| PATCH | `/api/memories/{id}/visibility` | Chủ | `{visibility}` | US-MEM-008 |
| GET | `/api/memories/{id}/images` | Tùy chọn | Danh sách ảnh (URL ký sẵn) | US-MEM-003 |
| POST | `/api/memories/{id}/images` | Chủ | Thêm/bỏ/sắp xếp ảnh (`images[]`, `remove_media_ids`, `image_order`) | US-MEM-003 |
| GET | `/api/media/{id}?exp=&sig=` | Tùy chọn + chữ ký | Tệp ảnh | US-MEM-003 |
| GET | `/api/memories/{id}/video` | Tùy chọn | Thông tin video (URL ký sẵn) | US-MEM-004 |
| POST | `/api/memories/{id}/video` | Chủ | Tải/thay/gỡ video (`video`, `remove_video`) | US-MEM-004 |
| GET | `/api/media/{id}/stream?exp=&sig=` | Tùy chọn + chữ ký | Phát video, hỗ trợ `Range` | US-MEM-004 |
| GET | `/api/memories/{id}/hashtags` | Tùy chọn | Hashtag kèm nguồn | US-MEM-010 |
| POST | `/api/memories/{id}/hashtags` | Chủ | Thêm hashtag `{hashtags, ai_hashtags?, replace?}` | US-MEM-010 |
| PUT, PATCH | `/api/memories/{id}/hashtags` | Chủ | Thay toàn bộ hashtag | US-MEM-010 |
| DELETE | `/api/memories/{id}/hashtags/{tag}` | Chủ | Bỏ một hashtag | US-MEM-010 |
| GET | `/api/hashtags`, `/api/memories/mine/hashtags` | Đăng nhập | Hashtag của tôi kèm số ký ức | US-MEM-010, US-EXP-005 |
| POST | `/api/memories/suggest-hashtags` | Đăng nhập | Gợi ý `{title, content, place_name}` → `{hashtags, engine}` | FR-3.6 |
| GET | `/api/memories/mine` | Đăng nhập | Ký ức của tôi, lọc + phân trang | US-EXP-005 |
| GET | `/api/memories/mine/pins` | Đăng nhập | Ghim xanh (≤ 1000) | US-EXP-005 |
| GET | `/api/memories/mine/years` | Đăng nhập | Các năm có ký ức | US-EXP-005 |
| GET | `/api/trips` | Đăng nhập | Chuyến đi của tôi (tóm tắt) | US-MEM-009 |
| POST | `/api/trips` | Đăng nhập | Tạo `{name, description?, memory_ids?}` → 201 | US-MEM-009 |
| GET | `/api/trips/{id}` | Chủ | Chi tiết + ký ức đã xếp | US-MEM-009 |
| PATCH, PUT | `/api/trips/{id}` | Chủ | Sửa `{name, description}` | US-MEM-009 |
| DELETE | `/api/trips/{id}` | Chủ | Xóa chuyến đi (giữ ký ức) | US-MEM-009 |
| POST | `/api/trips/{id}/memories` | Chủ | Thêm `{memory_ids}` | US-MEM-009 |
| DELETE | `/api/trips/{id}/memories/{memory_id}` | Chủ | Bỏ ký ức khỏi chuyến đi | US-MEM-009 |

## B.4 M3 Khám phá

| Phương thức | Đường dẫn | Quyền | Mô tả | Story |
|---|---|---|---|---|
| GET | `/api/explore/pins` | Công khai | Ghim công khai trong khung `south, west, north, east` (≤ 500); lọc `q`, `hashtags`, `region` | US-EXP-001–004 |
| GET | `/api/explore/memories` | Công khai | Danh sách công khai, cùng bộ lọc, `page`, `page_size` (12, tối đa 50) | US-EXP-001–004 |
| GET | `/api/explore/overview` | Công khai | `{public_memories, regions}` | FR-3.8 |
| GET | `/api/explore/regions` | Công khai | 34 tỉnh/thành kèm số ký ức công khai | US-EXP-003 |
| GET | `/api/explore/hashtags?limit=` | Công khai | Hashtag phổ biến (mặc định 20, tối đa 100) | US-EXP-002 |

## B.5 M4 Quản trị (tiền tố `/api/admin`, quyền Admin)

| Phương thức | Đường dẫn | Mô tả | Story |
|---|---|---|---|
| GET | `/stats` | Thống kê tài khoản, ký ức | FR-4.11 |
| GET | `/memories?q=&status=&page=&page_size=` | Ký ức công khai chưa xóa | US-ADM-001 |
| POST | `/memories/{id}/hide` | Ẩn `{reason}` | US-ADM-001 |
| POST | `/memories/{id}/unhide` | Bỏ ẩn `{reason?}` | US-ADM-001 |
| POST | `/memories/{id}/delete` | Xóa mềm `{reason}` | US-ADM-002 |
| GET | `/users?q=&status=&page=&page_size=` | Tài khoản kèm số ký ức | US-ADM-003 |
| POST | `/users/{id}/lock` | Khóa `{reason}` | US-ADM-003 |
| POST | `/users/{id}/unlock` | Mở khóa `{reason?}` | US-ADM-003 |
| GET | `/actions?action=&page=&page_size=` | Nhật ký thao tác | FR-4.12 |

## B.6 Trang giao diện do máy chủ phục vụ

| Đường dẫn | Nguồn |
|---|---|
| `/` | `app/shared/ui/index.html` |
| `/ban-do` | `app/shared/ui/map.html` |
| `/js/*` | `app/shared/js/` |
| `/accounts/*`, `/accounts/js/*` | `app/modules/accounts/ui/`, `js/` |
| `/memories/*`, `/memories/js/*` | `app/modules/memories/ui/`, `js/` |
| `/explore/*`, `/explore/js/*` | `app/modules/explore/ui/`, `js/` |
| `/admin/`, `/admin/js/*` | `app/modules/admin/ui/`, `js/` |
