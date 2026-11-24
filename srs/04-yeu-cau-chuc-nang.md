[← Mục lục SRS](README.md)

# 4. Yêu cầu chức năng

Cột **Trạng thái** theo quy ước ở [mục lục](README.md#quy-ước). Yêu cầu có mã từ FR-x.10 trở lên (hoặc ghi "mới ở 1.0") là chức năng đã có trong sản phẩm nhưng chưa được ghi ở bản 0.3.

## 4.1 Phần 1: Tài khoản, xác thực và hồ sơ cá nhân

| **Mã** | **Yêu cầu** | **GĐ** | **Trạng thái** |
|---|---|---|---|
| FR-1.1 | Người dùng đăng ký bằng tên hiển thị, email và mật khẩu. Hệ thống kiểm tra email đúng định dạng và chưa được dùng (không phân biệt hoa thường), mật khẩu đủ mạnh (BR-07). Đăng ký xong được đăng nhập ngay, chưa bắt buộc xác thực email. | GĐ1 | ✅ Đạt |
| FR-1.2 | Người dùng đăng nhập bằng email và mật khẩu. Sai thì báo chung "Email hoặc mật khẩu không đúng"; sai 5 lần trong 15 phút thì tạm khóa đăng nhập 15 phút (BR-15); tài khoản bị quản trị viên khóa thì không vào được. | GĐ1 | ✅ Đạt |
| FR-1.3 | Người dùng đăng nhập bằng tài khoản Google (OAuth 2.0 / OpenID Connect, luồng authorization code ở máy chủ). | GĐ1 | 🔧 Cần cấu hình (code và test giả lập đạt) |
| FR-1.4 | Người dùng đăng nhập bằng tài khoản Facebook (Graph API, luồng authorization code ở máy chủ). | GĐ1 | 🔧 Cần cấu hình (code và test giả lập đạt) |
| FR-1.5 | Quên mật khẩu: gửi email chứa liên kết đặt lại có hiệu lực 30 phút, dùng một lần; trang đặt lại kiểm tra liên kết ngay khi mở (BR-14, BR-23). | GĐ1 | ✅ Đạt |
| FR-1.6 | Đăng xuất. Đổi mật khẩu khi đang đăng nhập (phải nhập đúng mật khẩu hiện tại, mật khẩu mới phải khác mật khẩu cũ); tài khoản chỉ đăng nhập mạng xã hội được **tạo** mật khẩu lần đầu. | GĐ1 | ✅ Đạt |
| FR-1.7 | Hồ sơ cá nhân: tên hiển thị (≤ 100 ký tự), email, giới thiệu ngắn (≤ 500 ký tự), ảnh đại diện (≤ 5 MB, .jpg/.jpeg/.png/.webp/.gif, đã xóa EXIF/GPS); người dùng sửa được, gỡ được ảnh đại diện. | GĐ1 | ✅ Đạt |
| FR-1.8 | Trang hồ sơ hiển thị ký ức (lọc theo năm) và chuyến đi của người dùng. | GĐ1 | ✅ Đạt |
| FR-1.9 | Đăng nhập Google/Facebook lần đầu thì tự tạo tài khoản (tên hiển thị lấy từ nhà cung cấp). Nếu đã có tài khoản cùng email thì liên kết vào tài khoản đó theo BR-08 và BR-18. Tài khoản chưa có email (Facebook không trả email) được nhắc bổ sung email trong hồ sơ. | GĐ1 | 🔧 Cần cấu hình; ⚠️ cách liên kết Facebook xem mục 11.3 |
| FR-1.10 | *(mới ở 1.0)* Trang hồ sơ hiển thị thống kê: tổng số ký ức, số ký ức công khai, số chuyến đi, số tỉnh/thành đã có ký ức. | GĐ1 | ✅ Đạt |
| FR-1.11 | *(mới ở 1.0)* Trang đăng nhập/đăng ký tự làm mờ nút Google hoặc Facebook khi máy chủ chưa cấu hình đủ khóa của nhà cung cấp đó. | GĐ1 | ✅ Đạt |

## 4.2 Phần 2: Ký ức, chuyến đi và bản đồ

| **Mã** | **Yêu cầu** | **GĐ** | **Trạng thái** |
|---|---|---|---|
| FR-2.1 | Bản đồ toàn màn hình (Leaflet + OpenStreetMap). Ghim của người dùng màu xanh (xanh đậm khi riêng tư, xám khi bị quản trị viên ẩn), ghim công khai của cộng đồng màu đỏ, ghim vị trí đang chọn màu tím. | GĐ1 | ✅ Đạt |
| FR-2.2 | Tạo ký ức bằng hai cách: (a) nhập tên địa điểm, nhấn Enter hoặc "Tìm", chọn một trong tối đa 5 kết quả; (b) nhấp đúp hoặc nhấp chuột phải trên bản đồ (máy tính), nhấn giữ 600 ms (điện thoại), hoặc bấm "Chọn trên bản đồ" rồi nhấp. Ghim tím kéo được để chỉnh vị trí; tên địa điểm tự điền bằng reverse geocode, sửa được. | GĐ1 | ✅ Đạt |
| FR-2.3 | Mỗi ký ức gồm: tiêu đề (bắt buộc, ≤ 200 ký tự), tên địa điểm (bắt buộc, ≤ 255 ký tự), tọa độ (bắt buộc), ngày (bắt buộc, không ở tương lai), nội dung (không giới hạn độ dài thực tế, kiểu MEDIUMTEXT), ảnh, video, hashtag, chế độ riêng tư/công khai (mặc định riêng tư). | GĐ1 | ✅ Đạt (trừ hashtag, xem FR-2.9) |
| FR-2.4 | Đính kèm tối đa 10 ảnh (mỗi ảnh ≤ 5 MB; .jpg, .jpeg, .png, .webp, .gif) và tối đa 1 video (≤ 50 MB; .mp4, .mov). Máy chủ kiểm đuôi tệp, kiểu MIME và chữ ký nhị phân của tệp; xóa EXIF/GPS khỏi ảnh trước khi lưu. | GĐ1 | ✅ Đạt |
| FR-2.5 | Người dùng sửa tiêu đề, nội dung, địa điểm, tọa độ, ngày, ảnh, video của ký ức mình; xóa ký ức sau bước xác nhận. Xóa là xóa mềm, đồng thời gỡ ký ức khỏi mọi chuyến đi. | GĐ1 | ✅ Đạt |
| FR-2.6 | Chuyển chế độ riêng tư ⇄ công khai bất kỳ lúc nào. Khi chuyển sang công khai, giao diện hỏi xác nhận vì khách cũng sẽ xem được nội dung, ảnh, video. | GĐ1 | ✅ Đạt |
| FR-2.7 | Tạo chuyến đi (tên ≤ 150 ký tự, mô tả ≤ 500 ký tự), sửa, xóa; thêm/bỏ ký ức trong chuyến đi. Ký ức trong chuyến đi xếp theo ngày ký ức, trùng ngày thì theo thời điểm tạo. Xóa chuyến đi không xóa ký ức. | GĐ1 | ✅ Đạt qua panel Chuyến đi; ⚠️ ô chọn chuyến đi trong form ký ức chưa lưu (mục 11.3) |
| FR-2.8 | Nhấp vào ghim để xem chi tiết: tiêu đề, ngày, địa điểm, tỉnh/thành, nội dung, ảnh, video (phát trực tiếp, tua được), hashtag (đánh dấu hashtag do AI gợi ý), tác giả (tên hiển thị, ảnh đại diện). Chủ ký ức thấy thêm nút sửa, xóa, đổi chế độ và danh sách chuyến đi chứa ký ức. | GĐ1 | ✅ Đạt |
| FR-2.9 | Gắn tối đa 10 hashtag cho ký ức theo định dạng BR-06; hashtag dùng chung không phân biệt hoa thường; gợi ý tự động bằng AI ở FR-3.6. | GĐ1 | ⚠️ API đạt; form ký ức trên giao diện chưa lưu được hashtag (mục 11.3) |
| FR-2.10 | *(mới ở 1.0)* Khi sửa ký ức: thêm ảnh, bỏ ảnh, sắp xếp lại thứ tự ảnh; thay hoặc gỡ video. Có thanh tiến trình khi tải tệp lên. | GĐ1 | ✅ Đạt |
| FR-2.11 | *(mới ở 1.0)* Chuyến đi hiển thị số ký ức, ngày bắt đầu và kết thúc (theo ký ức sớm nhất và muộn nhất), ảnh bìa (ảnh đầu tiên của từng ký ức); khi mở chuyến đi, bản đồ thu phóng vừa các ghim và vẽ lộ trình nét đứt nối các ký ức theo thời gian. | GĐ1 | ✅ Đạt |
| FR-2.12 | *(mới ở 1.0)* Nút "về vị trí hiện tại" dùng định vị của trình duyệt; bật/tắt riêng lớp ghim của tôi và lớp ghim cộng đồng. | GĐ1 | ✅ Đạt |
| FR-2.13 | *(mới ở 1.0)* Khi tạo hoặc đổi tọa độ ký ức, hệ thống tự xác định tỉnh/thành theo ranh giới địa lý (BR-25). | GĐ1 | ✅ Đạt |

## 4.3 Phần 3: Khám phá, bộ lọc và tìm kiếm

| **Mã** | **Yêu cầu** | **GĐ** | **Trạng thái** |
|---|---|---|---|
| FR-3.1 | Panel Khám phá: lọc ký ức công khai theo **tỉnh/thành** (một trong 34, có số ký ức công khai mỗi tỉnh) và theo **hashtag** (chọn nhiều, phép OR, BR-16); kết hợp được cả hai (phép AND giữa hai loại lọc). | GĐ1 | ✅ Đạt |
| FR-3.2 | Tìm ký ức công khai theo từ khóa trong tiêu đề hoặc tên địa điểm, không phân biệt dấu và hoa thường, từ 2 đến 100 ký tự; kết quả liên quan nhất lên trước, cùng mức thì ký ức có ngày mới hơn lên trước. | GĐ1 | ✅ Đạt |
| FR-3.3 | Panel Của tôi: lọc ký ức của mình theo năm, tháng hoặc khoảng ngày (gồm cả hai đầu); lọc thêm theo chế độ, hashtag của tôi, từ khóa; sắp xếp tăng hoặc giảm theo ngày. | GĐ1 | ✅ Đạt |
| FR-3.4 | Thanh bên có danh sách chuyến đi và bộ lọc hashtag; kết quả lọc cập nhật đồng thời ở danh sách và ghim trên bản đồ. | GĐ1 | ✅ Đạt |
| FR-3.5 | Danh sách phân trang ("Tải thêm"): Khám phá 12 mục/trang, Của tôi và Quản trị 20 mục/trang (tối đa 50 hoặc 100). Bản đồ chỉ tải ghim trong vùng đang xem, tối đa 500 ghim công khai mỗi lần; tải lại sau khi ngừng kéo/thu phóng 250 ms. | GĐ1 | ✅ Đạt |
| FR-3.6 | Gợi ý hashtag tự động: nút "✨ Gợi ý hashtag" trong form gửi tiêu đề, nội dung, địa điểm tới Google Gemini, nhận 4–6 hashtag tiếng Việt; người dùng chọn hashtag muốn dùng. Không có khóa Gemini hoặc Gemini lỗi thì gợi ý theo từ khóa trong văn bản. Hashtag chọn từ gợi ý AI được lưu nguồn `ai`. | GĐ1 (vốn là tùy chọn) | ✅ Đạt phần gợi ý; ⚠️ lưu hashtag từ form, xem FR-2.9 |
| FR-3.7 | *(mới ở 1.0)* Tùy chọn "Chỉ trong vùng bản đồ đang xem" cho danh sách Khám phá; nút "Bỏ lọc" trả về toàn bộ ký ức công khai. | GĐ1 | ✅ Đạt |
| FR-3.8 | *(mới ở 1.0)* Trang mở đầu hiển thị 3 ký ức công khai mới nhất, tổng số ký ức công khai và số tỉnh/thành có ký ức; panel Khám phá hiển thị tối đa 12 hashtag phổ biến nhất trong ký ức công khai, cho nhập thêm hashtag tùy ý. | GĐ1 | ✅ Đạt |

## 4.4 Phần 4: Quản trị và cộng đồng

| **Mã** | **Yêu cầu** | **GĐ** | **Trạng thái** |
|---|---|---|---|
| FR-4.1 | Thích (like) và bỏ thích một ký ức công khai. | GĐ2 | ⛔ Chưa làm |
| FR-4.2 | Bình luận vào ký ức công khai; chủ bình luận xóa được bình luận của mình, chủ ký ức xóa được bình luận trong ký ức của mình. | GĐ2 | ⛔ Chưa làm |
| FR-4.3 | Lưu ký ức/địa điểm vào danh sách "Muốn đi"; xem và bỏ lưu. | GĐ2 | ⛔ Chưa làm |
| FR-4.4 | Theo dõi người dùng khác và xem ký ức công khai của họ. | GĐ2 | ⛔ Chưa làm |
| FR-4.5 | Báo cáo ký ức hoặc bình luận vi phạm, kèm lý do; quản trị viên xử lý hàng đợi báo cáo. | GĐ2 | ⛔ Chưa làm |
| FR-4.6 | Quản trị viên ẩn/bỏ ẩn hoặc xóa ký ức công khai vi phạm, khóa/mở khóa tài khoản vi phạm. Khóa tài khoản thì người đó bị đăng xuất ngay và toàn bộ ký ức công khai của họ ẩn khỏi cộng đồng tới khi mở khóa. | GĐ1 | ✅ Đạt |
| FR-4.7 | Chỉ người dùng đã đăng nhập mới được thích, bình luận, lưu, theo dõi, báo cáo. | GĐ2 | ⛔ Chưa làm |
| FR-4.8 | Quản trị viên xem và tìm danh sách ký ức công khai (theo tiêu đề, tên địa điểm, tên tác giả), lọc theo trạng thái đang hiển thị/đã ẩn, để phát hiện vi phạm. | GĐ1 | ✅ Đạt |
| FR-4.9 | Mọi thao tác ẩn, xóa, khóa phải có lý do (≤ 500 ký tự); bỏ ẩn và mở khóa không bắt buộc lý do. Mọi thao tác quản trị được ghi nhật ký không sửa, không xóa được (người thao tác, thời điểm, loại thao tác, đối tượng, lý do). | GĐ1 | ✅ Đạt |
| FR-4.10 | Quản trị viên tìm tài khoản theo email hoặc tên hiển thị, lọc theo trạng thái hoạt động/đã khóa; danh sách hiện vai trò, trạng thái và số ký ức. | GĐ1 | ✅ Đạt |
| FR-4.11 | *(mới ở 1.0)* Trang quản trị có thống kê: tổng tài khoản, tài khoản bị khóa, tổng ký ức, ký ức công khai, ký ức bị ẩn, ký ức mới trong 7 ngày. | GĐ1 | ✅ Đạt |
| FR-4.12 | *(mới ở 1.0)* Trang quản trị có tab Nhật ký thao tác, lọc theo loại thao tác, phân trang. | GĐ1 | ✅ Đạt |
| FR-4.13 | *(mới ở 1.0)* Quản trị viên cũng ẩn, bỏ ẩn, xóa được ngay trong khung chi tiết ký ức trên bản đồ (cùng API và quy tắc với trang quản trị). | GĐ1 | ✅ Đạt |
