[← Danh sách epic](README.md)

# 5.3 Epic EP1: Tài khoản và hồ sơ cá nhân

Module M1 (`app/modules/accounts/`). 8 story, 33 tiêu chí nghiệm thu. Kết quả kiểm thử tích hợp: 28 đạt, 5 cần cấu hình OAuth.

## US-ACC-001: Đăng ký tài khoản bằng email

> **Là một người dùng mới chưa có tài khoản Bản đồ Ký ức, Tôi muốn tạo tài khoản bằng email và mật khẩu, Để có không gian riêng để lưu ký ức và quay lại sử dụng ở những lần sau.**

| **Epic** | **EP1** | **Ưu tiên** | **Must** |
|---|---|---|---|
| FR liên quan | FR-1.1 | Trạng thái | ✅ Hoàn thành |
| Phụ thuộc | Không có | Kiểm thử | `tests/accounts/test_register.py`; tích hợp 5/5 đạt |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (5 AC đo được).

### Tiêu chí nghiệm thu (5)

#### AC1: Đăng ký thành công

| **Given** | **khách đang ở trang đăng ký và email "an@example.com" chưa có tài khoản** |
|---|---|
| When | khách nhập tên hiển thị "An", email "an@example.com", mật khẩu "Matkhau123" rồi gửi |
| Then | tài khoản mới được tạo với vai trò người dùng |
| And | khách được đăng nhập và chuyển tới bản đồ ký ức của mình |
| And | hệ thống hiển thị thông báo "Đăng ký thành công" |

#### AC2: Mật khẩu đúng độ dài tối thiểu

| **Given** | **khách đang ở trang đăng ký** |
|---|---|
| When | khách nhập mật khẩu "Abcdef12" gồm đúng 8 ký tự có chữ và số |
| Then | mật khẩu được chấp nhận và tài khoản được tạo |

#### AC3: Mật khẩu quá ngắn

| **Given** | **khách đang ở trang đăng ký** |
|---|---|
| When | khách nhập mật khẩu "Abc12" gồm 5 ký tự |
| Then | hệ thống không tạo tài khoản |
| And | hệ thống hiển thị "Mật khẩu phải có ít nhất 8 ký tự gồm chữ và số" |

#### AC4: Email đã được đăng ký

| **Given** | **đã tồn tại tài khoản với email "an@example.com"** |
|---|---|
| When | khách đăng ký với email "an@example.com" (kể cả viết "AN@Example.com") |
| Then | hệ thống không tạo thêm tài khoản |
| And | hệ thống hiển thị "Email này đã được đăng ký" |

#### AC5: Email sai định dạng

| **Given** | **khách đang ở trang đăng ký** |
|---|---|
| When | khách nhập email "an@" |
| Then | hệ thống hiển thị "Email không hợp lệ" |
| And | không có tài khoản nào được tạo |

### Đặc tả cài đặt

- API: `POST /api/auth/register` với `{display_name, email, password}`. Thành công trả 201 kèm JWT và thông tin người dùng (`AuthResponse`), trình duyệt lưu JWT và vào `/ban-do`.
- Kiểm tra: tên hiển thị sau khi cắt khoảng trắng phải có 1–100 ký tự ("Tên hiển thị không được để trống và tối đa 100 ký tự"); email ≤ 255 ký tự, kiểm bằng `email-validator`, lưu chữ thường; mật khẩu ≥ 8 ký tự, có ít nhất một chữ cái và một chữ số, không quá 72 byte UTF-8 ("Mật khẩu không được dài quá 72 byte", giới hạn của bcrypt).
- Email trùng trả 409. Hai yêu cầu đồng thời cùng email: ràng buộc UNIQUE chặn bản thứ hai, vẫn trả "Email này đã được đăng ký".
- GĐ1 chưa bắt buộc xác thực email; cột `users.email_verified_at` có sẵn nhưng chưa được ghi.

## US-ACC-002: Đăng nhập bằng email và mật khẩu

> **Là một người dùng đã đăng ký tài khoản bằng email, Tôi muốn đăng nhập bằng email và mật khẩu của mình, Để truy cập lại các ký ức và chuyến đi đã lưu.**

