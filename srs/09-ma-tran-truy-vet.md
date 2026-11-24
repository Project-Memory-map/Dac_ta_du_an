[← Mục lục SRS](README.md)

# 9. Ma trận truy vết và kết quả kiểm thử

## 9.1 Tổng quan kiểm thử

| Loại | Phạm vi | Môi trường | Kết quả |
|---|---|---|---|
| Test tự động (`pytest`) | 238 test theo từng story/AC, chạy khoảng 20 giây | SQLite trong bộ nhớ, nhà cung cấp OAuth và Nominatim giả lập | Tất cả đạt (yêu cầu bắt buộc trước khi mở PR) |
| Kiểm thử tích hợp qua API (`kiem_thu_api.py`) | 108 AC + 3 phép kiểm lỗi tiềm ẩn (lệch giờ, lộ đường dẫn ảnh đại diện, gán tỉnh/thành) | Máy chủ thật, MySQL 8 thật (có trigger, FULLTEXT), Nominatim thật | Lần 2: 103 đạt, 5 cần cấu hình |
| Kiểm thử giao diện (`kiem_thu_giao_dien.mjs`) | 46 ca cần trình duyệt: đăng ký, đăng nhập, tạo ký ức, ảnh/video, xác nhận xóa, nhấp đúp/chạm giữ, hồ sơ, đặt lại mật khẩu, đăng xuất + nút Back, quản trị, vùng trống | Edge máy tính 1366×800 và Pixel 7 giả lập (Playwright) | 46/46 đạt, 0 lỗi JavaScript, không tràn ngang |

Diễn biến kiểm thử tích hợp (task T05B58C):

| Lần chạy | Đạt | Lỗi | Chặn | Cần cấu hình |
|---|---|---|---|---|
| Lần 1, trước khi sửa (21/11/2026) | 86 | 5 | 17 | — |
| Lần 2, sau khi sửa 16 lỗi | **103** | **0** | **0** | **5** |

18 lỗi tìm được: 16 đã sửa, 1 ghi nhận theo thiết kế (L11, token còn dùng được sau đăng xuất), 1 chờ khóa OAuth (L12). Chi tiết ở `tests/kiem_thu_tich_hop/danh_sach_loi.json` và sheet "Danh sách lỗi" của báo cáo.

Giới hạn của kiểm thử hiện có: kiểm thử tích hợp tạo hashtag bằng cách gọi thẳng API hashtag, nên không phát hiện lỗi form ký ức không lưu hashtag/chuyến đi (mục 11.3). Chưa có kiểm thử tải (NFR-P1, NFR-P2) và chưa kiểm trên điện thoại thật.

## 9.2 Ma trận yêu cầu chức năng

