# Danh sách task — Setup Dev Flow Framework + Vue 3 trên WSL (Ubuntu 24.04)

Breakdown từ [`../WSL-DEV-ENVIRONMENT-GUIDE.md`](../WSL-DEV-ENVIRONMENT-GUIDE.md) (bản 1.2) thành các task
nhỏ, độc lập, giao được cho **Claude Code** chạy trong WSL. Tên file có hậu tố `_human` là task **bắt buộc
con người thực hiện** (thao tác trên Windows host, hộp thoại xác nhận GUI, đăng nhập/nhập thông tin cá
nhân, hoặc chính là bước bootstrap cài Claude Code). Mọi file **không** có hậu tố `_human` mặc định giao
cho Claude Code chạy trong WSL.

Ký hiệu ngữ cảnh lệnh dùng trong từng task:

| Ký hiệu | Ý nghĩa |
|---|---|
| `[WSL, ubuntu]` | Chạy trong terminal WSL, user `ubuntu` |
| `[WSL, ubuntu, sudo]` | Như trên, cần `sudo` |
| `[host]` | Chạy trong PowerShell trên Windows |
| *(không có ký hiệu, mô tả bằng lời)* | Thao tác GUI (VSCode, trình duyệt, hộp thoại Windows) |

## Vai trò & tiền đề

- **Tiền đề chung:** WSL2 (Ubuntu 24.04, `noble`) đã được cài trên máy — bản thân việc cài WSL **không**
  nằm trong bộ task này.
- **Task 04 (human)** là bootstrap: cài Claude Code trong WSL. Đây là task **duy nhất bắt buộc chạy thủ
  công trước tiên**, vì Claude Code chưa tồn tại để nhận các task khác. Sau Task 04, mọi task Claude Code
  còn lại có thể giao tuần tự cho `claude` chạy trong WSL.
- Số thứ tự trong tên file bám theo **số "Bước" của tài liệu gốc** để dễ đối chiếu — **không** phải lúc
  nào cũng là thứ tự thực hiện thực tế (xem cột "Thứ tự" trong bảng dưới, đặc biệt Task 04 chạy đầu tiên
  dù mang số 04).

## Thứ tự thực hiện (giai đoạn 1 — dựng image dùng chung)

| # | Task | Loại | Nguồn (Bước) | Phụ thuộc |
|---|---|---|---|---|
| 1 | [Cài Claude Code (bootstrap)](04-claude-code-install_human.md) | 🧑 Human | 4 | — |
| 2 | [Cấu hình WSL nền tảng](00-wsl-base-config.md) | 🤖 Claude | 0.1–0.5 | 1 |
| 3 | [Restart WSL để nhận `wsl.conf`](00b-wsl-restart_human.md) | 🧑 Human | 0.3 | 2 |
| 4 | [Cập nhật hệ thống & gói nền](01-system-update-base-packages.md) | 🤖 Claude | 1 | 3 |
| 5 | [Git (cấu hình dùng chung)](02-git-setup.md) | 🤖 Claude | 2 | 4 |
| 6 | [Node.js 24.18 qua nvm](03-nodejs-nvm.md) | 🤖 Claude | 3 | 4 |
| 7 | [PHP 8.3 + Composer](05-php83-composer.md) | 🤖 Claude | 5 | 4 |
| 8 | [MySQL 8.4](06-mysql84.md) | 🤖 Claude | 6 | 4 |
| 9 | [Redis](07-redis.md) | 🤖 Claude | 7 | 4 |
| 10 | [Apache (mod_php)](08-apache-modphp.md) | 🤖 Claude | 8 | 7 |
| 11 | [HTTPS mkcert — phần WSL](09-https-mkcert-wsl.md) | 🤖 Claude | 9.1–9.3, 9.5 | 10 |
| 12 | [HTTPS mkcert — tin cậy CA trên Windows](09b-https-mkcert-trust-windows_human.md) | 🧑 Human | 9.4 | 11 |
| 13 | [Cấu hình Xdebug 3](10-xdebug-config.md) | 🤖 Claude | 10.1–10.3 | 10 |
| 14 | [Adminer](11-adminer.md) | 🤖 Claude | 11 | 11, 8 |
| 15 | [SSH server — phần WSL](12-ssh-server-wsl.md) | 🤖 Claude | 12.1–12.3 | 4 |
| 16 | [Docker Engine (optional)](13-docker-optional.md) | 🤖 Claude | 13 | 4 |
| 17 | [Scaffold dự án monorepo](14-project-scaffold-structure.md) | 🤖 Claude | 14 | 6, 7, 10 |
| 18 | [VSCode Remote - WSL](15-vscode-remote-wsl_human.md) | 🧑 Human | 15 | 17 |
| 19 | [Backend — package `Acme.App`](17-1-backend-scaffold-acme-app.md) | 🤖 Claude | 17.1 | 17 |
| 20 | [Frontend — mount Vue vào thẻ gốc](17-2-frontend-vue-mount.md) | 🤖 Claude | 17.2 | 17 |
| 21 | [Chạy Vite dev server](17-3-run-vite-dev-server.md) | 🤖 Claude | 17.3 | 20 |
| 22 | [Kiểm chứng `/api/health`](17-4-verify-backend-health.md) | 🤖 Claude | 17.4 | 19, 11, 8, 9, 13 |
| 23 | [Kiểm chứng Fluid → Vue trong trình duyệt](17-5-verify-fluid-vue-browser_human.md) | 🧑 Human | 17.5 | 21, 22, 12 |
| 24 | [Xdebug + VSCode — kiểm chứng](17-7-verify-xdebug-vscode_human.md) | 🧑 Human | 17.7 | 13, 18, 19 |
| 25 | [Docker + service Go — kiểm chứng](17-8-verify-docker-go-service.md) | 🤖 Claude | 17.8 | 16, 17 |
| 26 | [Commit & push scaffold](17-10-commit-push-scaffold.md) | 🤖 Claude ⚠️ cần xác nhận trước `push` | 17.10 | 19, 20, 21, 22 |
| — | [Bảng xử lý sự cố khi kiểm chứng](17-9-troubleshooting-reference.md) | 📖 Tham khảo | 17.9 | — |
| 27 | [Dọn dẹp trước khi export](18-pre-export-cleanup.md) | 🤖 Claude ⚠️ hành động phá huỷ, cần xác nhận | 18 | 26 |
| 28 | [Export WSL](19-export-wsl_human.md) | 🧑 Human | 19 | 27 |