| **Epic** | **EP1** | **Ưu tiên** | **Must** |
|---|---|---|---|
| FR liên quan | FR-1.2 | Trạng thái | ✅ Hoàn thành |
| Phụ thuộc | US-ACC-001 | Kiểm thử | `tests/accounts/test_login.py`; tích hợp 5/5 đạt |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (5 AC).

### Tiêu chí nghiệm thu (5)

#### AC1: Đăng nhập thành công

| **Given** | **tài khoản "an@example.com" đang hoạt động** |
|---|---|
| When | người dùng nhập đúng email và mật khẩu |
| Then | người dùng được chuyển tới bản đồ ký ức của mình |
| And | tên hiển thị của người dùng xuất hiện trên màn hình |

#### AC2: Email không phân biệt hoa thường

| **Given** | **tài khoản đăng ký với email "an@example.com"** |
|---|---|
| When | người dùng nhập "AN@Example.com" cùng mật khẩu đúng |
| Then | người dùng đăng nhập thành công |

#### AC3: Sai mật khẩu

| **Given** | **tài khoản "an@example.com" đang hoạt động** |
|---|---|
| When | người dùng nhập mật khẩu sai |
| Then | hệ thống hiển thị "Email hoặc mật khẩu không đúng" |
| And | thông báo không cho biết email hay mật khẩu bị sai |

#### AC4: Tài khoản bị khóa

| **Given** | **tài khoản "an@example.com" đã bị quản trị viên khóa** |
|---|---|
| When | người dùng nhập đúng email và mật khẩu |
| Then | hệ thống không cho đăng nhập |
| And | hệ thống hiển thị "Tài khoản của bạn đã bị khóa" |

#### AC5: Nhập sai nhiều lần liên tiếp

| **Given** | **người dùng đã nhập sai mật khẩu 5 lần liên tiếp trong 15 phút** |
|---|---|
| When | người dùng nhập tiếp lần thứ 6, kể cả với mật khẩu đúng |
| Then | hệ thống từ chối đăng nhập |
| And | hệ thống hiển thị "Bạn đã nhập sai quá nhiều lần, vui lòng thử lại sau 15 phút" |

### Đặc tả cài đặt

- API: `POST /api/auth/login` với `{email, password}` trả JWT; `GET /api/auth/me` trả người dùng hiện tại. Cả hai đặt `Cache-Control: no-store`.
- Mã lỗi: sai thông tin 401; tài khoản bị khóa 403; đang tạm khóa đăng nhập 429. Lần sai thứ 5 cũng trả ngay thông báo tạm khóa.
- Bộ đếm (BR-15): `failed_login_count`, `first_failed_at`, `login_locked_until` trên bảng `users`. Cửa sổ 15 phút tính từ lần sai đầu tiên; quá 15 phút thì đếm lại. Đăng nhập đúng xóa bộ đếm và ghi `last_login_at`.
- Chống dò tài khoản: email không tồn tại vẫn chạy so khớp bcrypt với một bản băm giả để thời gian phản hồi tương đương, và trả cùng thông báo như sai mật khẩu. Tài khoản chỉ có đăng nhập mạng xã hội (chưa có mật khẩu) cũng nhận "Email hoặc mật khẩu không đúng".
- Trạng thái khóa được kiểm sau khi mật khẩu đúng, nên người không biết mật khẩu không biết được tài khoản có bị khóa hay không.
- Dòng `users` được khóa (`SELECT … FOR UPDATE`) trong lúc kiểm, để nhiều yêu cầu đồng thời không làm sai bộ đếm.

## US-ACC-003: Đăng nhập bằng tài khoản Google

> **Là một người dùng có tài khoản Google, không muốn nhớ thêm mật khẩu, Tôi muốn đăng nhập bằng tài khoản Google, Để vào hệ thống nhanh mà không phải tạo mật khẩu riêng.**

| **Epic** | **EP1** | **Ưu tiên** | **Must** |
|---|---|---|---|
| FR liên quan | FR-1.3, FR-1.9 | Trạng thái | ✅ Code hoàn thành · 🔧 cần khóa Google thật |
| Phụ thuộc | Không có | Kiểm thử | `tests/accounts/test_us_acc_003_google.py` (10 ca, nhà cung cấp giả lập); tích hợp: AC3 đạt, AC1/AC2/AC4 cần cấu hình |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (4 AC).

