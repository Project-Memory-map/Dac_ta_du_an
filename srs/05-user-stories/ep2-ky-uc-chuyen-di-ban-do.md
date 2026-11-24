[← Danh sách epic](README.md)

# 5.4 Epic EP2: Ký ức, chuyến đi và bản đồ

Module M2 (`app/modules/memories/`). 10 story, 41 tiêu chí nghiệm thu. Kết quả kiểm thử tích hợp: 41/41 đạt. Lưu ý: AC của US-MEM-010 đạt khi gọi API hashtag trực tiếp; form ký ức trên giao diện chưa lưu hashtag (mục 11.3, vấn đề số 1).

Quy tắc quyền xem dùng chung cho mọi API đọc ký ức, ảnh, video, hashtag (BR-03, BR-04): chủ ký ức luôn xem được ký ức của mình (kể cả khi bị ẩn); người khác chỉ xem được khi ký ức **công khai**, **không bị ẩn**, **chưa xóa** và chủ ký ức **không bị khóa**. Không thỏa thì trả 404 "Không tìm thấy ký ức", không tiết lộ ký ức có tồn tại.

## US-MEM-001: Tạo ký ức bằng cách tìm địa điểm

> **Là một người dùng đã đăng nhập vừa đi du lịch về, muốn ghi lại kỷ niệm tại một địa điểm có tên, Tôi muốn nhập tên địa điểm rồi tạo ký ức với tiêu đề, nội dung và ngày, Để kỷ niệm được gắn đúng vị trí trên bản đồ để dễ tìm lại.**

| **Epic** | **EP2** | **Ưu tiên** | **Must** |
|---|---|---|---|
| FR liên quan | FR-2.2, FR-2.3 | Trạng thái | ✅ Hoàn thành |
| Phụ thuộc | US-ACC-002 | Kiểm thử | `tests/memories/test_us_mem_001_geocode_search.py` (8 ca); tích hợp 5/5 đạt (gọi Nominatim thật) |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (5 AC).

### Tiêu chí nghiệm thu (5)

#### AC1: Tạo ký ức bằng địa điểm tìm được

| **Given** | **người dùng đang ở bản đồ của mình** |
|---|---|
| When | người dùng nhập "Hồ Gươm", nhấn Enter, chọn kết quả đầu tiên, điền tiêu đề "Chiều thu Hà Nội", nội dung, ngày 12/10/2025 rồi lưu |
| Then | ghim xuất hiện đúng vị trí Hồ Gươm trên bản đồ |
| And | ký ức ở chế độ riêng tư |
| And | ngày hiển thị là 12/10/2025 |

#### AC2: Câu chuyện dài

| **Given** | **người dùng đang tạo ký ức** |
|---|---|
| When | người dùng nhập nội dung dài 20.000 ký tự và lưu |
| Then | hệ thống lưu nguyên vẹn, không cắt bớt nội dung |

#### AC3: Không tìm thấy địa điểm

| **Given** | **người dùng đang tạo ký ức** |
|---|---|
| When | người dùng nhập tên địa điểm "xyzxyz" không có kết quả |
| Then | hệ thống hiển thị "Không tìm thấy địa điểm. Hãy thử tên khác hoặc chọn trực tiếp trên bản đồ" |
| And | không có ghim nào được tạo |

#### AC4: Thiếu tiêu đề

| **Given** | **người dùng đã chọn địa điểm** |
|---|---|
| When | người dùng để trống tiêu đề và lưu |
| Then | hệ thống hiển thị "Vui lòng nhập tiêu đề" |
| And | ký ức không được lưu |

#### AC5: Ngày trong tương lai

| **Given** | **hôm nay là 02/11/2026** |
|---|---|
| When | người dùng chọn ngày 03/11/2026 và lưu |
| Then | hệ thống hiển thị "Ngày không được ở tương lai" |
| And | ký ức không được lưu |

### Đặc tả cài đặt

