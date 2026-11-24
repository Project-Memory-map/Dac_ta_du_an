[← Mục lục SRS](README.md)

# 3. Tổ chức module và tiến độ thực hiện

## 3.1 Bốn module chức năng

Mã nguồn chia theo chức năng. Mỗi module có `ui/` (HTML, CSS), `js/` (xử lý trình duyệt) và các file `.py` (API, lưu trữ). Mỗi module tương ứng một epic ở mục 5.

| Module | Thư mục | Epic | User story | Bảng CSDL sở hữu |
|---|---|---|---|---|
| Lõi dùng chung | `app/core/`, `app/shared/` | — | — | — |
| M1 Tài khoản | `app/modules/accounts/` | EP1 | US-ACC-001 → 008 | `users`, `oauth_accounts`, `password_reset_tokens` |
| M2 Ký ức, chuyến đi, bản đồ | `app/modules/memories/` | EP2 (+ US-EXP-005, FR-3.6) | US-MEM-001 → 010, US-EXP-005 | `memories`, `media`, `trips`, `trip_memories`, `hashtags`, `memory_hashtags` |
| M3 Khám phá | `app/modules/explore/` | EP3 | US-EXP-001 → 004 | `regions` |
| M4 Quản trị | `app/modules/admin/` | EP4 | US-ADM-001 → 003 | `admin_actions` |

Lõi dùng chung gồm: đọc cấu hình (`config.py`), kết nối MySQL (`database.py`), JWT và ký URL (`security.py`), băm mật khẩu (`passwords.py`), khung phản hồi (`response.py`), lớp lỗi và bộ xử lý lỗi (`errors.py`); phía trình duyệt có `api.js` (gọi API, giữ JWT), `ui.js` (hộp thoại, thông báo, dựng DOM an toàn), `map.js` (Leaflet), `shell.js` (khung trang bản đồ, tìm địa điểm).

Quan hệ giữa các module: M2, M3, M4 lấy người dùng đăng nhập qua `accounts/deps.py`; M2 gọi `explore/region_lookup.py` để gán tỉnh/thành; M1 dùng dịch vụ ảnh của M2 cho ảnh đại diện; M4 cập nhật `memories.status`, `memories.deleted_at`, `users.status`.

## 3.2 Tiến độ

| Giai đoạn | Nội dung | Kết quả |
|---|---|---|
| Sprint 0 | Dựng khung dự án dùng chung: lõi, `get_current_user`, `map.js`, khung phản hồi API, schema MySQL | Hoàn thành |
| Sprint 1–3 | 26 user story của 4 epic, mỗi story một nhánh và một pull request kèm test | Hoàn thành |
| FR-3.6 (T05B590) | Gợi ý hashtag bằng Gemini | Hoàn thành (vốn là tính năng tùy chọn) |
| Kiểm thử tích hợp (T05B58C, 21/11/2026) | Chạy lại 108 AC trên máy chủ thật, MySQL thật, Edge cỡ máy tính và điện thoại | Lần 1: 86 đạt, 5 lỗi, 17 chặn. Sửa 16 lỗi. Lần 2: **103 đạt, 0 lỗi, 0 chặn, 5 cần cấu hình OAuth** |

Story point, sprint cụ thể và người phụ trách từng story được ghi trong Product Backlog (.xlsx) và trên bảng Trello của nhóm; mã task Trello (dạng `T05B5xx`) nằm ở đầu mỗi commit.
