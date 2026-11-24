[← Danh sách epic](README.md)

# 5.5 Epic EP3: Khám phá, bộ lọc và tìm kiếm

Module M3 (`app/modules/explore/`) cho US-EXP-001 đến 004; US-EXP-005 nằm ở module M2. 5 story, 20 tiêu chí nghiệm thu. Kết quả kiểm thử tích hợp: 20/20 đạt.

Mọi truy vấn công khai đi qua một hàm chung (`public_query()` trong `explore/router.py`, tương đương view `v_public_memories` trong CSDL), áp đủ 4 điều kiện của BR-03: `visibility = 'public'`, `status = 'active'`, `deleted_at IS NULL`, tác giả `status = 'active'`. Kết quả chỉ có tên hiển thị và ảnh đại diện của tác giả, không có email (BR-12).

## US-EXP-001: Xem bản đồ ký ức công khai

> **Là một khách du lịch chưa đăng nhập đang tìm cảnh đẹp để đi trải nghiệm, Tôi muốn xem các ghim ký ức công khai của cộng đồng trên bản đồ, Để chọn được điểm đến cho chuyến đi sắp tới.**

| **Epic** | **EP3** | **Ưu tiên** | **Must** |
|---|---|---|---|
| FR liên quan | FR-2.1, FR-3.5, FR-3.8 | Trạng thái | ✅ Hoàn thành |
| Phụ thuộc | Không có | Kiểm thử | `tests/test_us_exp_001_004.py`; tích hợp 4/4 đạt |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (4 AC).

### Tiêu chí nghiệm thu (4)

#### AC1: Xem ghim công khai

| **Given** | **có 20 ký ức công khai trong vùng đang xem và 5 ký ức riêng tư của người khác** |
|---|---|
| When | khách mở bản đồ |
| Then | bản đồ hiển thị 20 ghim công khai |
| And | không hiển thị ký ức riêng tư |

#### AC2: Vùng chưa có ký ức

| **Given** | **vùng đang xem chưa có ký ức công khai nào** |
|---|---|
| When | khách di chuyển bản đồ tới vùng đó (với tùy chọn "Chỉ trong vùng bản đồ đang xem") |
| Then | hệ thống hiển thị "Chưa có ký ức công khai trong khu vực này" |

#### AC3: Thông tin tác giả

| **Given** | **khách đang xem chi tiết một ký ức công khai** |
|---|---|
| When | khách xem phần thông tin tác giả |
| Then | chỉ hiển thị tên hiển thị (và ảnh đại diện nếu có) |
| And | không hiển thị email của tác giả |

#### AC4: Ký ức bị quản trị viên ẩn

| **Given** | **một ký ức công khai đã bị quản trị viên ẩn** |
|---|---|
| When | khách mở bản đồ |
| Then | ký ức đó không xuất hiện |

### Đặc tả cài đặt

- Ghim: `GET /api/explore/pins?south=&west=&north=&east=` (+ bộ lọc `q`, `hashtags`, `region`). Chỉ trả ghim trong khung đang xem, tối đa 500, sắp ngày ký ức mới trước. Mỗi ghim gồm id, tiêu đề, địa điểm, ngày, tọa độ, tên tác giả. Khung vượt kinh tuyến 180° (tây > đông) được xử lý đúng; nam > bắc trả 422 "Vùng bản đồ không hợp lệ".
- Danh sách: `GET /api/explore/memories` (cùng bộ lọc, `page`, `page_size` mặc định 12, tối đa 50). Trả `items`, `total`, `page`, `page_size`, `pages`. Không có kết quả thì `message` cho biết lý do: theo từ khóa, theo hashtag, hoặc "Chưa có ký ức công khai trong khu vực này".
- Tổng quan: `GET /api/explore/overview` trả số ký ức công khai và số tỉnh/thành có ký ức (dùng ở trang mở đầu và panel Khám phá).
- Bản đồ tải lại ghim 250 ms sau khi người dùng ngừng kéo/thu phóng. Người đã đăng nhập thấy thêm lớp ghim xanh của mình (`GET /api/memories/mine/pins`, tối đa 1000); hai lớp bật/tắt riêng.
- Chi tiết ghim công khai mở bằng API của US-MEM-005, không cần đăng nhập.