## Nhánh riêng — build production trên Windows host (chạy sau khi có Task 26)

| # | Task | Loại | Nguồn (Bước) | Phụ thuộc |
|---|---|---|---|---|
| a | [Build Vue 3 production trên Windows](16-build-production-vue-windows_human.md) | 🧑 Human | 16 | 26 (repo đã push), 6 |
| b | [Đặt `devServer: ''` + flush cache](17-6-toggle-production-build-config.md) | 🤖 Claude | 17.6 (bước 2) | a |
| c | [Kiểm chứng bản build trong trình duyệt](17-6b-verify-production-build_human.md) | 🧑 Human | 17.6 (bước 3) | b |

## Thứ tự thực hiện (giai đoạn 2 — mỗi member tự import về máy mình)

| # | Task | Loại | Nguồn (Bước) | Phụ thuộc |
|---|---|---|---|---|
| 1 | [Import WSL](20-1-import-wsl_human.md) | 🧑 Human | 20.1 | Task 28 (image đã export) |
| 2 | [Tin cậy CA trên Windows](20-2-trust-ca-windows_human.md) | 🧑 Human | 20.2 | 1 |
| 3 | [Thiết lập cá nhân (mật khẩu, git identity, login Claude Code)](20-3-post-import-setup_human.md) | 🧑 Human | 20.3 | 2 |
| 4 | [Thêm SSH public key (tuỳ chọn)](12b-ssh-add-member-key_human.md) | 🧑 Human | 12.4 | 3 |
| 5 | [Git Credential Manager (tuỳ chọn)](20-4-git-credential-manager_human.md) | 🧑 Human | 20.4 | 3 |
| 6 | [Clone dự án, cài dependency, chạy thử](20-5-open-project-run_human.md) | 🧑 Human (khởi động) → 🤖 Claude (các lệnh cài đặt) | 20.5 | 3 |

## Tài liệu tham khảo dùng chung / tuỳ chọn

- [Hướng dẫn thiết lập môi trường Dev (gộp Giai đoạn 2 + build production + export/update image)](guide_setup_dev_wls.md) —
  dùng khi clear/import lại WSL hoặc onboard member mới, đã điền sẵn giá trị thật của dự án
- [Bảng xử lý sự cố (Task 17)](17-9-troubleshooting-reference.md) — Bước 17.9
- [`.wslconfig` — giới hạn RAM/CPU](21-wslconfig-tuning_human.md) — Phụ lục A (tuỳ chọn, per-máy)
- [Script cập nhật môi trường không cần export lại](22-update-env-script.md) — Phụ lục B (tuỳ chọn)

## Ghi chú vận hành quan trọng

- ⚠️ **Task 17-10** (git push) và **Task 18** (xoá dữ liệu/lịch sử trong WSL) đều được đánh dấu cần **xác
  nhận rõ ràng của người dùng** trước khi Claude Code thực thi, dù về mặt phân loại chúng là task Claude
  Code — lý do nằm trong từng file (push code hiển thị với người khác; xoá dữ liệu không thể hoàn tác).
- Toàn bộ bảng "Lưu ý quan trọng khi export/import" (32 mục) trong tài liệu gốc không được tách thành task
  riêng — đây là tài liệu tra cứu khi gặp sự cố cụ thể, xem trực tiếp
  `../WSL-DEV-ENVIRONMENT-GUIDE.md` § *Lưu ý quan trọng khi export/import*.
- Checklist tóm tắt (Phụ lục D) và lịch sử thay đổi tài liệu (Phụ lục E) trong file gốc là nội dung tổng
  hợp lại các task ở trên — không tạo task riêng.
