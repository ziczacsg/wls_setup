# Task 17-5 (human): Kiểm chứng kiến trúc Fluid → Vue trong trình duyệt

**Loại:** Con người (Human)
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 17.5
**Phụ thuộc:** Task 17-3 (Vite đang chạy), Task 17-4 (backend OK), Task 09b (CA đã tin cậy trên Windows)

## Vì sao cần con người thực hiện

Kiểm chứng này đòi hỏi quan sát trực quan trong trình duyệt trên Windows (ổ khoá HTTPS, DevTools Console,
Network tab, hành vi hot-reload khi sửa file) — không thể xác minh đầy đủ bằng script/CLI.

## Các bước thực hiện

Mở `https://flow.localhost:8443/demo` và kiểm tra lần lượt:

1. **Ổ khoá HTTPS** hợp lệ (không cảnh báo chứng chỉ).
2. Thấy nhãn **"Vue 3 đã mount ✔"** và danh sách 3 sản phẩm.
3. **Xem nguồn trang (Ctrl+U):** HTML chỉ có `<div data-vue-component="ProductList" data-props="...JSON...">`
   (rỗng) và hai thẻ `<script type="module">` trỏ tới `https://localhost:5173` → chứng minh Fluid chỉ tạo
   thẻ gốc, UI do Vue dựng.
4. **DevTools → Console:** có dòng `[vite] connected.` (WebSocket HMR chạy qua `wss://`).
5. **Hot-reload:** sửa chữ trong `frontend/src/components/ProductList.vue` (vd đổi "Vue 3 đã mount ✔") →
   trang tự cập nhật, **không** reload cả trang.
6. Bấm **"Tải lại từ API"** → tab Network có request `GET /api/products` trả JSON, danh sách vẫn hiển thị.
7. Mở `https://flow.localhost:8443/api/health` và `https://localhost:8444` (Adminer, đăng nhập bằng user
   `flow`).

## Tiêu chí hoàn thành

Cả 7 mục trên đều đạt như mô tả.

## Ghi chú

Nếu bất kỳ mục nào thất bại, tra bảng xử lý sự cố ở **Task 17-9 (tham khảo)**.
