# Task 17-6b (human): Kiểm chứng bản build production trong trình duyệt

**Loại:** Con người (Human)
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 17.6 (bước 3)
**Phụ thuộc:** Task 17-6

## Vì sao cần con người thực hiện

Cần xem nguồn trang trong trình duyệt trên Windows để xác nhận đúng thẻ `<link>`/`<script>` được nạp.

## Các bước thực hiện

Tải lại `https://flow.localhost:8443/demo`: xem nguồn trang phải thấy `<link ... app.css>` và
`<script type="module" ... app.js>` trỏ tới `_Resources/Static/Packages/Acme.App/app/...`, và trang hoạt
động như trước (danh sách sản phẩm, nút "Tải lại từ API" hoạt động).

## Tiêu chí hoàn thành

Trang hiển thị đúng bằng bản build tĩnh, không còn phụ thuộc Vite dev server đang chạy.

## Ghi chú

Sau khi xác nhận xong, báo lại để chạy phần "trả lại `devServer`" ở cuối **Task 17-6** và khởi động lại
**Task 17-3** để tiếp tục dev với hot-reload.