### Tiêu chí nghiệm thu (4)

#### AC1: Đăng nhập Google lần đầu

| **Given** | **chưa có tài khoản nào dùng email Google "an@gmail.com"** |
|---|---|
| When | người dùng chọn "Đăng nhập bằng Google" và đồng ý cấp quyền |
| Then | hệ thống tạo tài khoản mới với tên hiển thị lấy từ Google |
| And | người dùng được chuyển tới bản đồ ký ức của mình |

#### AC2: Email Google trùng tài khoản email đã có

| **Given** | **đã có tài khoản đăng ký bằng email "an@gmail.com" và có 3 ký ức, và Google xác nhận email này đã được xác thực** |
|---|---|
| When | người dùng đăng nhập bằng Google với email "an@gmail.com" |
| Then | hệ thống liên kết vào tài khoản đã có, không tạo tài khoản thứ hai |
| And | người dùng thấy đủ 3 ký ức cũ của mình |

#### AC3: Người dùng từ chối cấp quyền

| **Given** | **người dùng đang ở màn hình xin quyền của Google** |
|---|---|
| When | người dùng chọn "Từ chối" |
| Then | hệ thống đưa người dùng về trang đăng nhập |
| And | hệ thống hiển thị "Đăng nhập Google chưa hoàn tất" |

#### AC4: Tài khoản bị khóa

| **Given** | **tài khoản liên kết với "an@gmail.com" đã bị khóa** |
|---|---|
| When | người dùng đăng nhập bằng Google |
| Then | hệ thống không cho đăng nhập |
| And | hệ thống hiển thị "Tài khoản của bạn đã bị khóa" |

### Đặc tả cài đặt

