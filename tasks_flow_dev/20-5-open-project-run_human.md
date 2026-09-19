# Task 20-5 (human): Clone dự án, cài dependency, chạy thử

**Loại:** Con người (Human) để khởi động ban đầu; sau đó các lệnh cụ thể **có thể giao cho Claude Code**
(xem Ghi chú)
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 20 (mục 20.5)
**Phụ thuộc:** Task 20-3, Task 15 (VSCode Remote - WSL), Task 17-10 (repo skeleton đã tồn tại trên Git)

## Các bước thực hiện

```bash
[WSL, ubuntu]
cd ~/projects
git clone <URL_REPO> myproject
cd myproject
code .
```

```bash
[WSL, ubuntu]
cd ~/projects/myproject/backend && composer install && ./flow doctrine:migrate
```

```bash
[WSL, ubuntu]
cd ~/projects/myproject/frontend && nvm use && npm ci && npm run dev
```

Trình duyệt: `https://flow.localhost:8443/demo`; kiểm tra thêm `https://flow.localhost:8443/api/health`.

## Tiêu chí hoàn thành

Trang `/demo` hiển thị đúng (giống Task 17-5), `/api/health` trả `ok` cho mọi trường.

## Ghi chú

- `git clone` (cần URL_REPO) và mở VSCode (`code .`) nên do member tự chạy lần đầu, vì gắn với danh tính
  Git cá nhân (Task 20-3) và trải nghiệm GUI của họ.
- Sau khi đã đăng nhập Claude Code (Task 20-3, bước 5), các lệnh `composer install`,
  `./flow doctrine:migrate`, `npm ci`, `npm run dev` **có thể giao lại cho Claude Code** chạy trong WSL của
  chính member đó — đây là các lệnh thuần shell, không khác gì Task 14/17 đã chạy khi dựng image gốc.
