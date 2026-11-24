# Sơ đồ: Bản đồ Ký ức

[← Trang chính](../README.md) · Văn bản: [srs/](../srs/README.md)

Phiên bản 1.0 · 24/11/2026. Cả 9 hình được vẽ lại theo sản phẩm đã cài đặt (repo commit `442abf3`).

- [Sơ đồ use case](01-use-case.md)
  - Hình 1: Sơ đồ use case tổng quát (GĐ1)
- [Kiến trúc và phụ thuộc module](02-kien-truc.md)
  - Hình 2: Sơ đồ kiến trúc tổng thể (3 lớp)
  - Hình 3: Sơ đồ phụ thuộc module (Core, M1–M4)
- [Sơ đồ thực thể liên kết (ERD)](03-erd.md)
  - Hình 4: ERD Module Tài khoản
  - Hình 5: ERD Module Ký ức, Chuyến đi, Bản đồ
  - Hình 6: ERD Module Quản trị
- [Luồng xử lý](04-luong-xu-ly.md)
  - Hình 7: Luồng xử lý tạo ký ức kèm ảnh
  - Hình 8: Luồng xử lý đăng nhập bằng Google
  - Hình 9: Luồng xử lý xem ghim công khai trong vùng nhìn bản đồ

## Sửa hình

Mỗi hình có mã nguồn Mermaid trong [src/](src/) (`hinh-1.mmd` … `hinh-9.mmd`). Sửa file `.mmd` rồi xuất lại PNG bằng mermaid-cli, ví dụ:

```bash
npx -p @mermaid-js/mermaid-cli@11.4.2 mmdc -i src/hinh-5.mmd -o images/hinh-5.png -w 1800 -s 2 -b white
```

Hình 1 dùng bộ dàn trang ELK (khai báo ở đầu file), các hình còn lại dùng mặc định. Ký tự `#` trong chữ của sơ đồ tuần tự phải viết thành `#35;`.
