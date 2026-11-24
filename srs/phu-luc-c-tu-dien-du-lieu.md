[← Mục lục SRS](README.md)

# Phụ lục C: Từ điển dữ liệu

Nguồn: [schema.sql](../schema.sql) (bản cuối, trùng `database/schema.sql` trong repo). MySQL 8.0, InnoDB, `utf8mb4_0900_ai_ci`. Khóa chính kiểu `BIGINT UNSIGNED AUTO_INCREMENT`. ERD ở [so-do/03-erd.md](../so-do/03-erd.md).

## C.1 Bảng

### users: tài khoản và hồ sơ (M1)

| Cột | Kiểu | Ràng buộc | Ý nghĩa |
|---|---|---|---|
| id | BIGINT UNSIGNED | PK | |
| email | VARCHAR(255) | NULL, UNIQUE | Chữ thường (trigger chuẩn hóa). NULL được với tài khoản Facebook không có email |
| password_hash | VARCHAR(255) | NULL | Bản băm bcrypt; NULL = chỉ đăng nhập mạng xã hội |
| display_name | VARCHAR(100) | NOT NULL | Tên hiển thị công khai |
| avatar_path | VARCHAR(500) | NULL | Đường dẫn lưu trữ nội bộ, không trả ra API |
| bio | VARCHAR(500) | NULL | Giới thiệu ngắn |
| role | ENUM('user','admin') | DEFAULT 'user' | |
| status | ENUM('active','locked') | DEFAULT 'active' | `locked` = bị quản trị viên khóa |
| email_verified_at | DATETIME | NULL | Dự phòng cho xác thực email (chưa ghi) |
| token_version | INT UNSIGNED | DEFAULT 0 | BR-20 |
| failed_login_count | TINYINT UNSIGNED | DEFAULT 0 | BR-15 |
| first_failed_at | DATETIME | NULL | BR-15 |
| login_locked_until | DATETIME | NULL | BR-15 |
| last_login_at | DATETIME | NULL | |
| created_at, updated_at | DATETIME | MySQL tự ghi | |

### oauth_accounts: liên kết Google/Facebook (M1)

| Cột | Kiểu | Ràng buộc | Ý nghĩa |
|---|---|---|---|
| id | BIGINT UNSIGNED | PK | |
| user_id | BIGINT UNSIGNED | FK → users, CASCADE | |
| provider | ENUM('google','facebook') | | |
| provider_user_id | VARCHAR(191) | UNIQUE (provider, provider_user_id) | Id phía nhà cung cấp |
| created_at | DATETIME | | |

Thêm UNIQUE (user_id, provider): mỗi tài khoản liên kết tối đa một tài khoản của mỗi nhà cung cấp.

### password_reset_tokens: liên kết đặt lại mật khẩu (M1)

| Cột | Kiểu | Ràng buộc | Ý nghĩa |
|---|---|---|---|
| id | BIGINT UNSIGNED | PK | |
| user_id | BIGINT UNSIGNED | FK → users, CASCADE | |
| token_hash | CHAR(64) | UNIQUE | SHA-256 của token |
| expires_at | DATETIME | NOT NULL | Tạo + 30 phút |
| used_at | DATETIME | NULL | Khác NULL = đã dùng hoặc đã vô hiệu |
| created_at | DATETIME | | Dùng để giới hạn 1 liên kết/60 giây |

### regions: 34 tỉnh/thành (M3)

| Cột | Kiểu | Ràng buộc | Ý nghĩa |
|---|---|---|---|
| id | BIGINT UNSIGNED | PK | |
| name | VARCHAR(100) | NOT NULL | Ví dụ "Lâm Đồng" |
| slug | VARCHAR(120) | UNIQUE | Ví dụ `lam-dong`; khớp `slug` trong `provinces.geojson` |
| level | ENUM('province','city','other') | DEFAULT 'province' | 6 thành phố trực thuộc trung ương là `city` |

### memories: ký ức (M2)

