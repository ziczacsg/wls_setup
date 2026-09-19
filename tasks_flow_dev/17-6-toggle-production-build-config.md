# Task 17-6: Chuyển sang bản build production (đặt `devServer: ''` + flush cache)

**Loại:** Claude Code
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 17.6 (bước 2)
**Phụ thuộc:** Task 16 (human) — đã có `app.js`/`app.css` trong
`backend/Packages/Application/Acme.App/Resources/Public/app/` qua robocopy

## Mục đích

Tắt Vite dev server, đổi cấu hình Development để Fluid nạp bản build tĩnh thay vì module từ Vite.

## Các bước thực hiện

1. Dừng tiến trình `npm run dev` (Task 17-3) nếu đang chạy nền.
2. Sửa `backend/Configuration/Development/Settings.yaml`:

```yaml
Acme:
  App:
    frontend:
      devServer: ''
```

3. Flush cache:

```bash
[WSL, ubuntu]
cd ~/projects/myproject/backend && ./flow flow:cache:flush
```

## Kiểm tra (Verify)

```bash
[WSL, ubuntu]
grep -A2 'frontend:' ~/projects/myproject/backend/Configuration/Development/Settings.yaml
ls ~/projects/myproject/backend/Packages/Application/Acme.App/Resources/Public/app/
```

Kỳ vọng thấy `devServer: ''` và các file `app.js`, `app.css` đã tồn tại (copy từ Task 16, human).

## Ghi chú

- Nếu chưa có bản build (Task 16 chưa chạy), `ViewHelper` sẽ báo lỗi không tìm thấy resource `app/app.js`
  — đó là hành vi mong đợi, không phải lỗi cấu hình.
- Sau khi con người kiểm chứng xong ở **Task 17-6b (human)**, cần trả lại
  `devServer: 'https://localhost:5173'` để tiếp tục dev (lặp lại bước 2 ở trên với giá trị ngược lại, rồi
  `flow:cache:flush`, rồi chạy lại Task 17-3).
