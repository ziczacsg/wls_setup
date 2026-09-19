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
- [ ] 11. HTTPS mkcert — phần WSL — `09-https-mkcert-wsl.md` ⏭️ bỏ qua theo yêu cầu người dùng (2026-09-19) — xem `.claude/task.md`
- [ ] 12. HTTPS mkcert — tin cậy CA trên Windows — `09b-https-mkcert-trust-windows_human.md` 🧑 ⏭️ bỏ qua theo yêu cầu người dùng (2026-09-19)
- [ ] 13. Cấu hình Xdebug 3 — `10-xdebug-config.md`
- [ ] 14. Adminer — `11-adminer.md`
- [x] 15. SSH server — phần WSL — `12-ssh-server-wsl.md`
- [ ] 16. Docker Engine (optional) — `13-docker-optional.md` ⏭️ bỏ qua theo yêu cầu người dùng (2026-09-19) — xem `.claude/task.md`
- [x] 17. Scaffold dự án monorepo — `14-project-scaffold-structure.md`
- [ ] 18. VSCode Remote - WSL — `15-vscode-remote-wsl_human.md` 🧑
- [x] 19. Backend — package `MKintai.App` — `17-1-backend-scaffold-acme-app.md` (đã đổi tên package thật thành `MKintai.App` thay placeholder `Acme.App`, xác nhận với người dùng 2026-09-19)
- [x] 20. Frontend — mount Vue vào thẻ gốc — `17-2-frontend-vue-mount.md` (outDir trỏ tới package thật `MKintai.App`)
- [x] 21. Chạy Vite dev server — `17-3-run-vite-dev-server.md` (chạy nền tại `http://localhost:5173` — HTTP thay vì HTTPS vì Task 09/mkcert đã bị bỏ qua)
- [x] 22. Kiểm chứng `/api/health` — `17-4-verify-backend-health.md` (5/6 trường đạt: flowContext, php.version, php.sapi, php.xdebug, mysql, redis đều OK; `https: false` do Task 09/mkcert bị bỏ qua trước đó — kiểm tra qua `http://localhost:8080/api/health` thay vì `https://...:8443`)
- [ ] 23. Kiểm chứng Fluid → Vue trong trình duyệt — `17-5-verify-fluid-vue-browser_human.md` 🧑
- [ ] 24. Xdebug + VSCode — kiểm chứng — `17-7-verify-xdebug-vscode_human.md` 🧑
- [ ] 25. Docker + service Go — kiểm chứng — `17-8-verify-docker-go-service.md` ⏭️ bỏ qua theo yêu cầu người dùng (2026-09-19) — phụ thuộc Task 16 (Docker), xem `.claude/task.md`
- [ ] 26. Commit & push scaffold — `17-10-commit-push-scaffold.md` ⚠️ cần xác nhận trước `push`
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