- Tìm địa điểm: `GET /api/geocode?q=` (không cần đăng nhập). Có ở thanh tìm trên cùng của bản đồ và trong form ký ức. Chỉ gọi khi nhấn Enter hoặc nút "Tìm", không gọi khi đang gõ (BR-19).
- Từ khóa 2–200 ký tự ("Nhập tên địa điểm từ 2 đến 200 ký tự"). Gọi Nominatim `/search` (`format=jsonv2`, `limit=5`, `accept-language=vi`), trả tối đa 5 kết quả `{name, display_name, latitude, longitude}`. Kết quả được cache 24 giờ (tối đa 1000 từ khóa).
- Nominatim lỗi mạng: 503 "Không kết nối được dịch vụ tìm địa điểm. Hãy thử lại hoặc chọn trực tiếp trên bản đồ". Nominatim từ chối vì quá tải: 503 "Dịch vụ địa điểm đang giới hạn yêu cầu. Vui lòng thử lại sau." Hàng chờ nội bộ quá 5 giây: 503 "Nhiều người đang tìm địa điểm cùng lúc, vui lòng thử lại sau vài giây".
- Tạo ký ức: `POST /api/memories` (multipart form: `title`, `place_name`, `latitude`, `longitude`, `memory_date`, `content`, `visibility`). Kiểm theo thứ tự: tọa độ ("Vui lòng chọn vị trí trên bản đồ hoặc tìm địa điểm", "Tọa độ không hợp lệ"), tên địa điểm ("Vui lòng nhập tên địa điểm"), tiêu đề (bắt buộc, ≤ 200 ký tự: "Tiêu đề tối đa 200 ký tự"), ngày ("Vui lòng chọn ngày", "Ngày không được ở tương lai"), chế độ ("Chế độ hiển thị không hợp lệ"). Tọa độ làm tròn 6 chữ số thập phân. Thành công trả 201 "Đã lưu ký ức".
- Giao diện cũng kiểm tiêu đề, ngày, số hashtag, số ảnh trước khi gửi; máy chủ kiểm lại; trigger MySQL `trg_memories_bi` chặn lần thứ ba ngày ở tương lai.
- Khi tạo, hệ thống tự gán tỉnh/thành theo tọa độ (BR-25). Ảnh và video gửi sau, bằng API riêng của US-MEM-003, US-MEM-004.

## US-MEM-002: Tạo ký ức bằng cách chọn vị trí trên bản đồ

> **Là một người dùng đã đăng nhập vừa đến một nơi không có tên trong danh mục địa điểm (một con suối, một đoạn đèo), Tôi muốn chọn trực tiếp vị trí trên bản đồ để tạo ký ức, Để ghim được cả những nơi tìm không ra bằng tên.**

| **Epic** | **EP2** | **Ưu tiên** | **Must** |
|---|---|---|---|
| FR liên quan | FR-2.2 | Trạng thái | ✅ Hoàn thành |
| Phụ thuộc | US-ACC-002 | Kiểm thử | `tests/test_us_mem_002_pick_on_map.py`; tích hợp 3/3 đạt (máy tính nhấp đúp, điện thoại giả lập chạm giữ) |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (3 AC).

### Tiêu chí nghiệm thu (3)

#### AC1: Chọn vị trí rồi tạo ký ức

| **Given** | **người dùng đang xem bản đồ** |
|---|---|
| When | người dùng nhấp đúp (hoặc nhấn giữ trên điện thoại) vào một vị trí và điền tiêu đề, nội dung, ngày rồi lưu |
| Then | ghim xuất hiện đúng vị trí đã chọn |
| And | ký ức ở chế độ riêng tư |

#### AC2: Vị trí ngoài danh mục khu vực

| **Given** | **người dùng chọn vị trí ở nước ngoài, ví dụ Paris** |
|---|---|
| When | người dùng lưu ký ức |
| Then | ký ức vẫn được tạo |
| And | ký ức không gắn khu vực nào |

#### AC3: Khách chưa đăng nhập

| **Given** | **khách chưa đăng nhập đang xem bản đồ** |
|---|---|
| When | khách nhấp đúp vào một vị trí |
| Then | hệ thống hiện hộp thoại "Cần đăng nhập" với lựa chọn "Đăng nhập" hoặc "Để sau" |
| And | không có ghim nào được tạo |

### Đặc tả cài đặt

- Cách chọn vị trí: nhấp đúp hoặc nhấp chuột phải (máy tính); nhấn giữ một ngón 600 ms (điện thoại; di chuyển ngón tay hoặc chạm hai ngón thì hủy); hoặc nút "🎯 Chọn trên bản đồ" trong form rồi nhấp một điểm.
- Sau khi chọn, hiện ghim tím kéo được; kéo xong thì cập nhật tọa độ. Sau khi chạm giữ đã kích hoạt, sự kiện `touchend` bị chặn để cú nhấp do trình duyệt tự sinh không đóng hộp thoại (lỗi L09 đã sửa).
- Tên địa điểm tự điền bằng `GET /api/geocode/reverse?lat=&lng=` (Nominatim `/reverse`, cache theo tọa độ làm tròn 4 chữ số, tối đa 512 mục, chung cổng 1 yêu cầu/giây). Không có kết quả: "Không tìm thấy tên địa điểm cho vị trí này"; lỗi mạng: 503 "Không lấy được tên địa điểm, vui lòng tự nhập". Hai trường hợp này người dùng tự nhập tên trước khi lưu.
- Khách: hộp thoại "Cần đăng nhập"; chọn "Đăng nhập" thì sang `/accounts/login.html?next=/ban-do`. API `POST /api/memories` không có JWT trả 401.
- Vị trí ngoài Việt Nam (xa ranh giới tỉnh gần nhất quá khoảng 5 km): `region_id = NULL`.