| Cột | Kiểu | Ràng buộc | Ý nghĩa |
|---|---|---|---|
| id | BIGINT UNSIGNED | PK | |
| user_id | BIGINT UNSIGNED | FK → users, CASCADE | Chủ sở hữu, không đổi được (trigger) |
| region_id | BIGINT UNSIGNED | NULL, FK → regions, SET NULL | Tỉnh/thành tự gán; NULL = ngoài Việt Nam |
| title | VARCHAR(200) | NOT NULL | |
| content | MEDIUMTEXT | NOT NULL | Có thể rỗng |
| memory_date | DATE | NOT NULL | Không ở tương lai (trigger) |
| place_name | VARCHAR(255) | NOT NULL | |
| latitude | DECIMAL(9,6) | CHECK −90..90 | |
| longitude | DECIMAL(9,6) | CHECK −180..180 | |
| visibility | ENUM('private','public') | DEFAULT 'private' | BR-01 |
| status | ENUM('active','hidden') | DEFAULT 'active' | `hidden` = quản trị viên ẩn |
| deleted_at | DATETIME | NULL | Xóa mềm |
| created_at, updated_at | DATETIME | MySQL tự ghi | |

Chỉ mục: `idx_mem_user_date (user_id, memory_date)`, `idx_mem_public_geo (visibility, latitude, longitude)`, `idx_mem_region (region_id, visibility)`, `idx_mem_pub (visibility, status, deleted_at, latitude)`, FULLTEXT `ft_mem_search (title, place_name) WITH PARSER ngram`.

### media: ảnh/video (M2)

| Cột | Kiểu | Ràng buộc | Ý nghĩa |
|---|---|---|---|
| id | BIGINT UNSIGNED | PK | |
| memory_id | BIGINT UNSIGNED | FK → memories, CASCADE | |
| media_type | ENUM('image','video') | | |
| file_path | VARCHAR(500) | NOT NULL | Đường dẫn tương đối trong `uploads/` |
| thumb_path | VARCHAR(500) | NULL | Dự phòng cho ảnh thu nhỏ (chưa dùng) |
| mime_type | VARCHAR(100) | NOT NULL | |
| size_bytes | INT UNSIGNED | NOT NULL | Sau khi xóa siêu dữ liệu |
| sort_order | SMALLINT UNSIGNED | DEFAULT 0 | Thứ tự ảnh |
| created_at | DATETIME | | |

### trips, trip_memories: chuyến đi (M2)

| Bảng.Cột | Kiểu | Ràng buộc | Ý nghĩa |
|---|---|---|---|
| trips.id | BIGINT UNSIGNED | PK | |
| trips.user_id | BIGINT UNSIGNED | FK → users, CASCADE | |
| trips.name | VARCHAR(150) | NOT NULL | |
| trips.description | VARCHAR(500) | NULL | |
| trips.created_at, updated_at | DATETIME | | |
| trip_memories.trip_id | BIGINT UNSIGNED | PK, FK → trips, CASCADE | |
| trip_memories.memory_id | BIGINT UNSIGNED | PK, FK → memories, CASCADE | |
| trip_memories.added_at | DATETIME | | |

### hashtags, memory_hashtags: hashtag (M2)

| Bảng.Cột | Kiểu | Ràng buộc | Ý nghĩa |
|---|---|---|---|
| hashtags.id | BIGINT UNSIGNED | PK | |
| hashtags.name | VARCHAR(30) `utf8mb4_0900_as_ci` | UNIQUE | Không có "#", chữ thường; phân biệt dấu, không phân biệt hoa thường |
| hashtags.created_at | DATETIME | | |
| memory_hashtags.memory_id | BIGINT UNSIGNED | PK, FK → memories, CASCADE | |
| memory_hashtags.hashtag_id | BIGINT UNSIGNED | PK, FK → hashtags, CASCADE | |
| memory_hashtags.source | ENUM('manual','ai') | DEFAULT 'manual' | Nguồn hashtag |

### admin_actions: nhật ký quản trị (M4)