| **FR** | **User story** | **Test tự động** | **Tích hợp** | **Trạng thái** |
|---|---|---|---|---|
| FR-1.1 | US-ACC-001 | `accounts/test_register.py` | 5/5 | ✅ |
| FR-1.2 | US-ACC-002 | `accounts/test_login.py` | 5/5 | ✅ |
| FR-1.3 | US-ACC-003 | `accounts/test_us_acc_003_google.py` | 1/4, 3 cần cấu hình | 🔧 |
| FR-1.4 | US-ACC-004 | `accounts/test_us_acc_004_facebook.py` | 1/3, 2 cần cấu hình | 🔧 |
| FR-1.5 | US-ACC-005 | `accounts/test_us_acc_005_reset_password.py` | 4/4 | ✅ |
| FR-1.6 | US-ACC-007, US-ACC-008 | `accounts/test_us_acc_007_change_password.py`, `test_us_acc_008_logout.py` | 7/7 | ✅ |
| FR-1.7 | US-ACC-006 | `accounts/test_us_acc_006_profile.py` | 5/5 | ✅ |
| FR-1.8 | US-ACC-006 | `accounts/test_us_acc_006_profile.py` | (chung) | ✅ |
| FR-1.9 | US-ACC-003, US-ACC-004 | như FR-1.3, FR-1.4 | (chung) | 🔧 · ⚠️ Facebook |
| FR-1.10 | US-ACC-006 | `accounts/test_us_acc_006_profile.py` | — | ✅ |
| FR-1.11 | US-ACC-003, US-ACC-004 | test `providers` trong 2 file OAuth | — | ✅ |
| FR-2.1 | US-EXP-001 | `test_us_exp_001_004.py` | 4/4 | ✅ |
| FR-2.2 | US-MEM-001, US-MEM-002 | `memories/test_us_mem_001_geocode_search.py`, `test_us_mem_002_pick_on_map.py` | 8/8 | ✅ |
| FR-2.3 | US-MEM-001 | `memories/test_us_mem_001_geocode_search.py`, `test_us_mem_002_pick_on_map.py` | (chung) | ✅ |
| FR-2.4 | US-MEM-003, US-MEM-004 | `test_us_mem_003_images.py`, `test_us_mem_004_video.py` | 9/9 | ✅ |
| FR-2.5 | US-MEM-006, US-MEM-007 | `memories/test_region_assignment_and_media.py`, `memories/test_us_mem_007_delete.py` | 8/8 | ✅ |
| FR-2.6 | US-MEM-008 | `memories/test_us_mem_008_privacy.py` | 3/3 | ✅ |
| FR-2.7 | US-MEM-009 | `memories/test_us_mem_009_trips.py` | 5/5 | ✅ · ⚠️ ô chọn trong form |
| FR-2.8 | US-MEM-005 | `memories/test_us_mem_008_privacy.py`, `test_us_exp_001_004.py` | 4/4 | ✅ |
| FR-2.9 | US-MEM-010 | `memories/test_us_mem_010_hashtags.py` | 4/4 (qua API) | ⚠️ form |
| FR-2.10 | US-MEM-003, 004, 006 | `test_us_mem_003_images.py`, `test_us_mem_004_video.py` | (chung) | ✅ |
| FR-2.11 | US-MEM-009 | `memories/test_us_mem_009_trips.py` | (chung) | ✅ |
| FR-2.12 | — (giao diện) | — | kiểm tay | ✅ |
| FR-2.13 | US-EXP-003, US-MEM-006 | `memories/test_region_assignment_and_media.py`, `test_us_exp_003_regions.py` | (chung) | ✅ |
| FR-3.1 | US-EXP-002, US-EXP-003 | `test_us_exp_002_hashtags.py`, `test_us_exp_003_regions.py` | 8/8 | ✅ |
| FR-3.2 | US-EXP-004 | `test_us_exp_001_004.py` | 4/4 | ✅ |
| FR-3.3 | US-EXP-005 | `test_us_exp_005_mine.py` | 4/4 | ✅ |
| FR-3.4 | US-EXP-002 | `test_us_exp_002_hashtags.py` | (chung) | ✅ |
| FR-3.5 | US-EXP-001 | `test_us_exp_001_004.py` | (chung) | ✅ |
| FR-3.6 | US-MEM-010 (mở rộng) | `test_fr_3_6_ai_hashtags.py` | — | ✅ gợi ý · ⚠️ lưu từ form |
| FR-3.7 | US-EXP-001 | `test_us_exp_001_004.py` | (chung) | ✅ |
| FR-3.8 | US-EXP-001, US-EXP-002 | `test_us_exp_002_hashtags.py`, `test_us_exp_003_regions.py` | — | ✅ |
| FR-4.1 – FR-4.5, FR-4.7 | — | — | — | ⛔ GĐ2 |
| FR-4.6 | US-ADM-001, 002, 003 | `admin/test_us_adm_00*.py` | 14/14 | ✅ |
| FR-4.8 | US-ADM-001 | `admin/test_us_adm_001_hide_memory.py` | (chung) | ✅ |
| FR-4.9 | US-ADM-001, 002, 003 | `admin/test_us_adm_00*.py` | (chung) | ✅ |
| FR-4.10 | US-ADM-003 | `admin/test_us_adm_003_lock_user.py` | (chung) | ✅ |
| FR-4.11 – FR-4.13 | US-ADM-001 | `admin/test_us_adm_001_hide_memory.py` | — | ✅ |

Đường dẫn test tính từ thư mục `tests/` của repo. "(chung)" là AC đã tính ở dòng FR phía trên của cùng story.

## 9.3 Ma trận quy tắc nghiệp vụ

| **BR** | **Story / AC chứng minh** |
|---|---|
| BR-01 | US-MEM-001 AC1, US-MEM-002 AC1 |
| BR-02 | US-MEM-006 AC3, US-MEM-007 AC4, US-MEM-008 AC3 |
| BR-03 | US-EXP-001 AC1, AC4; US-ADM-001 AC1; US-ADM-003 (ghi chú) |
| BR-04 | US-MEM-005 AC3, US-MEM-008 AC2 |
| BR-05 | US-MEM-001 AC5 |
| BR-06 | US-MEM-003 AC2–AC5, US-MEM-004 AC2–AC4, US-MEM-010 AC3–AC4 |
| BR-07 | US-ACC-001 AC2–AC3, US-ACC-007 AC3 |
| BR-08 | US-ACC-001 AC4, US-ACC-002 AC2, US-ACC-003 AC2 |
| BR-09 | US-MEM-009 AC1–AC3 |
| BR-10 | US-ADM-001 AC1, AC3; US-ADM-002 AC1–AC2; US-ADM-003 AC1 |
| BR-11 | US-ADM-003 AC3 |
| BR-12 | US-EXP-001 AC3 |
| BR-13 | US-MEM-007 AC1 (phần xóa mềm) |
| BR-14 | US-ACC-005 AC3–AC4 |
| BR-15 | US-ACC-002 AC5 |
| BR-16 | US-EXP-002 AC2, US-EXP-003 AC3 |
| BR-17 | US-MEM-008 AC2, US-MEM-007 AC1 |
| BR-18 | US-ACC-003 AC2 |
| BR-19 | US-MEM-001 AC1, AC3 |
| BR-20 | US-ACC-005 AC3, US-ACC-007 AC1, US-ADM-003 AC1 |
| BR-21 – BR-30 | Kiểm bằng test tự động và phép kiểm bổ sung của kiểm thử tích hợp; không có AC riêng |
