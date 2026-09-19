# Task 16 (human): Build production Vue 3 trên Windows host

**Loại:** Con người (Human)
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 16
**Phụ thuộc:** Task 03 (biết đúng phiên bản Node patch), Task 14 (frontend đã khởi tạo, đã push lên Git —
Task 17-10)

## Vì sao cần con người thực hiện

Theo thiết kế của tài liệu, `npm run build` **không chạy trong WSL** — chạy trên Windows host, vì
Vite/esbuild/rollup cài binary riêng theo hệ điều hành. Việc này đòi hỏi cài Node trên Windows (nvm-windows)
và clone riêng một bản trên `C:\build\...`, ngoài phạm vi của Claude Code chạy trong WSL.

## Các bước thực hiện

### 16.1. Chuẩn bị Node trên Windows

Dùng [nvm-windows](https://github.com/coreybutler/nvm-windows), **cùng phiên bản patch** với WSL (lấy từ
kết quả Task 03, vd `24.18.0`):

```powershell
[host]
nvm install 24.18.0        # dùng ĐÚNG phiên bản ghi trong frontend/.nvmrc
nvm use 24.18.0
node -v
```

### 16.2. Build từ bản clone riêng phía Windows

```powershell
[host]
# Một lần
git clone <URL_REPO> C:\build\myproject

# Mỗi lần build
cd C:\build\myproject\frontend
git pull
npm ci
npm run build
# → sinh ra C:\build\myproject\backend\Packages\Application\Acme.App\Resources\Public\app\app.js, app.css
```

### 16.3. Đưa bản build sang WSL để thử chế độ production

```powershell
[host]
robocopy "C:\build\myproject\backend\Packages\Application\Acme.App\Resources\Public\app" `
         "\\wsl.localhost\flow-dev\home\ubuntu\projects\myproject\backend\Packages\Application\Acme.App\Resources\Public\app" /MIR
```

## Tiêu chí hoàn thành

- `app.js` và `app.css` xuất hiện trong `Resources/Public/app/` cả ở `C:\build\...` và trong WSL (qua
  `\\wsl.localhost\...`).

## Ghi chú

- Sau bước này, chuyển sang **Task 17-6** (Claude Code đặt `devServer: ''` + flush cache) rồi
  **Task 17-6b (human)** để kiểm chứng trong trình duyệt.
- **Không** chạy `npm ci` trên `node_modules` mà WSL đang dùng qua `\\wsl.localhost\...` — luôn dùng bản
  clone riêng ở `C:\build\...`.