## US-EXP-002: Lọc ký ức công khai theo hashtag

> **Là một bạn trẻ thích đi phượt, muốn tìm ký ức theo chủ đề, Tôi muốn lọc ký ức công khai theo hashtag, Để chỉ xem những ký ức đúng chủ đề mình quan tâm.**

| **Epic** | **EP3** | **Ưu tiên** | **Must** |
|---|---|---|---|
| FR liên quan | FR-3.1, FR-3.4, FR-3.8 | Trạng thái | ✅ Hoàn thành |
| Phụ thuộc | US-EXP-001 | Kiểm thử | `tests/test_us_exp_002_hashtags.py`; tích hợp 4/4 đạt |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (4 AC).

### Tiêu chí nghiệm thu (4)

#### AC1: Lọc theo một hashtag

| **Given** | **có 8 ký ức công khai gắn "dalat" và 12 ký ức khác** |
|---|---|
| When | người dùng chọn hashtag "dalat" |
| Then | bản đồ và danh sách chỉ hiển thị 8 ký ức gắn "dalat" |

#### AC2: Chọn nhiều hashtag

| **Given** | **có ký ức gắn "dalat" và ký ức gắn "phuot"** |
|---|---|
| When | người dùng chọn cả hai hashtag |
| Then | hiển thị các ký ức gắn ít nhất một trong hai hashtag |

#### AC3: Hashtag chưa có ký ức

| **Given** | **chưa có ký ức công khai nào gắn "bienxanh"** |
|---|---|
| When | người dùng chọn hashtag "bienxanh" |
| Then | hệ thống hiển thị "Chưa có ký ức nào với hashtag này" |

#### AC4: Bỏ lọc

| **Given** | **người dùng đang lọc theo "dalat"** |
|---|---|
| When | người dùng bỏ chọn hashtag (hoặc bấm "Bỏ lọc") |
| Then | bản đồ hiển thị lại toàn bộ ký ức công khai trong vùng đang xem |

### Đặc tả cài đặt

- Tham số `hashtags` của `/api/explore/pins` và `/api/explore/memories`: nhiều hashtag cách nhau bởi khoảng trắng hoặc dấu phẩy, có hoặc không có "#", không phân biệt hoa thường; phép OR giữa các hashtag (BR-16).
- Hashtag phổ biến: `GET /api/explore/hashtags?limit=` (mặc định 20, tối đa 100; panel dùng 12), đếm số ký ức công khai mỗi hashtag, nhiều nhất lên trước. Ô "Thêm" cho nhập hashtag không có trong danh sách phổ biến.
- Nhấp một hashtag trong chi tiết ký ức cũng mở panel Khám phá lọc theo hashtag đó.
- Bộ lọc áp dụng đồng thời cho danh sách và ghim trên bản đồ.

## US-EXP-003: Lọc ký ức công khai theo khu vực

> **Là một khách du lịch muốn xem ký ức tại một tỉnh thành cụ thể, Tôi muốn lọc ký ức công khai theo khu vực, Để tìm nhanh các địa điểm quanh nơi mình sắp đến.**

| **Epic** | **EP3** | **Ưu tiên** | **Must** |
|---|---|---|---|
| FR liên quan | FR-3.1, FR-2.13 | Trạng thái | ✅ Hoàn thành |
| Phụ thuộc | US-EXP-001 | Kiểm thử | `tests/test_us_exp_003_regions.py`, `tests/memories/test_region_assignment_and_media.py`; tích hợp 4/4 đạt |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (4 AC).

### Tiêu chí nghiệm thu (4)

#### AC1: Lọc theo khu vực

| **Given** | **có 6 ký ức công khai thuộc "Lâm Đồng" và 10 ký ức ở nơi khác** |
|---|---|
| When | người dùng chọn khu vực "Lâm Đồng" |
| Then | chỉ hiển thị 6 ký ức thuộc "Lâm Đồng" |

#### AC2: Ký ức chưa gắn khu vực

| **Given** | **có ký ức công khai không thuộc khu vực nào** |
|---|---|
| When | người dùng lọc theo một khu vực bất kỳ |
| Then | ký ức đó không xuất hiện trong kết quả lọc |
| And | ký ức đó vẫn xuất hiện khi không lọc khu vực |

