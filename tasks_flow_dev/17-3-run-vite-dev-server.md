# Task 17-3: Chạy Vite dev server (hot-reload) trong WSL

**Loại:** Claude Code
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 17.3
**Phụ thuộc:** Task 17-2

## Mục đích

Khởi động Vite dev server ở `https://localhost:5173`, giữ chạy nền để phục vụ hot-reload cho trang Fluid.

## Các bước thực hiện

```bash
[WSL, ubuntu]
cd ~/projects/myproject/frontend
export NVM_DIR="$HOME/.nvm"; . "$NVM_DIR/nvm.sh"
nvm use          # đọc .nvmrc

npm run dev      # Vite lắng nghe https://localhost:5173
```

> Claude Code nên chạy lệnh này **ở chế độ nền** (background) vì nó là tiến trình giữ chạy liên tục, rồi
> xác nhận log khởi động thành công (`Local: https://localhost:5173/`) trước khi coi task hoàn tất.

## Kiểm tra (Verify)

```bash
[WSL, ubuntu]
curl -sk -o /dev/null -w '%{http_code}\n' https://localhost:5173/@vite/client   # kỳ vọng: 200
```

## Ghi chú

- Việc mở trình duyệt lần đầu tại `https://localhost:5173/@vite/client` để trình duyệt tin chứng chỉ là
  thao tác người dùng, thuộc phạm vi **Task 09b (human)** / **Task 17-5 (human)**.
- Vite dev server cần chạy **song song** với Apache trong suốt các bước kiểm chứng còn lại của Task 17
  (17-4, 17-5) cho tới khi chuyển sang thử bản build production (Task 17-6).