- Luồng: `GET /api/auth/google/login` sinh `state` ngẫu nhiên, lưu cookie `google_oauth_state` (HttpOnly, SameSite=Lax, Secure khi không ở dev, sống 10 phút), chuyển sang Google với scope `openid email profile`. Google gọi lại `GET /api/auth/google/callback`; máy chủ so khớp `state`, đổi `code` lấy token, đọc hồ sơ.
- Tìm tài khoản theo thứ tự: (1) đã có liên kết `oauth_accounts` với id Google → đăng nhập vào tài khoản đó; (2) có tài khoản cùng email → chỉ liên kết khi Google báo `email_verified = true`, nếu không thì quay về trang đăng nhập với "Email Google chưa được xác thực, hãy đăng nhập bằng mật khẩu"; (3) chưa có → tạo tài khoản mới không có mật khẩu.
- Thành công: chuyển tới `/accounts/oauth-callback.html#token=…&notice=…`. JWT nằm sau dấu `#` nên không gửi lên máy chủ và không lưu vào lịch sử truy cập máy chủ; trang này lưu token rồi vào `/ban-do`.
- Lỗi khác (sai `state`, lỗi mạng, id Google đã gắn với tài khoản khác) đều quay về trang đăng nhập với thông báo tương ứng ("Đăng nhập Google chưa hoàn tất" hoặc "Không thể liên kết tài khoản Google này").
- Cấu hình: `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `GOOGLE_REDIRECT_URI`. Thiếu một trong hai khóa thì nút Google bị làm mờ (`GET /api/auth/providers`).

## US-ACC-004: Đăng nhập bằng tài khoản Facebook

> **Là một người dùng có tài khoản Facebook, không muốn nhớ thêm mật khẩu, Tôi muốn đăng nhập bằng tài khoản Facebook, Để vào hệ thống nhanh mà không phải tạo mật khẩu riêng.**

| **Epic** | **EP1** | **Ưu tiên** | **Should** |
|---|---|---|---|
| FR liên quan | FR-1.4, FR-1.9 | Trạng thái | ✅ Code hoàn thành · 🔧 cần khóa Facebook thật |
| Phụ thuộc | Không có | Kiểm thử | `tests/accounts/test_us_acc_004_facebook.py` (nhà cung cấp giả lập); tích hợp: AC3 đạt, AC1/AC2 cần cấu hình |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (3 AC).

### Tiêu chí nghiệm thu (3)

#### AC1: Đăng nhập Facebook lần đầu

| **Given** | **chưa có tài khoản nào liên kết với Facebook của người dùng** |
|---|---|
| When | người dùng chọn "Đăng nhập bằng Facebook" và đồng ý cấp quyền |
| Then | hệ thống tạo tài khoản mới với tên hiển thị lấy từ Facebook |
| And | người dùng được chuyển tới bản đồ ký ức của mình |

#### AC2: Facebook không cung cấp email

| **Given** | **tài khoản Facebook của người dùng không có email công khai** |
|---|---|
| When | người dùng đăng nhập bằng Facebook lần đầu |
| Then | hệ thống vẫn tạo tài khoản |
| And | hệ thống đưa người dùng tới trang hồ sơ, nhắc bổ sung email để dùng chức năng quên mật khẩu |

#### AC3: Người dùng từ chối cấp quyền

| **Given** | **người dùng đang ở màn hình xin quyền của Facebook** |
|---|---|
| When | người dùng chọn "Hủy" |
| Then | hệ thống đưa người dùng về trang đăng nhập |
| And | hệ thống hiển thị "Đăng nhập Facebook chưa hoàn tất" |

### Đặc tả cài đặt

- Luồng giống Google: `GET /api/auth/facebook/login` (scope `email`, cookie `facebook_oauth_state`) → Facebook → `GET /api/auth/facebook/callback` → đọc `id,name,email` từ Graph API.
- Tìm tài khoản: (1) đã có liên kết với id Facebook → đăng nhập; (2) Facebook trả email trùng tài khoản đã có → **liên kết vào tài khoản đó** (bản cài đặt hiện không kiểm email đã xác thực, khác BR-18 bản 0.3; xem mục 11.3); (3) chưa có → tạo tài khoản mới, email có thể rỗng.
- Tài khoản không có email: chuyển tới `/accounts/profile.html?notice=need_email`, trang hồ sơ hiện lời nhắc bổ sung email.
- Tài khoản bị khóa: quay về trang đăng nhập với "Tài khoản của bạn đã bị khóa".
- Cấu hình: `FACEBOOK_APP_ID`, `FACEBOOK_APP_SECRET`, `FACEBOOK_REDIRECT_URI`, `FACEBOOK_HTTP_TIMEOUT_SECONDS`.

## US-ACC-005: Đặt lại mật khẩu khi quên

> **Là một người dùng đã đăng ký bằng email nhưng quên mật khẩu, Tôi muốn nhận liên kết đặt lại mật khẩu qua email, Để lấy lại quyền truy cập tài khoản mà không mất ký ức đã lưu.**

| **Epic** | **EP1** | **Ưu tiên** | **Must** |
|---|---|---|---|
| FR liên quan | FR-1.5 | Trạng thái | ✅ Hoàn thành |
| Phụ thuộc | US-ACC-001 | Kiểm thử | `tests/accounts/test_us_acc_005_reset_password.py` (10 ca); tích hợp 4/4 đạt |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (4 AC).

### Tiêu chí nghiệm thu (4)

#### AC1: Yêu cầu đặt lại với email đã đăng ký

| **Given** | **tài khoản "an@example.com" đang hoạt động** |
|---|---|
| When | người dùng nhập "an@example.com" ở trang quên mật khẩu |
| Then | hệ thống gửi email chứa liên kết đặt lại có hiệu lực 30 phút |
| And | hệ thống hiển thị "Nếu email tồn tại, chúng tôi đã gửi hướng dẫn đặt lại mật khẩu" |

#### AC2: Email chưa đăng ký

| **Given** | **không có tài khoản nào dùng email "khong.co@example.com"** |
|---|---|
| When | người dùng nhập "khong.co@example.com" ở trang quên mật khẩu |
| Then | hệ thống hiển thị đúng thông báo như trường hợp email đã đăng ký |
| And | hệ thống không gửi email nào |

#### AC3: Đặt mật khẩu mới thành công

| **Given** | **người dùng mở liên kết đặt lại còn hiệu lực** |
|---|---|
| When | người dùng nhập mật khẩu mới "Matkhaumoi9" hợp lệ và gửi |
| Then | mật khẩu mới có hiệu lực |
| And | mật khẩu cũ không còn đăng nhập được |
| And | liên kết đó không dùng lại được |

#### AC4: Liên kết hết hạn

| **Given** | **liên kết đặt lại đã được gửi cách đây 31 phút** |
|---|---|
| When | người dùng mở liên kết |
| Then | hệ thống hiển thị "Liên kết đã hết hạn, vui lòng yêu cầu lại" |
| And | mật khẩu không bị thay đổi |

### Đặc tả cài đặt

- API: `POST /api/auth/forgot-password` `{email}`; `POST /api/auth/validate-reset-token` `{token}` (trang đặt lại gọi khi vừa mở, báo ngay liên kết hết hạn); `POST /api/auth/reset-password` `{token, new_password}`.
- Token: 32 byte ngẫu nhiên (`secrets.token_urlsafe`), CSDL chỉ lưu SHA-256 (`password_reset_tokens.token_hash`). Liên kết dạng `{APP_BASE_URL}/accounts/reset-password.html?token=…`.
- Mỗi tài khoản nhận tối đa 1 liên kết mỗi 60 giây; tài khoản bị khóa nhận cùng thông báo nhưng không được cấp liên kết (BR-23).
- Đặt lại thành công: đổi mật khẩu, tăng `token_version` (mọi phiên cũ bị đăng xuất), xóa bộ đếm sai mật khẩu, đánh dấu đã dùng **mọi** liên kết chưa dùng của tài khoản. Thông báo "Đặt lại mật khẩu thành công, bạn có thể đăng nhập bằng mật khẩu mới". Liên kết đã dùng hoặc sai: "Liên kết không hợp lệ hoặc đã được sử dụng".
- Gửi email chạy nền sau khi đã trả lời, qua SMTP (STARTTLS cổng 587 hoặc SSL); lỗi SMTP chỉ ghi log, không ghi token. Môi trường thật dùng Gmail SMTP với App Password; kiểm thử dùng Mailtrap. Ở `APP_ENV=dev` chưa cấu hình SMTP thì API trả `dev_reset_url` để thử; môi trường khác không bao giờ trả liên kết qua API.

## US-ACC-006: Cập nhật hồ sơ cá nhân

> **Là một người dùng đã đăng nhập muốn người khác nhận ra mình trên các ký ức công khai, Tôi muốn chỉnh sửa tên hiển thị, ảnh đại diện và giới thiệu ngắn, Để hồ sơ phản ánh đúng con người mình.**

| **Epic** | **EP1** | **Ưu tiên** | **Should** |
|---|---|---|---|
| FR liên quan | FR-1.7, FR-1.8, FR-1.10 | Trạng thái | ✅ Hoàn thành |
| Phụ thuộc | US-ACC-002 | Kiểm thử | `tests/accounts/test_us_acc_006_profile.py` (10 ca); tích hợp 5/5 đạt |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (5 AC).

### Tiêu chí nghiệm thu (5)

#### AC1: Đổi tên hiển thị

| **Given** | **người dùng đang ở trang hồ sơ với tên "An"** |
|---|---|
| When | người dùng đổi tên thành "Dread" và lưu |
| Then | tên "Dread" được lưu |
| And | các ký ức công khai của người dùng hiển thị tên "Dread" |

#### AC2: Tên hiển thị đúng giới hạn

| **Given** | **người dùng đang ở trang hồ sơ** |
|---|---|
| When | người dùng nhập tên dài đúng 100 ký tự và lưu |
| Then | hệ thống chấp nhận và lưu tên |

#### AC3: Tên hiển thị quá dài

| **Given** | **người dùng đang ở trang hồ sơ** |
|---|---|
| When | người dùng nhập tên dài 101 ký tự và lưu |
| Then | hệ thống không lưu |
| And | hệ thống hiển thị "Tên hiển thị tối đa 100 ký tự" |

#### AC4: Ảnh đại diện vượt dung lượng

| **Given** | **người dùng đang chọn ảnh đại diện** |
|---|---|
| When | người dùng chọn ảnh 6 MB |
| Then | hệ thống từ chối ảnh |
| And | hệ thống hiển thị "Ảnh không được vượt quá 5 MB" |

#### AC5: Tệp không phải ảnh

| **Given** | **người dùng đang chọn ảnh đại diện** |
|---|---|
| When | người dùng chọn tệp "baocao.pdf" (hoặc một tệp không phải ảnh đã đổi đuôi thành .jpg) |
| Then | hệ thống từ chối tệp |
| And | hệ thống hiển thị "Định dạng tệp không được hỗ trợ" |

### Đặc tả cài đặt

- API: `PATCH /api/users/me` `{display_name?, email?, bio?}` (chỉ cập nhật trường có gửi); `POST /api/users/me/avatar` (multipart, trường `avatar`); `DELETE /api/users/me/avatar`; `GET /api/users/{id}/avatar` (công khai); `GET /api/users/me/stats`.
- Tên hiển thị rỗng: "Vui lòng nhập tên hiển thị". Giới thiệu > 500 ký tự: "Giới thiệu tối đa 500 ký tự". Đổi sang email đã có người dùng: "Email này đã được đăng ký". Email để trống thì giữ nguyên (tài khoản Facebook có thể chưa có email).
- Ảnh đại diện: ≤ 5 MB; .jpg, .jpeg, .png, .webp, .gif; kiểm cả chữ ký nhị phân; xóa EXIF/GPS (BR-22). Lưu ở `uploads/avatars/{user_id}_{ngẫu nhiên}.{đuôi}`, xóa tệp cũ khi đổi.
- Đường dẫn ảnh đại diện trả ra API luôn là `/api/users/{id}/avatar?v=…` (không lộ đường dẫn lưu trữ); `v` đổi mỗi lần đổi ảnh nên trình duyệt được phép cache 1 ngày.
- Tên tác giả trên ký ức công khai đọc trực tiếp từ bảng `users`, nên đổi tên là mọi ký ức hiển thị tên mới ngay.
- Thống kê: số ký ức chưa xóa, số ký ức công khai, số chuyến đi, số tỉnh/thành khác nhau đã có ký ức.

## US-ACC-007: Đổi mật khẩu khi đang đăng nhập

> **Là một người dùng đã đăng nhập đăng ký bằng email và mật khẩu, Tôi muốn đổi mật khẩu hiện tại sang mật khẩu mới, Để giữ tài khoản an toàn khi nghi ngờ mật khẩu bị lộ.**

| **Epic** | **EP1** | **Ưu tiên** | **Should** |
|---|---|---|---|
| FR liên quan | FR-1.6 | Trạng thái | ✅ Hoàn thành |
| Phụ thuộc | US-ACC-002 | Kiểm thử | `tests/accounts/test_us_acc_007_change_password.py`; tích hợp 4/4 đạt |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (4 AC).

### Tiêu chí nghiệm thu (4)

#### AC1: Đổi mật khẩu thành công

| **Given** | **người dùng đang đăng nhập** |
|---|---|
| When | người dùng nhập đúng mật khẩu hiện tại và mật khẩu mới "Matkhaumoi9" hợp lệ |
| Then | mật khẩu mới có hiệu lực |
| And | hệ thống hiển thị "Đổi mật khẩu thành công" |

#### AC2: Sai mật khẩu hiện tại

| **Given** | **người dùng đang đăng nhập** |
|---|---|
| When | người dùng nhập sai mật khẩu hiện tại |
| Then | mật khẩu không bị đổi |
| And | hệ thống hiển thị "Mật khẩu hiện tại không đúng" |

#### AC3: Mật khẩu mới không hợp lệ

| **Given** | **người dùng đang đăng nhập** |
|---|---|
| When | người dùng nhập mật khẩu mới "abc" gồm 3 ký tự |
| Then | mật khẩu không bị đổi |
| And | hệ thống hiển thị "Mật khẩu phải có ít nhất 8 ký tự gồm chữ và số" |

#### AC4: Tạo mật khẩu cho tài khoản chỉ đăng nhập mạng xã hội

| **Given** | **người dùng đăng nhập bằng Google, chưa từng đặt mật khẩu (password_hash rỗng)** |
|---|---|
| When | người dùng vào Hồ sơ cá nhân, chọn "Tạo mật khẩu" và nhập mật khẩu hợp lệ |
| Then | hệ thống lưu mật khẩu mới và hiển thị "Tạo mật khẩu thành công" |
| And | từ đó người dùng đăng nhập được bằng email và mật khẩu, bên cạnh đăng nhập Google |

### Đặc tả cài đặt

- API: `POST /api/auth/change-password` `{current_password?, new_password}`. Trang hồ sơ đọc `has_password` và `providers` từ `/api/auth/me` để hiện "Đổi mật khẩu" hay "Tạo mật khẩu".
- Tài khoản đã có mật khẩu: thiếu mật khẩu hiện tại → "Vui lòng nhập mật khẩu hiện tại"; mật khẩu mới trùng mật khẩu cũ → "Mật khẩu mới phải khác mật khẩu hiện tại". Lỗi trả mã 400 (không dùng 401, vì 401 khiến trình duyệt tự đăng xuất).
- Thành công: tăng `token_version` nên mọi phiên khác bị đăng xuất; API trả JWT mới để phiên hiện tại tiếp tục dùng.

## US-ACC-008: Đăng xuất khỏi hệ thống

> **Là một người dùng đã đăng nhập đang dùng máy tính dùng chung, Tôi muốn đăng xuất khi dùng xong, Để người khác không xem được ký ức riêng tư của mình.**

| **Epic** | **EP1** | **Ưu tiên** | **Must** |
|---|---|---|---|
| FR liên quan | FR-1.6 | Trạng thái | ✅ Hoàn thành |
| Phụ thuộc | US-ACC-002 | Kiểm thử | `tests/accounts/test_us_acc_008_logout.py`; tích hợp 3/3 đạt (cả máy tính và điện thoại) |

INVEST: I ✅ · N ✅ · V ✅ · E ✅ · S ✅ · T ✅ (3 AC).

### Tiêu chí nghiệm thu (3)

#### AC1: Đăng xuất thành công

| **Given** | **người dùng đang đăng nhập** |
|---|---|
| When | người dùng chọn "Đăng xuất" |
| Then | người dùng được chuyển về trang đăng nhập hoặc bản đồ công khai |
| And | các thông tin cá nhân không còn hiển thị |

#### AC2: Mở lại trang riêng tư sau khi đăng xuất

| **Given** | **người dùng vừa đăng xuất** |
|---|---|
| When | người dùng mở lại trang hồ sơ cá nhân |
| Then | hệ thống yêu cầu đăng nhập |
| And | ký ức riêng tư không hiển thị |

#### AC3: Quay lại bằng nút trở về của trình duyệt

| **Given** | **người dùng vừa đăng xuất** |
|---|---|
| When | người dùng nhấn nút trở về của trình duyệt |
| Then | trang hiển thị yêu cầu đăng nhập thay vì nội dung cá nhân |

### Đặc tả cài đặt

- API: `POST /api/auth/logout` (cần JWT) trả "Đăng xuất thành công" kèm `Cache-Control: no-store`; trình duyệt xóa JWT khỏi `localStorage` (hoặc bộ nhớ nếu trình duyệt chặn `localStorage`).
- Trang không thuộc `/api` có `Cache-Control: no-cache`, nên bấm trở về sẽ tải lại trang và kiểm lại phiên.
- Đăng xuất **không** tăng `token_version` (để không đăng xuất các thiết bị khác). Hệ quả: JWT đã bị lộ trước khi đăng xuất vẫn dùng được tới khi hết hạn; thời hạn mặc định `ACCESS_TOKEN_MINUTES = 1440` (24 giờ). Đây là rủi ro đã ghi nhận (lỗi L11, mục 11.3).
- Trình duyệt nhận mã 401 từ bất kỳ API nào khi đang có token thì tự xóa token (phiên hết hạn hoặc bị thu hồi).