## US-MEM-003: Đính kèm ảnh vào ký ức

> **Là một người dùng đã đăng nhập đang tạo hoặc sửa một ký ức, Tôi muốn đính kèm nhiều ảnh vào ký ức, Để kỷ niệm có hình ảnh sinh động khi xem lại.**

| **Epic** | **EP2** | **Ưu tiên** | **Must** |
|---|---|---|---|
| FR liên quan | FR-2.4, FR-2.10 | Trạng thái | ✅ Hoàn thành |
| Phụ thuộc | US-MEM-001 | Kiểm thử | `tests/test_us_mem_003_images.py`; tích hợp 5/5 đạt |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (5 AC).

### Tiêu chí nghiệm thu (5)

#### AC1: Đính kèm 3 ảnh

| **Given** | **người dùng đang tạo ký ức** |
|---|---|
| When | người dùng chọn 3 ảnh .jpg, mỗi ảnh 2 MB, và lưu |
| Then | cả 3 ảnh được lưu theo thứ tự đã chọn |
| And | chi tiết ký ức hiển thị đủ 3 ảnh |

#### AC2: Đúng giới hạn tối đa

| **Given** | **người dùng đang tạo ký ức** |
|---|---|
| When | người dùng chọn 10 ảnh, mỗi ảnh đúng 5 MB |
| Then | hệ thống chấp nhận cả 10 ảnh |

#### AC3: Vượt số lượng ảnh

| **Given** | **ký ức đã có 10 ảnh** |
|---|---|
| When | người dùng thêm ảnh thứ 11 |
| Then | hệ thống từ chối ảnh thứ 11 |
| And | hệ thống hiển thị "Mỗi ký ức tối đa 10 ảnh" |

#### AC4: Ảnh vượt dung lượng

| **Given** | **người dùng đang chọn ảnh** |
|---|---|
| When | người dùng chọn ảnh 6 MB |
| Then | hệ thống từ chối ảnh đó |
| And | hệ thống hiển thị "Mỗi ảnh không được vượt quá 5 MB" |

#### AC5: Định dạng không hỗ trợ

| **Given** | **người dùng đang chọn ảnh** |
|---|---|
| When | người dùng chọn tệp "setup.exe" |
| Then | hệ thống từ chối tệp |
| And | hệ thống hiển thị "Định dạng tệp không được hỗ trợ" |

### Đặc tả cài đặt

- API: `GET /api/memories/{id}/images` (theo quyền xem); `POST /api/memories/{id}/images` (chủ ký ức; multipart: `images[]` ảnh mới, `remove_media_ids` id ảnh cần bỏ, `image_order` thứ tự id ảnh giữ lại). Một lần gọi vừa thêm, vừa bỏ, vừa sắp xếp; số ảnh sau cùng không được quá 10.
- Định dạng nhận: .jpg, .jpeg, .png, .webp, .gif. Một tệp hợp lệ phải khớp cả ba: đuôi tệp, kiểu MIME trình duyệt gửi, và chữ ký nhị phân đầu tệp (ví dụ JPEG bắt đầu `FF D8 FF`). **Không nhận .heic** (khác bản 0.3, vì đa số trình duyệt không hiển thị được HEIC).
- Trước khi lưu, xóa siêu dữ liệu bằng bộ lọc Python thuần (không nén lại ảnh): JPEG bỏ APP1 (EXIF/XMP), APP3–APP15, COM, giữ APP2 (hồ sơ màu ICC); PNG bỏ `eXIf`, `tEXt`, `zTXt`, `iTXt`, `tIME`; WebP bỏ `EXIF`, `XMP`. Ảnh GIF và tệp có cấu trúc lạ được giữ nguyên.
- Lưu ở `uploads/images/{uuid}.{đuôi}`. Lỗi giữa chừng thì xóa các tệp đã ghi và hoàn tác CSDL. Trigger `trg_media_bi` chặn thêm lần nữa ở CSDL (10 ảnh, 5 MB).
- Ảnh chỉ mở được qua URL có chữ ký (BR-17). Chưa có ảnh thu nhỏ: `thumb_url` trả cùng URL với ảnh gốc (xem NFR-P4).
- Giao diện: ô chọn tệp lọc sẵn định dạng, đếm "x/10", cho xem trước, bỏ và kéo đổi thứ tự trước khi lưu, có thanh tiến trình tải lên.

## US-MEM-004: Đính kèm video vào ký ức

> **Là một người dùng đã đăng nhập có một đoạn video ngắn ghi lại khoảnh khắc đáng nhớ, Tôi muốn đính kèm một video vào ký ức, Để giữ lại cả động tác và âm thanh của khoảnh khắc đó.**