#### AC3: Kết hợp khu vực và hashtag

| **Given** | **có ký ức "Lâm Đồng" gắn "phuot" và ký ức "Lâm Đồng" không gắn hashtag nào** |
|---|---|
| When | người dùng chọn khu vực "Lâm Đồng" và hashtag "phuot" |
| Then | chỉ hiển thị ký ức thỏa cả khu vực và hashtag |

#### AC4: Khu vực chưa có ký ức

| **Given** | **khu vực "Cao Bằng" chưa có ký ức công khai** |
|---|---|
| When | người dùng chọn "Cao Bằng" |
| Then | hệ thống hiển thị "Chưa có ký ức công khai trong khu vực này" |

### Đặc tả cài đặt

- Danh mục: `GET /api/explore/regions` trả 34 tỉnh/thành (sắp theo tên) kèm `level` (`province`/`city`) và số ký ức công khai của mỗi nơi. Ô chọn mặc định "Tất cả 34 tỉnh/thành".
- Tham số `region` của API khám phá nhận id hoặc slug (ví dụ `lam-dong`).
- Gán khu vực (BR-25) theo **tọa độ**, không theo tên Nominatim trả về (dữ liệu OSM có thể chưa cập nhật sắp xếp tỉnh năm 2025). Thuật toán (`explore/region_lookup.py`, Python thuần, không dùng shapely): lọc tỉnh theo khung bao, kiểm điểm-trong-đa-giác bằng ray casting có xét lỗ; không thuộc tỉnh nào thì lấy tỉnh có đường biên gần nhất trong 0,05° (khoảng 5 km) để ghim sát bờ biển hoặc biên giới vẫn được gán; xa hơn thì không gắn khu vực.
- Dữ liệu ranh giới: `app/modules/explore/data/provinces.geojson` (34 tỉnh/thành, khoảng 200 KB). Ký ức tạo trước khi có tính năng này được gán bù bằng `scripts/gan_khu_vuc.py` (giữ nguyên `updated_at`).

## US-EXP-004: Tìm kiếm ký ức công khai theo từ khóa

> **Là một bạn trẻ muốn tìm nhanh ký ức về một địa danh cụ thể, Tôi muốn tìm ký ức công khai bằng từ khóa, Để đi thẳng tới nội dung mình cần mà không phải duyệt bản đồ.**

| **Epic** | **EP3** | **Ưu tiên** | **Should** |
|---|---|---|---|
| FR liên quan | FR-3.2 | Trạng thái | ✅ Hoàn thành |
| Phụ thuộc | US-EXP-001 | Kiểm thử | `tests/test_us_exp_001_004.py`; tích hợp 4/4 đạt (FULLTEXT trên MySQL thật) |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (4 AC).

### Tiêu chí nghiệm thu (4)

#### AC1: Tìm theo địa danh

| **Given** | **có 5 ký ức công khai có tiêu đề hoặc tên địa điểm chứa "Hồ Gươm"** |
|---|---|
| When | người dùng tìm "Hồ Gươm" |
| Then | hiển thị 5 ký ức, ký ức mới nhất ở đầu danh sách |

#### AC2: Không phân biệt hoa thường

| **Given** | **có ký ức tên địa điểm "Hồ Gươm"** |
|---|---|
| When | người dùng tìm "hồ gươm" (hoặc "ho guom") |
| Then | kết quả giống khi tìm "Hồ Gươm" |

#### AC3: Từ khóa quá ngắn

| **Given** | **người dùng đang ở ô tìm kiếm** |
|---|---|
| When | người dùng nhập 1 ký tự và tìm |
| Then | hệ thống hiển thị "Nhập ít nhất 2 ký tự" |

#### AC4: Không có kết quả

| **Given** | **không ký ức công khai nào khớp "xyzxyz"** |
|---|---|
| When | người dùng tìm "xyzxyz" |
| Then | hệ thống hiển thị "Không tìm thấy kết quả cho xyzxyz" |

### Đặc tả cài đặt

