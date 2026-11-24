[← Mục lục SRS](README.md)

# 1. Giới thiệu

## 1.1 Mục đích

Tài liệu này đặc tả đầy đủ yêu cầu của hệ thống web **Bản đồ Ký ức**: một nhật ký trực tuyến cho phép người dùng lưu kỷ niệm, câu chuyện, hình ảnh và video gắn với địa điểm trên bản đồ. Hệ thống cũng là nơi để cộng đồng, nhất là các bạn trẻ thích đi đây đó, tham khảo và khám phá những địa điểm đẹp qua ký ức công khai của người khác.

Bản 1.0 được viết lại sau khi sản phẩm GĐ1 đã hoàn thành. Mỗi yêu cầu mô tả đúng hành vi của bản cài đặt, kèm trạng thái kiểm thử. Tài liệu dùng để:

- nghiệm thu đồ án: đối chiếu từng tiêu chí nghiệm thu với kết quả kiểm thử (mục 9);
- bảo trì: biết chính xác các giới hạn, thông báo lỗi, quy tắc mà code đang áp dụng;
- làm cơ sở cho GĐ2: biết phần nào đã có, phần nào còn nợ (mục 10, 11).

## 1.2 Phạm vi

**Đã cài đặt (GĐ1)**: ứng dụng web chạy trên trình duyệt máy tính và điện thoại, gồm:

- Tài khoản: đăng ký bằng email, đăng nhập bằng email/mật khẩu, Google hoặc Facebook; quên và đặt lại mật khẩu qua email; đổi hoặc tạo mật khẩu; đăng xuất; hồ sơ cá nhân có ảnh đại diện, giới thiệu và thống kê.
- Ký ức: ghim lên bản đồ bằng tìm tên địa điểm hoặc chọn trực tiếp trên bản đồ; tiêu đề, câu chuyện, ngày, tối đa 10 ảnh và 1 video, tối đa 10 hashtag; chế độ riêng tư/công khai; sửa, xóa.
- Chuyến đi: gom ký ức theo thời gian, xem lộ trình trên bản đồ.
- Khám phá: bản đồ ký ức công khai của cộng đồng, lọc theo hashtag và 34 tỉnh/thành, tìm theo từ khóa; lọc ký ức cá nhân theo năm, tháng, khoảng ngày.
- Gợi ý hashtag bằng AI (Google Gemini), có phương án dự phòng khi không có khóa AI.
- Quản trị: ẩn/bỏ ẩn, xóa ký ức vi phạm; khóa/mở khóa tài khoản; nhật ký thao tác; thống kê.

**Giai đoạn 2 (chưa làm)**: tương tác cộng đồng gồm thích, bình luận, lưu "Muốn đi", theo dõi người dùng, báo cáo vi phạm và hàng đợi xử lý báo cáo (FR-4.1 đến FR-4.5, FR-4.7).

**Ngoài phạm vi**: ứng dụng di động native, nhắn tin trực tiếp, thanh toán, mức riêng tư "chỉ bạn bè", chia sẻ chuyến đi công khai.

## 1.3 Định nghĩa và viết tắt

