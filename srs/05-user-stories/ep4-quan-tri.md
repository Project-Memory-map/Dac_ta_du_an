[← Danh sách epic](README.md)

# 5.6 Epic EP4: Quản trị nội dung

Module M4 (`app/modules/admin/`). 3 story, 14 tiêu chí nghiệm thu. Kết quả kiểm thử tích hợp: 14/14 đạt.

Mọi API dưới `/api/admin` đi qua `get_current_admin`: cần đăng nhập (401 nếu không) và `role = 'admin'` (403 "Bạn không có quyền truy cập" nếu không). Trang `/admin/` mở bằng tài khoản thường thì chuyển về `/ban-do?notice=forbidden` và hiện đúng thông báo đó. Mọi thao tác ghi một dòng vào `admin_actions` trong cùng giao dịch với thay đổi dữ liệu; dòng ký ức/tài khoản bị khóa (`SELECT … FOR UPDATE`) để hai quản trị viên không cùng sửa một đối tượng.

## US-ADM-001: Ẩn và bỏ ẩn ký ức công khai vi phạm

> **Là một quản trị viên vận hành hệ thống, đã đăng nhập với vai trò quản trị, Tôi muốn ẩn một ký ức công khai vi phạm và bỏ ẩn khi cần, Để cộng đồng không nhìn thấy nội dung không phù hợp.**

| **Epic** | **EP4** | **Ưu tiên** | **Should** |
|---|---|---|---|
| FR liên quan | FR-4.6, FR-4.8, FR-4.9, FR-4.11, FR-4.12, FR-4.13 | Trạng thái | ✅ Hoàn thành |
| Phụ thuộc | US-ACC-002 | Kiểm thử | `tests/admin/test_us_adm_001_hide_memory.py`; tích hợp 5/5 đạt |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (5 AC).

### Tiêu chí nghiệm thu (5)

#### AC1: Ẩn kèm lý do

| **Given** | **quản trị viên đang xem ký ức công khai số 123** |
|---|---|
| When | quản trị viên chọn "Ẩn" và nhập lý do "Hình ảnh không phù hợp" |
| Then | ký ức biến mất khỏi bản đồ công khai |
| And | chủ ký ức thấy trạng thái "Đã bị quản trị viên ẩn" |
| And | hệ thống ghi nhật ký gồm người thao tác, thời điểm và lý do |

#### AC2: Bỏ ẩn

| **Given** | **ký ức số 123 đang bị ẩn và vẫn ở chế độ công khai** |
|---|---|
| When | quản trị viên chọn "Bỏ ẩn" |
| Then | ký ức xuất hiện lại trên bản đồ công khai |

#### AC3: Thiếu lý do

| **Given** | **quản trị viên đang ẩn một ký ức** |
|---|---|
| When | quản trị viên để trống lý do |
| Then | hệ thống hiển thị "Vui lòng nhập lý do" |
| And | ký ức không bị ẩn |

#### AC4: Người dùng thường dùng chức năng quản trị

| **Given** | **một người dùng không phải quản trị viên** |
|---|---|
| When | người dùng cố mở trang quản trị |
| Then | hệ thống hiển thị "Bạn không có quyền truy cập" |
| And | không có thay đổi nào xảy ra |

#### AC5: Tìm kiếm ký ức công khai để duyệt

| **Given** | **quản trị viên đang ở trang quản trị nội dung** |
|---|---|
| When | quản trị viên nhập từ khóa tiêu đề hoặc tên tác giả vào ô tìm kiếm |
| Then | danh sách chỉ hiển thị ký ức công khai khớp từ khóa, kèm trạng thái ẩn/hiện (FR-4.8) |

### Đặc tả cài đặt