- Tham số `q` (2–100 ký tự, chuẩn NFC; quá dài: "Từ khóa tối đa 100 ký tự") của `/api/explore/pins` và `/api/explore/memories`.
- Trên MySQL: chỉ mục FULLTEXT `ft_mem_search (title, place_name) WITH PARSER ngram`, tìm ở BOOLEAN MODE với cả cụm từ trong ngoặc kép (tránh NATURAL MODE trả quá nhiều kết quả không liên quan). Collation `utf8mb4_0900_ai_ci` nên không phân biệt dấu và hoa thường.
- Sắp xếp: điểm liên quan giảm dần, rồi `memory_date` giảm dần ("mới nhất" là ngày của kỷ niệm, không phải ngày tạo bản ghi), rồi id giảm dần.
- Trên SQLite (môi trường test) dùng `LIKE` thay cho FULLTEXT; vì vậy tìm không dấu chỉ kiểm được trên MySQL (kiểm thử tích hợp).
- Tìm kiếm không xét nội dung (`content`) của ký ức công khai; ô tìm trong panel Của tôi thì có xét nội dung.

## US-EXP-005: Lọc ký ức cá nhân theo thời gian

> **Là một người dùng đã đăng nhập muốn xem lại hành trình của mình theo từng năm, Tôi muốn lọc ký ức của mình theo năm, tháng hoặc khoảng ngày, Để xem lại kỷ niệm của một giai đoạn cụ thể.**

| **Epic** | **EP3** (cài đặt ở M2) | **Ưu tiên** | **Must** |
|---|---|---|---|
| FR liên quan | FR-3.3 | Trạng thái | ✅ Hoàn thành |
| Phụ thuộc | US-MEM-001 | Kiểm thử | `tests/test_us_exp_005_mine.py`; tích hợp 4/4 đạt |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (4 AC).

### Tiêu chí nghiệm thu (4)

#### AC1: Lọc theo năm

| **Given** | **người dùng có 4 ký ức năm 2024 và 3 ký ức năm 2025** |
|---|---|
| When | người dùng chọn năm 2024 |
| Then | chỉ hiển thị 4 ký ức năm 2024, ngày cũ trước ngày mới |
| And | bao gồm cả ký ức riêng tư và công khai |

#### AC2: Khoảng ngày gồm cả hai đầu

| **Given** | **người dùng có ký ức ngày 01/03/2025 và 31/03/2025** |
|---|---|
| When | người dùng chọn từ 01/03/2025 đến 31/03/2025 |
| Then | cả hai ký ức đều xuất hiện |

#### AC3: Ngày bắt đầu sau ngày kết thúc

| **Given** | **người dùng đang chọn khoảng ngày** |
|---|---|
| When | người dùng chọn từ 31/03/2025 đến 01/03/2025 |
| Then | hệ thống hiển thị "Ngày bắt đầu phải trước hoặc bằng ngày kết thúc" |

#### AC4: Không có ký ức trong khoảng

| **Given** | **người dùng không có ký ức nào năm 2020** |
|---|---|
| When | người dùng chọn năm 2020 |
| Then | hệ thống hiển thị "Bạn chưa có ký ức nào trong khoảng thời gian này" |

### Đặc tả cài đặt

- API (cần đăng nhập): `GET /api/memories/mine` (danh sách phân trang, `page_size` mặc định 20, tối đa 50; `order=asc|desc`); `GET /api/memories/mine/pins` (ghim xanh, tối đa 1000); `GET /api/memories/mine/years` (các năm có ký ức, mới trước, để dựng ô chọn năm).
- Bộ lọc dùng chung cho danh sách và ghim: `year` (1900–2100), `month` (1–12), `date_from`, `date_to` (gồm cả hai đầu), `visibility` (`private`/`public`), `hashtags` (OR), `q` (≤ 100 ký tự, tìm trong tiêu đề, địa điểm, nội dung).
- Thứ tự: khi có lọc thời gian, giao diện xin `order=asc` (ngày cũ trước, AC1); không lọc thời gian thì mới trước. Cùng ngày xếp theo thời điểm tạo, rồi id.
- Không có kết quả mà không lọc thời gian: "Không có ký ức nào phù hợp bộ lọc".
- Trang hồ sơ dùng lại API này để hiện ký ức theo năm.