| **Epic** | **EP2** | **Ưu tiên** | **Should** |
|---|---|---|---|
| FR liên quan | FR-2.4, FR-2.10 | Trạng thái | ✅ Hoàn thành |
| Phụ thuộc | US-MEM-001 | Kiểm thử | `tests/test_us_mem_004_video.py`; tích hợp 4/4 đạt |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (4 AC).

### Tiêu chí nghiệm thu (4)

#### AC1: Đính kèm video hợp lệ

| **Given** | **người dùng đang tạo ký ức** |
|---|---|
| When | người dùng chọn video .mp4 dung lượng 30 MB và lưu |
| Then | video được lưu |
| And | chi tiết ký ức hiển thị video và phát được |

#### AC2: Đúng dung lượng tối đa

| **Given** | **người dùng đang tạo ký ức** |
|---|---|
| When | người dùng chọn video đúng 50 MB |
| Then | hệ thống chấp nhận video |

#### AC3: Video vượt dung lượng

| **Given** | **người dùng đang chọn video** |
|---|---|
| When | người dùng chọn video 60 MB |
| Then | hệ thống từ chối video |
| And | hệ thống hiển thị "Video không được vượt quá 50 MB" |

#### AC4: Thêm video thứ hai

| **Given** | **ký ức đã có 1 video** |
|---|---|
| When | người dùng thêm video thứ hai (không gỡ video cũ) |
| Then | hệ thống từ chối |
| And | hệ thống hiển thị "Mỗi ký ức chỉ có 1 video" |

### Đặc tả cài đặt

- API: `GET /api/memories/{id}/video`; `POST /api/memories/{id}/video` (multipart: `video`, `remove_video=true` để gỡ hoặc thay video cũ); `GET /api/media/{id}/stream?exp=&sig=` để phát.
- Định dạng: .mp4 (`video/mp4`), .mov (`video/quicktime`, video quay bằng iPhone); kiểm đuôi và chữ ký nhị phân; ≤ 50 MB, đọc theo từng khối để dừng ngay khi vượt dung lượng. Lưu ở `uploads/videos/`.
- Phát video hỗ trợ header `Range` (trả 206 và `Content-Range`; khoảng không hợp lệ trả 416), nên tua được và không phải tải hết tệp. Ký ức công khai phát thẳng từ URL có chữ ký; ký ức riêng tư tải qua blob vì phải kèm header đăng nhập.
- Chữ ký URL video gồm cả id và đường dẫn tệp, nên thay video thì link cũ hết tác dụng.
- Lưu trữ khi triển khai thật (Cloudflare R2 hoặc tương đương) chưa làm; xem mục 10.

## US-MEM-005: Xem chi tiết một ký ức

> **Là một người dùng đã đăng nhập đang xem bản đồ, Tôi muốn nhấp vào một ghim để xem chi tiết ký ức, Để đọc lại câu chuyện và xem ảnh, video của chuyến đi.**

| **Epic** | **EP2** | **Ưu tiên** | **Must** |
|---|---|---|---|
| FR liên quan | FR-2.8 | Trạng thái | ✅ Hoàn thành |
| Phụ thuộc | US-MEM-001 | Kiểm thử | `tests/memories/test_us_mem_008_privacy.py`, `tests/test_us_exp_001_004.py`; tích hợp 4/4 đạt |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (4 AC).

### Tiêu chí nghiệm thu (4)

#### AC1: Xem ký ức của chính mình

| **Given** | **người dùng có ký ức "Chiều thu Hà Nội" ngày 12/10/2025 kèm 3 ảnh** |
|---|---|
| When | người dùng nhấp vào ghim của ký ức đó |
| Then | chi tiết hiển thị tiêu đề, ngày 12/10/2025, tên địa điểm, nội dung, 3 ảnh và hashtag |
| And | người dùng thấy lựa chọn sửa và xóa |

#### AC2: Xem ký ức công khai của người khác

| **Given** | **một người dùng khác có ký ức công khai** |
|---|---|
| When | người dùng nhấp vào ghim của ký ức đó |
| Then | người dùng xem được nội dung và tên hiển thị của tác giả |
| And | không có lựa chọn sửa hay xóa |

#### AC3: Mở ký ức riêng tư của người khác

| **Given** | **ký ức số 123 là riêng tư và thuộc người khác** |
|---|---|
| When | người dùng mở trực tiếp địa chỉ của ký ức số 123 |
| Then | hệ thống hiển thị "Không tìm thấy ký ức" |
| And | không tiết lộ ký ức đó có tồn tại |

#### AC4: Ký ức đã bị quản trị viên ẩn