| Cột | Kiểu | Ràng buộc | Ý nghĩa |
|---|---|---|---|
| id | BIGINT UNSIGNED | PK | |
| admin_id | BIGINT UNSIGNED | FK → users, RESTRICT | Không xóa được quản trị viên đã có nhật ký |
| action | ENUM('hide_memory','unhide_memory','delete_memory','lock_user','unlock_user') | | |
| target_type | ENUM('memory','user') | Khớp với `action` (trigger) | |
| target_id | BIGINT UNSIGNED | | Id ký ức hoặc tài khoản |
| reason | VARCHAR(500) | Bắt buộc với hide/delete/lock (trigger) | |
| created_at | DATETIME | | |

## C.2 Trigger

| Trigger | Bảng, thời điểm | Tác dụng | Quy tắc |
|---|---|---|---|
| `trg_users_bi` | users, BEFORE INSERT | Email về chữ thường, bỏ khoảng trắng | BR-08 |
| `trg_users_bu` | users, BEFORE UPDATE | Chuẩn hóa email; tăng `token_version` khi đổi `password_hash` hoặc chuyển sang `locked` | BR-08, BR-20 |
| `trg_memories_bi` | memories, BEFORE INSERT | Chặn ngày ở tương lai | BR-05 |
| `trg_memories_bu` | memories, BEFORE UPDATE | Chặn ngày ở tương lai, chặn đổi chủ sở hữu | BR-05, BR-02 |
| `trg_media_bi` | media, BEFORE INSERT | ≤ 10 ảnh ≤ 5 MB, ≤ 1 video ≤ 50 MB | BR-06 |
| `trg_hashtags_bi` | hashtags, BEFORE INSERT | Bỏ "#", chữ thường, chỉ chữ/số/"_" | BR-06 |
| `trg_memory_hashtags_bi` | memory_hashtags, BEFORE INSERT | ≤ 10 hashtag/ký ức | BR-06 |
| `trg_trip_memories_bi` | trip_memories, BEFORE INSERT | Ký ức và chuyến đi cùng chủ | BR-09 |
| `trg_admin_actions_bi` | admin_actions, BEFORE INSERT | Người thao tác phải là admin; `action` khớp `target_type`; có lý do; không tự khóa | BR-10, BR-11 |
| `trg_admin_actions_bu` | admin_actions, BEFORE UPDATE | Cấm sửa nhật ký | BR-26 |
| `trg_admin_actions_bd` | admin_actions, BEFORE DELETE | Cấm xóa nhật ký | BR-26 |

Trigger báo lỗi bằng `SIGNAL SQLSTATE '45000'` (mã MySQL 1644); ứng dụng chuyển thành 422 `BUSINESS_RULE` với đúng câu của trigger. Khi đổi mật khẩu hoặc khóa tài khoản, ứng dụng và trigger cùng đặt `token_version = OLD + 1` nên không bị tăng hai lần.

## C.3 View

`v_public_memories`: ký ức hiện trên bản đồ cộng đồng (BR-03, BR-12). Gồm id, tiêu đề, nội dung, ngày, địa điểm, tọa độ, region_id, created_at, `author_id`, `author_name`, `author_avatar`; điều kiện `visibility = 'public' AND status = 'active' AND deleted_at IS NULL AND users.status = 'active'`. Ứng dụng áp cùng điều kiện qua ORM (`public_query`); view dùng cho truy vấn tay và báo cáo.

## C.4 Lưu trữ tệp

| Thư mục (trong `uploads/`) | Nội dung | Tên tệp |
|---|---|---|
| `images/` | Ảnh ký ức đã xóa siêu dữ liệu | `{uuid}.{đuôi}` |
| `videos/` | Video ký ức | tên ngẫu nhiên |
| `avatars/` | Ảnh đại diện | `{user_id}_{12 ký tự ngẫu nhiên}.{đuôi}` |

Thư mục nằm ngoài kho mã nguồn (`.gitignore`) và không được phục vụ tĩnh; chỉ đọc qua API có kiểm quyền.
