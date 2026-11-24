[← Trang chính](../README.md) · Sơ đồ: [so-do/](../so-do/README.md)

# Tài liệu đặc tả yêu cầu phần mềm (SRS): Bản đồ Ký ức

| **Phiên bản**   | **1.0 (bản hoàn thiện theo sản phẩm đã cài đặt)**                                                                                         |
|-----------------|-------------------------------------------------------------------------------------------------------------------------------------------|
| Ngày            | 24/11/2026                                                                                                                                |
| Nhóm thực hiện  | 4 thành viên                                                                                                                              |
| Ngôn ngữ        | Tiếng Việt                                                                                                                                |
| Mã nguồn        | https://github.com/Project-Memory-map/Memory_map (nhánh `master`, commit `442abf3` ngày 21/11/2026)                                       |
| Tài liệu đi kèm | [schema.sql](../schema.sql) (lược đồ cuối); Product Backlog (.xlsx); [so-do/](../so-do/README.md); báo cáo kiểm thử tích hợp 108 AC (trong repo) |

**Lịch sử thay đổi**

| **Phiên bản** | **Ngày**   | **Nội dung** |
|---------------|------------|--------------|
| 0.1           | 01/11/2026 | Bản nháp đầu tiên, soạn từ các câu trả lời ban đầu của nhóm. |
| 0.2           | 02/11/2026 | CSDL chốt MySQL; tương tác cộng đồng chuyển sang GĐ2. Bổ sung sơ đồ use case, 26 user story kèm tiêu chí nghiệm thu, quy tắc nghiệp vụ, yêu cầu phi chức năng, ma trận truy vết, hướng phát triển. |
| 0.3           | 04/11/2026 | Thêm BR-17 đến BR-22 (URL ảnh có chữ ký, liên kết OAuth chỉ khi email đã xác thực, giới hạn Nominatim, thu hồi JWT bằng token_version, chống XSS, xóa EXIF/GPS); chốt các mục \[TBD\]. |
| **1.0**       | 24/11/2026 | **Viết lại toàn bộ theo sản phẩm đã hoàn thành** (26/26 user story, 103/108 AC đạt kiểm thử tích hợp, 5 AC còn chờ khóa OAuth thật). Gỡ hết nhãn "(đề xuất)", ghi giá trị thực tế của mọi giới hạn. Cập nhật các điểm bản cài đặt khác bản 0.3: định dạng ảnh (bỏ .heic, thêm .webp/.gif), xóa EXIF không dùng Pillow, gán khu vực không dùng shapely, thời hạn JWT 24 giờ, thời hạn URL media 1 giờ, cách liên kết tài khoản Google/Facebook. FR-3.6 (AI gợi ý hashtag) đã làm. Thêm yêu cầu có trong sản phẩm nhưng chưa có trong 0.3 (thống kê, lộ trình chuyến đi, nhật ký quản trị...). Thêm Phụ lục B (danh mục API), Phụ lục C (từ điển dữ liệu). Vẽ lại cả 9 sơ đồ theo code (kèm mã nguồn Mermaid trong `so-do/src/`); đồng bộ Product Backlog (.xlsx). Mục 11 liệt kê các điểm lệch và vấn đề còn mở, trong đó có lỗi form ký ức không lưu hashtag/chuyến đi phát hiện khi soạn bản này. |

## Mục lục

- [1. Giới thiệu](01-gioi-thieu.md)
  - 1.1 Mục đích · 1.2 Phạm vi · 1.3 Định nghĩa và viết tắt · 1.4 Tài liệu liên quan · 1.5 Bố cục tài liệu
- [2. Mô tả tổng quan](02-mo-ta-tong-quan.md)
  - 2.1 Bối cảnh sản phẩm · 2.2 Chức năng chính · 2.3 Đối tượng và vai trò · 2.4 Sơ đồ use case · 2.5 Môi trường vận hành và công nghệ · 2.6 Cấu hình · 2.7 Ràng buộc · 2.8 Giả định và phụ thuộc
- [3. Tổ chức module và tiến độ thực hiện](03-phan-chia-cong-viec.md)
- [4. Yêu cầu chức năng](04-yeu-cau-chuc-nang.md)
  - 4.1 Tài khoản, xác thực, hồ sơ · 4.2 Ký ức, chuyến đi, bản đồ · 4.3 Khám phá, bộ lọc, tìm kiếm · 4.4 Quản trị và cộng đồng
- [5. Đặc tả User Story và tiêu chí nghiệm thu](05-user-stories/README.md)
  - [5.3 EP1 Tài khoản và hồ sơ](05-user-stories/ep1-tai-khoan.md) · [5.4 EP2 Ký ức, chuyến đi, bản đồ](05-user-stories/ep2-ky-uc-chuyen-di-ban-do.md) · [5.5 EP3 Khám phá, tìm kiếm](05-user-stories/ep3-kham-pha-tim-kiem.md) · [5.6 EP4 Quản trị](05-user-stories/ep4-quan-tri.md)
- [6. Yêu cầu giao diện ngoài](06-yeu-cau-giao-dien-ngoai.md)
  - 6.1 Giao diện người dùng · 6.2 Giao diện phần mềm (REST API) · 6.3 Dịch vụ bên ngoài · 6.4 Giao diện phần cứng
- [7. Yêu cầu phi chức năng](07-yeu-cau-phi-chuc-nang.md)
  - 7.1 Hiệu năng · 7.2 Bảo mật · 7.3 Tương thích và dễ dùng · 7.4 Độ tin cậy, bảo trì, kiểm thử
- [8. Quy tắc nghiệp vụ](08-quy-tac-nghiep-vu.md)
- [9. Ma trận truy vết và kết quả kiểm thử](09-ma-tran-truy-vet.md)
- [10. Hướng phát triển](10-huong-phat-trien.md)
- [11. Quyết định, điểm lệch so với bản 0.3 và vấn đề còn mở](11-quyet-dinh-va-van-de-mo.md)
- [Phụ lục A: Danh mục sơ đồ](phu-luc-a-danh-muc-so-do.md)
- [Phụ lục B: Danh mục API](phu-luc-b-danh-muc-api.md)
- [Phụ lục C: Từ điển dữ liệu](phu-luc-c-tu-dien-du-lieu.md)

## Quy ước

| Nhãn | Ý nghĩa |
|---|---|
| **GĐ1** | Phạm vi phiên bản này, đã cài đặt. |
| **GĐ2** | Giai đoạn sau, chưa cài đặt. |
| ✅ Đạt | Đã cài đặt và đạt kiểm thử (pytest và/hoặc kiểm thử tích hợp 108 AC). |
| ⚠️ Một phần | Đã cài đặt nhưng chưa đủ so với yêu cầu, hoặc có điểm lệch; chi tiết ở mục 11. |
| ⏳ Chưa làm | Thuộc GĐ1 nhưng chưa cài đặt, hoặc là quy trình vận hành chưa thực hiện. |
| 🔧 Cần cấu hình | Code và test giả lập đã đạt, chỉ thiếu khóa dịch vụ ngoài để chạy luồng thật. |

Các con số giới hạn trong tài liệu (độ dài, dung lượng, thời hạn, số lượng) lấy đúng theo mã nguồn và `schema.sql`. Khi sửa một con số trong code, cập nhật cả mục 8 (quy tắc nghiệp vụ) và story liên quan.