| **Given** | **ký ức của người dùng đã bị quản trị viên ẩn** |
|---|---|
| When | chủ ký ức mở ký ức đó |
| Then | chủ ký ức thấy trạng thái "Đã bị quản trị viên ẩn" |
| And | người dùng khác không thấy ký ức đó |

### Đặc tả cài đặt

- API: `GET /api/memories/{id}` (không bắt buộc đăng nhập). Trả: tiêu đề, nội dung, ngày, địa điểm, tọa độ, chế độ, `status`, `hidden_by_admin`, `is_owner`, tỉnh/thành, tác giả (id, tên hiển thị, `avatar_url`; không có email), hashtag kèm nguồn (`manual`/`ai`), danh sách media có URL ký sẵn, danh sách chuyến đi chứa ký ức (chỉ trả cho chủ), thời điểm tạo và cập nhật.
- Giao diện: nhấp ghim mở popup tóm tắt, bấm vào mở ngăn kéo chi tiết. Ký ức của mình có nút Sửa, Xóa, đổi Riêng tư/Công khai. Hashtag do AI gợi ý có dấu ✦; nhấp hashtag thì lọc ký ức công khai theo hashtag đó. Quản trị viên xem ký ức công khai của người khác có thêm nút Ẩn/Bỏ ẩn/Xóa (FR-4.13).
- Mọi dữ liệu người dùng nhập được đưa vào giao diện bằng `textContent` (BR-21).

## US-MEM-006: Sửa nội dung ký ức

> **Là một người dùng đã đăng nhập muốn bổ sung chi tiết cho một kỷ niệm đã ghi, Tôi muốn sửa tiêu đề, nội dung, địa điểm, ngày, ảnh và video của ký ức, Để ký ức luôn đúng và đầy đủ theo thời gian.**

| **Epic** | **EP2** | **Ưu tiên** | **Must** |
|---|---|---|---|
| FR liên quan | FR-2.5, FR-2.10 | Trạng thái | ✅ Hoàn thành (⚠️ sửa hashtag qua form: mục 11.3) |
| Phụ thuộc | US-MEM-001 | Kiểm thử | `tests/memories/test_region_assignment_and_media.py`; tích hợp 4/4 đạt |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (4 AC).

### Tiêu chí nghiệm thu (4)

#### AC1: Sửa tiêu đề và ảnh

| **Given** | **người dùng mở ký ức của mình có 3 ảnh** |
|---|---|
| When | người dùng đổi tiêu đề, thêm 1 ảnh, bỏ 1 ảnh rồi lưu |
| Then | chi tiết ký ức hiển thị tiêu đề mới và 3 ảnh |
| And | thời điểm cập nhật được ghi lại |

#### AC2: Đổi địa điểm

| **Given** | **người dùng mở ký ức của mình tại Hà Nội** |
|---|---|
| When | người dùng đổi địa điểm sang "Đà Lạt" và lưu |
| Then | ghim di chuyển tới Đà Lạt |
| And | khu vực của ký ức chuyển theo vị trí mới (Lâm Đồng) |

#### AC3: Sửa ký ức của người khác

| **Given** | **ký ức số 123 thuộc người dùng khác** |
|---|---|
| When | người dùng cố sửa ký ức số 123 |
| Then | hệ thống từ chối |
| And | hệ thống hiển thị "Bạn không có quyền sửa ký ức này" |
| And | nội dung ký ức không đổi |

#### AC4: Xóa trống tiêu đề khi sửa

| **Given** | **người dùng đang sửa ký ức của mình** |
|---|---|
| When | người dùng xóa hết tiêu đề và lưu |
| Then | hệ thống hiển thị "Vui lòng nhập tiêu đề" |
| And | ký ức giữ nguyên nội dung cũ |

### Đặc tả cài đặt

- API: `PATCH /api/memories/{id}` (multipart, chỉ cập nhật trường có gửi: `title`, `content`, `place_name`, `latitude`, `longitude`, `memory_date`, `visibility`). Ảnh và video sửa qua API của US-MEM-003, US-MEM-004.
- Không phải chủ: 403 "Bạn không có quyền sửa ký ức này". Ký ức không có hoặc đã xóa: 404.
- Gửi tọa độ phải đủ cả vĩ độ và kinh độ ("Cần gửi đủ cả vĩ độ và kinh độ"); đổi tọa độ thì tính lại tỉnh/thành.
- `updated_at` do MySQL tự ghi; các phép gán thời gian trong code dùng `func.now()` (giờ CSDL) để không lệch múi giờ với `created_at` (lỗi L14 đã sửa).
- Trigger `trg_memories_bu` chặn ngày ở tương lai và chặn đổi chủ sở hữu ký ức (BR-02).

## US-MEM-007: Xóa ký ức

> **Là một người dùng đã đăng nhập muốn bỏ một ký ức không còn muốn lưu, Tôi muốn xóa ký ức của mình, Để kiểm soát những gì được lưu trong nhật ký.**

