# Tiến độ triển khai — MKintai_Setup_WLS

Checklist theo dõi task nào đã thực thi **và kiểm chứng đạt** trên WSL này. Chỉ tick `[x]` sau khi mục
"Kiểm tra (Verify)" của task tương ứng trong `tasks_flow_dev/<file>.md` đã chạy và đạt kết quả mong đợi —
không tick trước khi verify xong. Thứ tự bám theo bảng trong [`README.md`](README.md).

## Giai đoạn 1 — dựng image dùng chung

- [x] 1. Cài Claude Code (bootstrap) — `04-claude-code-install_human.md` 🧑
- [x] 2. Cấu hình WSL nền tảng — `00-wsl-base-config.md`
- [x] 3. Restart WSL để nhận `wsl.conf` — `00b-wsl-restart_human.md` 🧑
- [x] 4. Cập nhật hệ thống & gói nền — `01-system-update-base-packages.md`
- [ ] 5. Git (cấu hình dùng chung) — `02-git-setup.md` ⏭️ tạm bỏ qua theo yêu cầu người dùng (2026-09-19) — cấu hình dùng chung (`init.defaultBranch`, `core.autocrlf`) thực ra đã đúng sẵn, nhưng `user.name`/`user.email` đã bị set global nên chưa tick — xem `.claude/task.md`
- [x] 6. Node.js 24.18 qua nvm — `03-nodejs-nvm.md`
- [x] 7. PHP 8.3 + Composer — `05-php83-composer.md`
- [x] 8. MySQL 8.4 — `06-mysql84.md`
- [x] 9. Redis — `07-redis.md`
- [x] 10. Apache (mod_php) — `08-apache-modphp.md`
- [x] 11. HTTPS mkcert — phần WSL — `09-https-mkcert-wsl.md` (đã cài `mkcert`, service `local-dev-certs` sinh cert tại `/etc/ssl/local-dev/`, Apache nghe 8443 (Flow) + `127.0.0.1:8444` (Adminer), 8080 redirect 301 → 8443, CA đã tin cậy trong kho hệ thống WSL + `NODE_EXTRA_CA_CERTS`; verify: `openssl s_client` → `Verify return code: 0 (ok)`, `curl http://localhost:8080` → 301 — 2026-09-19)
- [x] 12. HTTPS mkcert — tin cậy CA trên Windows — `09b-https-mkcert-trust-windows_human.md` 🧑 (người dùng xác nhận đã chạy `certutil -user -addstore Root` với `rootCA.pem` qua `\\wsl.localhost\...` và verify `https://localhost:8443` không còn cảnh báo `NET::ERR_CERT_AUTHORITY_INVALID` — 2026-09-19)
- [x] 13. Cấu hình Xdebug 3 — `10-xdebug-config.md` (`xdebug.ini` dev đã ghi đè + Apache restart, `xdebugctl` đã tạo, `.vscode/launch.json` tạo tại `~/projects/MKintai_Setup_WLS/.vscode/launch.json`; việc đặt breakpoint thực tế trong VSCode để lại Task 24/17-7)
- [x] 14. Adminer — `11-adminer.md` (cập nhật 2026-09-19 sau khi Task 11/mkcert hoàn tất: vhost đã chuyển về đúng thiết kế gốc HTTPS `127.0.0.1:8444` dùng chứng chỉ `/etc/ssl/local-dev/`, không còn dùng workaround HTTP 8080; verify: `curl -sk https://localhost:8444` → 200 OK)
- [x] 15. SSH server — phần WSL — `12-ssh-server-wsl.md`
- [ ] 16. Docker Engine (optional) — `13-docker-optional.md` ⏭️ bỏ qua theo yêu cầu người dùng (2026-09-19) — xem `.claude/task.md`
- [x] 17. Scaffold dự án monorepo — `14-project-scaffold-structure.md`
- [ ] 18. VSCode Remote - WSL — `15-vscode-remote-wsl_human.md` 🧑 ⏭️ chưa tick đủ theo yêu cầu người dùng (2026-09-19) — VSCode đã kết nối WSL, `xdebug.php-debug` + `Anthropic.claude-code` đã cài trong WSL (đủ để chạy Task 24); 5 extension còn lại trong `extensions.json` (Intelephense, Volar, ESLint, Prettier, EditorConfig) bị bỏ qua theo yêu cầu người dùng, không bắt buộc cài để tiếp tục
- [x] 19. Backend — package `MKintai.App` — `17-1-backend-scaffold-acme-app.md` (đã đổi tên package thật thành `MKintai.App` thay placeholder `Acme.App`, xác nhận với người dùng 2026-09-19)
- [x] 20. Frontend — mount Vue vào thẻ gốc — `17-2-frontend-vue-mount.md` (outDir trỏ tới package thật `MKintai.App`)
- [x] 21. Chạy Vite dev server — `17-3-run-vite-dev-server.md` (cập nhật 2026-09-19 sau Task 11/mkcert: khởi động lại, giờ chạy nền tại `https://localhost:5173` đúng thiết kế gốc, `devServer` trong `Settings.yaml` đổi lại thành `https://localhost:5173`)
- [x] 22. Kiểm chứng `/api/health` — `17-4-verify-backend-health.md` (cập nhật 2026-09-19 sau Task 11/mkcert: 6/6 trường đạt qua `https://localhost:8443/api/health`, kể cả `https: true`)
- [x] 23. Kiểm chứng Fluid → Vue trong trình duyệt — `17-5-verify-fluid-vue-browser_human.md` 🧑 (verify gốc 2026-09-19 qua `http://flow.localhost:8080/demo` lúc đang chạy HTTP, 7/7 mục đạt. Sau đó Task 11/mkcert hoàn tất cùng ngày, 8080 giờ redirect 301 sang `https://flow.localhost:8443/demo`; Task 12 [trust CA Windows] cũng đã xong nên không còn cảnh báo chứng chỉ — khuyến khích browser-check lại nhanh theo URL HTTPS mới nếu cần chắc chắn, không bắt buộc)
- [x] 24. Xdebug + VSCode — kiểm chứng — `17-7-verify-xdebug-vscode_human.md` 🧑 (verify gốc 2026-09-19 qua `http://localhost:8080/demo?XDEBUG_TRIGGER=1` lúc đang chạy HTTP, VSCode dừng đúng breakpoint. Sau Task 11/mkcert, URL đổi thành `https://flow.localhost:8443/demo?XDEBUG_TRIGGER=1`; Task 12 đã xong nên không còn cảnh báo CA — Xdebug không phụ thuộc protocol nên về lý thuyết vẫn hoạt động, chưa re-test lại qua HTTPS)
- [ ] 25. Docker + service Go — kiểm chứng — `17-8-verify-docker-go-service.md` ⏭️ bỏ qua theo yêu cầu người dùng (2026-09-19) — phụ thuộc Task 16 (Docker), xem `.claude/task.md`
- [x] 26. Commit & push scaffold — `17-10-commit-push-scaffold.md` (người dùng tự thực hiện commit `f0f7380 "init project"` + `git push` tới `origin` `https://github.com/ziczacsg/MKintai_Setup_WLS.git`; xác nhận `git status` sạch, `branch main` up to date với `origin/main` — 2026-09-19)
- [ ] 27. Dọn dẹp trước khi export — `18-pre-export-cleanup.md` ⚠️ hành động phá huỷ, cần xác nhận
- [ ] 28. Export WSL — `19-export-wsl_human.md` 🧑 ⏭️ bỏ qua theo yêu cầu người dùng (2026-09-19) — xem `.claude/task.md`