| **Thuật ngữ** | **Ý nghĩa** |
|---|---|
| Ký ức | Bản ghi gắn với một tọa độ, gồm tiêu đề, nội dung, ngày, tên địa điểm, ảnh/video, hashtag và chế độ riêng tư/công khai. |
| Ghim (pin) | Điểm đánh dấu của một ký ức trên bản đồ. Ghim **xanh**: ký ức của mình (xanh đậm nếu riêng tư, xám nếu bị quản trị viên ẩn). Ghim **đỏ**: ký ức công khai của cộng đồng. Ghim **tím**: vị trí đang chọn để tạo ký ức mới, kéo được. |
| Chuyến đi | Tập hợp ký ức của cùng một người, xếp theo ngày ký ức. Một ký ức có thể thuộc nhiều chuyến đi. |
| Lộ trình | Đường nét đứt màu tím nối các ký ức của một chuyến đi theo thứ tự thời gian. |
| Hashtag | Nhãn chủ đề gắn với ký ức, lưu không có dấu "#", chữ thường. Nguồn `manual` (người dùng nhập) hoặc `ai` (chọn từ gợi ý AI). |
| Khu vực | Một trong 34 tỉnh/thành Việt Nam sau sắp xếp năm 2025. Hệ thống tự xác định từ tọa độ của ký ức. |
| Riêng tư / Công khai | Riêng tư: chỉ chủ ký ức xem được. Công khai: mọi người, kể cả khách chưa đăng nhập, xem được (nếu không bị ẩn và chủ không bị khóa). |
| Bị ẩn | Trạng thái do quản trị viên đặt (`memories.status = 'hidden'`). Người khác không thấy; chủ ký ức vẫn thấy kèm nhãn "Đã bị quản trị viên ẩn". |
| Xóa mềm | Đánh dấu `deleted_at` thay vì xóa dòng trong CSDL. Ký ức biến mất khỏi mọi màn hình và API ngay lập tức. |
| Khóa đăng nhập tạm thời | Sau 5 lần sai mật khẩu trong 15 phút, tài khoản không đăng nhập bằng mật khẩu được trong 15 phút. Khác với **tài khoản bị khóa** do quản trị viên. |
| JWT | Mã phiên đăng nhập (HS256) gửi kèm header `Authorization: Bearer`. Chứa id người dùng và `token_version`. |
| token_version | Số phiên bản phiên đăng nhập của tài khoản. Tăng khi đổi/đặt lại mật khẩu hoặc khi bị khóa; mọi JWT mang số cũ hết hiệu lực ngay. |
| URL có chữ ký | Đường dẫn ảnh/video dạng `/api/media/{id}?exp=…&sig=…`, ký bằng HMAC-SHA256, có hạn dùng. |
| Nominatim | Dịch vụ tìm địa điểm của OpenStreetMap: `/search` (tên → tọa độ) và `/reverse` (tọa độ → tên). |
| Gemini | Mô hình AI của Google dùng để gợi ý hashtag (mặc định `gemini-2.5-flash`). |
| Trigger | Thủ tục trong MySQL tự chạy khi thêm/sửa/xóa dòng. Hệ thống dùng 11 trigger làm lớp chặn thứ hai cho quy tắc nghiệp vụ. |
| AC | Tiêu chí nghiệm thu (Acceptance Criteria), viết theo Given-When-Then. |
| SRS / SDD | Tài liệu đặc tả yêu cầu / Tài liệu thiết kế hệ thống. |
| INVEST | Tiêu chí kiểm tra user story: Independent, Negotiable, Valuable, Estimable, Small, Testable. |
| MoSCoW | Mức ưu tiên: Must (bắt buộc), Should (nên có), Could (có thể có), Won't (không làm lần này). |

## 1.4 Tài liệu liên quan

| Tài liệu | Vị trí | Nội dung |
|---|---|---|
| Mã nguồn | https://github.com/Project-Memory-map/Memory_map | Code, README từng thư mục, hướng dẫn chạy |
| Lược đồ CSDL | [schema.sql](../schema.sql) (bản sao của `database/schema.sql` trong repo) | 11 bảng, 11 trigger, 1 view |
| Danh mục tỉnh/thành | `database/seed_regions.sql` trong repo | 34 tỉnh/thành theo Nghị quyết 202/2025/QH15 |
| Product Backlog | [ProductBacklog_BanDoKyUc.xlsx](../ProductBacklog_BanDoKyUc.xlsx) | 26 user story, 108 AC dạng bảng cho Scrum, đồng bộ bản 1.0, kèm trạng thái và kết quả kiểm thử |
| Báo cáo kiểm thử tích hợp | `tests/kiem_thu_tich_hop/BaoCao_KiemThuTichHop_108AC.xlsx` trong repo | Kết quả 108 AC trên MySQL thật, máy tính và điện thoại; danh sách 18 lỗi |
| Sơ đồ | [so-do/](../so-do/README.md) | 9 hình: use case, kiến trúc, ERD, luồng xử lý (kèm mã nguồn Mermaid) |
| Quy ước làm việc nhóm | `CONTRIBUTING.md` trong repo | Nhánh, commit, PR, quy tắc code |
| Chuẩn tham khảo | IEEE 830 / ISO/IEC/IEEE 29148 | Cấu trúc tài liệu |

## 1.5 Bố cục tài liệu

Mục 2 mô tả tổng quan sản phẩm và môi trường. Mục 3 nêu cách chia module và tiến độ. Mục 4 liệt kê yêu cầu chức năng kèm trạng thái. Mục 5 chi tiết hóa bằng 26 user story và 108 tiêu chí nghiệm thu. Mục 6 và 7 nêu giao diện ngoài và yêu cầu phi chức năng. Mục 8 tập hợp quy tắc nghiệp vụ. Mục 9 nối yêu cầu với story, test và kết quả. Mục 10 nêu hướng phát triển. Mục 11 ghi quyết định, điểm lệch so với bản 0.3 và vấn đề còn mở. Phụ lục A, B, C lần lượt là danh mục sơ đồ, danh mục API và từ điển dữ liệu.