| **Epic** | **EP2** | **Ưu tiên** | **Must** |
|---|---|---|---|
| FR liên quan | FR-2.5 | Trạng thái | ✅ Hoàn thành (⏳ dọn tệp vật lý định kỳ chưa làm) |
| Phụ thuộc | US-MEM-001 | Kiểm thử | `tests/memories/test_us_mem_007_delete.py`; tích hợp 4/4 đạt |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (4 AC).

### Tiêu chí nghiệm thu (4)

#### AC1: Xóa sau khi xác nhận

| **Given** | **người dùng mở ký ức của mình có ảnh và hashtag** |
|---|---|
| When | người dùng chọn "Xóa" rồi xác nhận |
| Then | ký ức biến mất khỏi bản đồ và danh sách |
| And | ảnh và video của ký ức không còn truy cập được |

#### AC2: Hủy ở bước xác nhận

| **Given** | **người dùng chọn "Xóa" một ký ức** |
|---|---|
| When | người dùng chọn "Hủy" ở hộp xác nhận |
| Then | ký ức vẫn còn nguyên |

#### AC3: Xóa ký ức thuộc một chuyến đi

| **Given** | **ký ức nằm trong chuyến đi "Đà Lạt tháng 10" cùng 2 ký ức khác** |
|---|---|
| When | người dùng xóa ký ức đó |
| Then | chuyến đi vẫn còn với 2 ký ức còn lại |

#### AC4: Xóa ký ức của người khác

| **Given** | **ký ức số 123 thuộc người dùng khác** |
|---|---|
| When | người dùng cố xóa ký ức số 123 |
| Then | hệ thống từ chối ("Bạn không có quyền xóa ký ức này") |
| And | ký ức số 123 không bị ảnh hưởng |

### Đặc tả cài đặt

- API: `DELETE /api/memories/{id}`. Ghi `deleted_at = NOW()` và xóa các dòng `trip_memories` của ký ức trong cùng giao dịch. Trả "Đã xóa ký ức".
- Hộp xác nhận ghi rõ: tiêu đề ký ức, ảnh và video sẽ bị xóa khỏi bản đồ, danh sách và các chuyến đi, không thể hoàn tác.
- Sau khi xóa mềm, mọi API đọc ký ức, ảnh, video, hashtag đều trả 404; URL ảnh có chữ ký còn hạn cũng bị từ chối vì máy chủ kiểm `deleted_at` ở mỗi lần tải.
- Tệp ảnh/video vẫn nằm trên ổ đĩa. Script dọn tệp của ký ức đã xóa mềm quá 30 ngày (BR-13) **chưa được viết**; xem mục 11.3.

## US-MEM-008: Đổi chế độ riêng tư hoặc công khai của ký ức

> **Là một người dùng đã đăng nhập muốn chia sẻ một số kỷ niệm với cộng đồng và giữ kín số còn lại, Tôi muốn chuyển ký ức giữa chế độ riêng tư và công khai, Để quyết định ai được xem kỷ niệm của mình.**

| **Epic** | **EP2** | **Ưu tiên** | **Must** |
|---|---|---|---|
| FR liên quan | FR-2.6 | Trạng thái | ✅ Hoàn thành |
| Phụ thuộc | US-MEM-001 | Kiểm thử | `tests/memories/test_us_mem_008_privacy.py`; tích hợp 3/3 đạt |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (3 AC).

### Tiêu chí nghiệm thu (3)

#### AC1: Chuyển sang công khai

| **Given** | **ký ức của người dùng đang riêng tư** |
|---|---|
| When | người dùng chuyển sang công khai và xác nhận |
| Then | ký ức xuất hiện trên bản đồ công khai ở lần tải kế tiếp |
| And | khách xem được nội dung và ảnh của ký ức |

#### AC2: Chuyển về riêng tư

| **Given** | **ký ức của người dùng đang công khai** |
|---|---|
| When | người dùng chuyển về riêng tư |
| Then | ký ức biến mất khỏi bản đồ công khai |
| And | người khác mở đường dẫn ảnh của ký ức nhận thông báo "Bạn không có quyền truy cập tệp này" |

#### AC3: Người không phải chủ đổi chế độ

| **Given** | **ký ức số 123 thuộc người dùng khác** |
|---|---|
| When | người dùng cố đổi chế độ của ký ức số 123 |
| Then | hệ thống từ chối ("Bạn không có quyền sửa ký ức này") |
| And | chế độ của ký ức không đổi |

### Đặc tả cài đặt