- API: `GET /api/admin/stats`; `GET /api/admin/memories?q=&status=active|hidden&page=&page_size=` (chỉ ký ức công khai chưa xóa; `q` khớp tiêu đề, tên địa điểm hoặc tên tác giả; ký tự `%` và `_` người dùng gõ được hiểu là ký tự thường; mỗi dòng có ảnh bìa, tác giả và trạng thái tác giả); `POST /api/admin/memories/{id}/hide` `{reason}`; `POST /api/admin/memories/{id}/unhide` `{reason?}`.
- Lý do ẩn bắt buộc, cắt khoảng trắng, ≤ 500 ký tự ("Lý do tối đa 500 ký tự"); bỏ ẩn không bắt buộc lý do. Ẩn ký ức đang ẩn: 409 "Ký ức này đã bị ẩn"; bỏ ẩn ký ức đang hiển thị: 409 "Ký ức này đang hiển thị, không cần bỏ ẩn".
- Ẩn đặt `memories.status = 'hidden'`, không đổi `visibility`. Ký ức riêng tư hoặc đã xóa không thao tác được ("Ký ức không tồn tại").
- Trang quản trị có 3 tab: Ký ức công khai, Tài khoản, Nhật ký thao tác; phía trên là thống kê (FR-4.11). Nhật ký: `GET /api/admin/actions?action=` (lọc theo một trong 5 loại thao tác, phân trang 20).
- Quản trị viên cũng ẩn/bỏ ẩn được ngay trong chi tiết ký ức trên bản đồ (FR-4.13).
- Nội dung vi phạm gồm: hình ảnh/video đồi trụy, bạo lực, kích động thù hận, thông tin sai sự thật, xúc phạm danh dự cá nhân/tổ chức, vi phạm bản quyền nghiêm trọng.

## US-ADM-002: Xóa ký ức vi phạm nghiêm trọng

> **Là một quản trị viên vận hành hệ thống, đã đăng nhập với vai trò quản trị, Tôi muốn xóa hẳn một ký ức vi phạm nghiêm trọng, Để nội dung đó không còn tồn tại trong hệ thống.**

| **Epic** | **EP4** | **Ưu tiên** | **Should** |
|---|---|---|---|
| FR liên quan | FR-4.6, FR-4.9 | Trạng thái | ✅ Hoàn thành |
| Phụ thuộc | US-ADM-001 | Kiểm thử | `tests/admin/test_us_adm_002_delete_memory.py`; tích hợp 4/4 đạt |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (4 AC).

### Tiêu chí nghiệm thu (4)

#### AC1: Xóa kèm lý do

| **Given** | **quản trị viên đang xem ký ức số 123** |
|---|---|
| When | quản trị viên chọn "Xóa", nhập lý do và xác nhận |
| Then | ký ức và ảnh, video của ký ức không còn truy cập được |
| And | hệ thống ghi nhật ký thao tác kèm lý do |

#### AC2: Thiếu lý do

| **Given** | **quản trị viên đang xóa một ký ức** |
|---|---|
| When | quản trị viên để trống lý do |
| Then | hệ thống hiển thị "Vui lòng nhập lý do" |
| And | ký ức không bị xóa |

#### AC3: Ký ức đã bị chủ xóa trước đó

| **Given** | **chủ ký ức số 123 đã xóa ký ức** |
|---|---|
| When | quản trị viên mở ký ức số 123 |
| Then | hệ thống hiển thị "Ký ức không tồn tại" |

#### AC4: Người dùng thường dùng chức năng quản trị

| **Given** | **một người dùng không phải quản trị viên** |
|---|---|
| When | người dùng cố xóa ký ức bằng chức năng quản trị |
| Then | hệ thống từ chối (403) |
| And | ký ức không bị ảnh hưởng |

### Đặc tả cài đặt

- API: `POST /api/admin/memories/{id}/delete` `{reason}`. Xóa mềm như US-MEM-007 (`deleted_at = NOW()`); nhật ký thao tác vẫn giữ id ký ức. Ký ức không có, đã xóa hoặc không công khai: 404 "Ký ức không tồn tại".
- Khác với xóa của chủ ký ức, xóa của quản trị viên hiện **không** gỡ dòng `trip_memories`; ký ức vẫn không hiện trong chuyến đi vì mọi truy vấn chuyến đi bỏ qua ký ức đã xóa mềm.
- Giao diện hỏi lý do trong hộp nhập; câu hướng dẫn ghi "Ký ức sẽ bị xóa vĩnh viễn khỏi hệ thống" theo góc nhìn người dùng (về kỹ thuật vẫn là xóa mềm).