## Nhánh riêng — build production trên Windows host (sau Task 26)

- [ ] a. Build Vue 3 production trên Windows — `16-build-production-vue-windows_human.md` 🧑
- [ ] b. Đặt `devServer: ''` + flush cache — `17-6-toggle-production-build-config.md`
- [ ] c. Kiểm chứng bản build trong trình duyệt — `17-6b-verify-production-build_human.md` 🧑

## Giai đoạn 2 — mỗi member tự import về máy mình

- [ ] 1. Import WSL — `20-1-import-wsl_human.md` 🧑
- [ ] 2. Tin cậy CA trên Windows — `20-2-trust-ca-windows_human.md` 🧑
- [ ] 3. Thiết lập cá nhân — `20-3-post-import-setup_human.md` 🧑
- [ ] 4. Thêm SSH public key (tuỳ chọn) — `12b-ssh-add-member-key_human.md` 🧑
- [ ] 5. Git Credential Manager (tuỳ chọn) — `20-4-git-credential-manager_human.md` 🧑
- [ ] 6. Clone dự án, cài dependency, chạy thử — `20-5-open-project-run_human.md` 🧑

## Tuỳ chọn / tham khảo (không bắt buộc theo thứ tự)

- [ ] `.wslconfig` — giới hạn RAM/CPU — `21-wslconfig-tuning_human.md` 🧑
- [ ] Script cập nhật môi trường không cần export lại — `22-update-env-script.md`

---

**Quy ước:** 🧑 = task con người thực hiện (Claude Code không tự chạy). Task không có 🧑 mặc định giao
cho Claude Code. Khi bắt đầu một phiên làm việc mới, đọc file này trước để biết nên tiếp tục từ task nào.