- API: `PATCH /api/memories/{id}/visibility` `{visibility: "private" | "public"}`. Thông báo "Ký ức đã chuyển sang công khai" hoặc "Ký ức đã chuyển về riêng tư". Giá trị khác: "Chế độ hiển thị không hợp lệ".
- Chuyển sang công khai, giao diện hỏi xác nhận: "Mọi người (kể cả khách chưa đăng nhập) sẽ thấy nội dung, ảnh và video của ký ức này trên bản đồ cộng đồng…". Chuyển về riêng tư không cần xác nhận.
- Form tạo/sửa ký ức cũng có chọn 🔒 Riêng tư / 🌏 Công khai, mặc định riêng tư.
- Ảnh của ký ức riêng tư hoặc bị ẩn trả kèm `Cache-Control: private, no-store` để trình duyệt và proxy không lưu lại.
- Chỉ có hai mức riêng tư; mức "chỉ bạn bè" ngoài phạm vi.

## US-MEM-009: Tạo chuyến đi và thêm ký ức vào chuyến đi

> **Là một người dùng đã đăng nhập vừa đi chuyến "Đà Lạt tháng 10" và ghi nhiều ký ức, Tôi muốn gom các ký ức của một chuyến đi lại với nhau, Để xem lại hành trình theo đúng thứ tự thời gian.**

| **Epic** | **EP2** | **Ưu tiên** | **Must** |
|---|---|---|---|
| FR liên quan | FR-2.7, FR-2.11 | Trạng thái | ✅ Hoàn thành qua panel Chuyến đi (⚠️ ô chọn chuyến đi trong form ký ức: mục 11.3) |
| Phụ thuộc | US-MEM-001 | Kiểm thử | `tests/memories/test_us_mem_009_trips.py`; tích hợp 5/5 đạt |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (5 AC).

### Tiêu chí nghiệm thu (5)

#### AC1: Tạo chuyến đi và thêm ký ức

| **Given** | **người dùng có 3 ký ức ngày 10/10, 11/10, 12/10/2025** |
|---|---|
| When | người dùng tạo chuyến đi "Đà Lạt tháng 10" và thêm cả 3 ký ức |
| Then | chuyến đi hiển thị 3 ký ức theo thứ tự ngày tăng dần |

#### AC2: Hai ký ức cùng ngày

| **Given** | **chuyến đi có hai ký ức cùng ngày 11/10/2025** |
|---|---|
| When | người dùng mở chuyến đi |
| Then | ký ức tạo trước hiển thị trước |

#### AC3: Thêm ký ức của người khác

| **Given** | **ký ức số 123 thuộc người dùng khác** |
|---|---|
| When | người dùng cố thêm ký ức số 123 vào chuyến đi của mình |
| Then | hệ thống từ chối |
| And | hệ thống hiển thị "Chỉ thêm được ký ức của bạn" |

#### AC4: Chuyến đi thiếu tên

| **Given** | **người dùng đang tạo chuyến đi** |
|---|---|
| When | người dùng để trống tên và lưu |
| Then | hệ thống hiển thị "Vui lòng nhập tên chuyến đi" |
| And | chuyến đi không được tạo |

#### AC5: Sửa, xóa chuyến đi và bỏ một ký ức khỏi chuyến đi

| **Given** | **người dùng có chuyến đi "Đà Lạt tháng 10" với 3 ký ức** |
|---|---|
| When | người dùng đổi tên chuyến đi, bỏ một ký ức khỏi chuyến đi (ký ức vẫn còn trong hệ thống), rồi xóa chuyến đi |
| Then | tên chuyến đi được cập nhật trước khi xóa |
| And | ký ức bị bỏ không còn nằm trong chuyến đi nhưng vẫn tồn tại độc lập |
| And | sau khi xóa, chuyến đi không còn xuất hiện nhưng các ký ức thuộc chuyến đi vẫn còn nguyên |

### Đặc tả cài đặt

- API (đều cần đăng nhập, chỉ thao tác chuyến đi của mình, người khác nhận 403 "Bạn không có quyền thao tác trên chuyến đi này"): `GET /api/trips` (mới tạo lên trước); `POST /api/trips` `{name, description?, memory_ids?}`; `GET /api/trips/{id}`; `PATCH`/`PUT /api/trips/{id}`; `DELETE /api/trips/{id}`; `POST /api/trips/{id}/memories` `{memory_ids}`; `DELETE /api/trips/{id}/memories/{memory_id}` ("Ký ức không nằm trong chuyến đi này" nếu không có).
- Tên 1–150 ký tự ("Tên chuyến đi tối đa 150 ký tự"), mô tả ≤ 500 ký tự. Thêm ký ức đã có trong chuyến đi thì bỏ qua, không báo lỗi.
- Dữ liệu trả về: số ký ức, ngày bắt đầu/kết thúc (ngày ký ức sớm nhất/muộn nhất), danh sách ký ức đã xếp (ngày tăng dần, rồi thời điểm tạo, rồi id) kèm ảnh bìa và cờ có video. Ký ức đã xóa mềm không tính.
- Giao diện: mở chuyến đi thì bản đồ thu phóng vừa các ghim và vẽ lộ trình nét đứt màu tím theo thứ tự thời gian.
- Một ký ức thuộc được nhiều chuyến đi (bảng nối `trip_memories`, khóa chính `(trip_id, memory_id)`). Trigger `trg_trip_memories_bi` chặn thêm ký ức của người khác ở CSDL (BR-09).