## US-ADM-003: Khóa và mở khóa tài khoản người dùng

> **Là một quản trị viên vận hành hệ thống, đã đăng nhập với vai trò quản trị, Tôi muốn khóa tài khoản vi phạm và mở khóa khi cần, Để ngăn người vi phạm tiếp tục sử dụng hệ thống.**

| **Epic** | **EP4** | **Ưu tiên** | **Should** |
|---|---|---|---|
| FR liên quan | FR-4.6, FR-4.9, FR-4.10 | Trạng thái | ✅ Hoàn thành |
| Phụ thuộc | US-ACC-002 | Kiểm thử | `tests/admin/test_us_adm_003_lock_user.py`; tích hợp 5/5 đạt |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (5 AC).

### Tiêu chí nghiệm thu (5)

#### AC1: Khóa tài khoản kèm lý do

| **Given** | **quản trị viên đang xem tài khoản "an@example.com"** |
|---|---|
| When | quản trị viên chọn "Khóa" và nhập lý do |
| Then | tài khoản không đăng nhập được, phiên đang mở bị đăng xuất ngay |
| And | hệ thống ghi nhật ký thao tác kèm lý do |

#### AC2: Mở khóa tài khoản

| **Given** | **tài khoản "an@example.com" đang bị khóa** |
|---|---|
| When | quản trị viên chọn "Mở khóa" |
| Then | người dùng đăng nhập lại được |
| And | ký ức và chuyến đi của người dùng còn nguyên |

#### AC3: Quản trị viên tự khóa mình

| **Given** | **quản trị viên đang xem tài khoản của chính mình** |
|---|---|
| When | quản trị viên chọn "Khóa" |
| Then | hệ thống từ chối |
| And | hệ thống hiển thị "Bạn không thể khóa chính tài khoản của mình" |

#### AC4: Người dùng thường dùng chức năng quản trị

| **Given** | **một người dùng không phải quản trị viên** |
|---|---|
| When | người dùng cố khóa một tài khoản khác |
| Then | hệ thống từ chối (403) |
| And | tài khoản kia không bị ảnh hưởng |

#### AC5: Tìm kiếm tài khoản

| **Given** | **quản trị viên đang ở trang quản trị tài khoản** |
|---|---|
| When | quản trị viên nhập email hoặc tên hiển thị vào ô tìm kiếm |
| Then | danh sách chỉ hiển thị tài khoản khớp từ khóa, kèm trạng thái active/locked (FR-4.10) |

### Đặc tả cài đặt

- API: `GET /api/admin/users?q=&status=active|locked&page=&page_size=` (mỗi dòng có tên hiển thị, email, vai trò, trạng thái, số ký ức); `POST /api/admin/users/{id}/lock` `{reason}`; `POST /api/admin/users/{id}/unlock` `{reason?}`.
- Khóa: đặt `users.status = 'locked'` và tăng `token_version` (BR-20), nên mọi JWT đang dùng hết hiệu lực ngay; đăng nhập bằng mật khẩu, Google, Facebook đều bị chặn. Khóa tài khoản đang khóa: 409 "Tài khoản này đã bị khóa"; mở khóa tài khoản đang hoạt động: 409 "Tài khoản này đang hoạt động, không cần mở khóa". Không có tài khoản: 404 "Không tìm thấy tài khoản".
- Toàn bộ ký ức công khai của người bị khóa biến mất khỏi cộng đồng vì mọi truy vấn công khai lọc tác giả `status = 'active'` (BR-03). Hệ thống không sửa `memories.status`, nên mở khóa thì ký ức hiện lại đúng trạng thái cũ (BR-29).
- Tự khóa mình bị chặn ở API (422) và ở trigger `trg_admin_actions_bi` (BR-11). Hiện hệ thống cho phép một quản trị viên khóa một quản trị viên khác; xem mục 11.3.