## US-MEM-010: Gắn hashtag cho ký ức

> **Là một người dùng đã đăng nhập muốn ký ức của mình dễ được tìm thấy theo chủ đề, Tôi muốn gắn các hashtag như \#dalat, \#phuot vào ký ức, Để ký ức xuất hiện khi người khác lọc theo chủ đề.**

| **Epic** | **EP2** | **Ưu tiên** | **Must** |
|---|---|---|---|
| FR liên quan | FR-2.9, FR-3.6 | Trạng thái | ⚠️ API hoàn thành; form ký ức chưa lưu hashtag (mục 11.3) |
| Phụ thuộc | US-MEM-001 | Kiểm thử | `tests/memories/test_us_mem_010_hashtags.py`, `tests/test_fr_3_6_ai_hashtags.py`; tích hợp 4/4 đạt (qua API) |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (4 AC).

### Tiêu chí nghiệm thu (4)

#### AC1: Gắn hai hashtag

| **Given** | **người dùng đang tạo ký ức** |
|---|---|
| When | người dùng nhập "#dalat" và "#phuot" rồi lưu |
| Then | ký ức có hai hashtag "dalat" và "phuot" |
| And | chi tiết ký ức hiển thị cả hai |

#### AC2: Không phân biệt hoa thường

| **Given** | **hashtag "dalat" đã tồn tại** |
|---|---|
| When | người dùng nhập "#DaLat" cho ký ức khác |
| Then | hệ thống dùng chung hashtag "dalat", không tạo hashtag mới |

#### AC3: Hashtag có ký tự không hợp lệ

| **Given** | **người dùng đang nhập hashtag** |
|---|---|
| When | người dùng nhập "#đà lạt!" |
| Then | hệ thống từ chối |
| And | hệ thống hiển thị "Hashtag chỉ gồm chữ, số, dấu gạch dưới (tối đa 30 ký tự)" |

#### AC4: Quá số lượng hashtag

| **Given** | **ký ức đã có 10 hashtag** |
|---|---|
| When | người dùng thêm hashtag thứ 11 |
| Then | hệ thống từ chối |
| And | hệ thống hiển thị "Mỗi ký ức tối đa 10 hashtag" |

### Đặc tả cài đặt

- API: `POST /api/memories/{id}/hashtags` `{hashtags, ai_hashtags?, replace?}` (mặc định thêm vào hashtag đang có; `replace=true` thay toàn bộ); `PUT`/`PATCH /api/memories/{id}/hashtags` (thay toàn bộ); `DELETE /api/memories/{id}/hashtags/{tag}`; `GET /api/memories/{id}/hashtags` (theo quyền xem); `GET /api/hashtags` hoặc `/api/memories/mine/hashtags` (hashtag của tôi kèm số ký ức); `POST /api/memories/suggest-hashtags` (gợi ý, FR-3.6).
- Chuẩn hóa (BR-06): bỏ "#" ở đầu, chuẩn Unicode NFC, chữ thường; chỉ gồm chữ cái (kể cả chữ có dấu tiếng Việt, ví dụ "đàlạt" hợp lệ), chữ số, "_"; không khoảng trắng; 1–30 ký tự. Chuỗi nhập được tách theo khoảng trắng, dấu phẩy, dấu chấm phẩy; hashtag trùng trong cùng lần nhập bị gộp.
- Hashtag `hashtags.name` dùng collation `utf8mb4_0900_as_ci`: không phân biệt hoa thường nhưng **phân biệt dấu** ("đàlạt" và "dalat" là hai hashtag khác nhau).
- Nguồn hashtag: hashtag chọn từ gợi ý AI lưu `source = 'ai'`, còn lại `manual`; chi tiết ký ức đánh dấu hashtag AI bằng ✦.
- Trigger `trg_hashtags_bi` (chuẩn hóa, kiểm ký tự) và `trg_memory_hashtags_bi` (tối đa 10/ký ức) chặn thêm ở CSDL.
- **Vấn đề còn mở**: form tạo/sửa ký ức gửi trường `hashtags` và `ai_hashtags` kèm `POST /api/memories` / `PATCH /api/memories/{id}`, nhưng hai API này không đọc hai trường đó, và giao diện không gọi API hashtag ở trên. Kết quả: hashtag nhập trong form không được lưu. Hướng sửa ở mục 11.3.
