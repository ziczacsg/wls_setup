# Tham khảo: Bảng xử lý khi kiểm chứng Task 17 thất bại

**Loại:** Tài liệu tham khảo (không phải task hành động — dùng khi Task 17-4/17-5/17-6b/17-7 thất bại)
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 17.9

| Triệu chứng | Nguyên nhân thường gặp | Cách xử lý |
|---|---|---|
| Trình duyệt báo `NET::ERR_CERT_AUTHORITY_INVALID` | CA chưa import vào Windows / chưa khởi động lại trình duyệt | Task 09b (human): `certutil`, đóng hẳn trình duyệt; Firefox bật `security.enterprise_roots.enabled` |
| Trang `/demo` trống, Console: `Failed to load module script` / `ERR_CERT...` tại `localhost:5173` | Vite chưa chạy, hoặc trình duyệt chưa tin chứng chỉ ở `localhost:5173` | Kiểm tra Task 17-3 đang chạy; mở `https://localhost:5173/@vite/client` |
| Console: lỗi CORS khi nạp module | Origin trang không khớp regex `cors.origin` | Dùng `flow.localhost` hoặc `localhost`; kiểm tra `vite.config.ts` (Task 17-2) |
| Có `[vite] connected` nhưng sửa file không cập nhật | Sửa file ngoài WSL / watcher | Sửa bằng VSCode Remote - WSL (Task 15); kiểm tra file nằm trong `/home/ubuntu` |
| `/api/health` → `mysql.ok=false` | Sai mật khẩu/DB, MySQL chưa chạy | `systemctl status mysql`; kiểm tra `Settings.yaml` (Task 06, 14) |
| `/api/health` → `redis.ok=false` | Redis chưa chạy hoặc `redis` chưa nạp cho Apache | `redis-cli ping`; `sudo phpenmod -v 8.3 -s apache2 redis && sudo systemctl restart apache2` |
| `sapi` = `cli` hoặc lỗi 500 với `php_value` | mod_php chưa bật / đang dùng MPM khác | Task 08: `apache2ctl -M \| grep -E "mpm\|php"` |
| Flow báo `Access Denied` | Policy chặn action | Kiểm tra `Policy.yaml` của package |
| Lỗi `Exception` của Flow | — | Xem `backend/Data/Logs/Exceptions/` và `Data/Logs/System_Development.log` |
| `403 Forbidden` | Quyền thư mục / user Apache | `APACHE_RUN_USER=ubuntu` (Task 08), `AllowOverride All` |

Bảng xử lý sự cố tổng quát hơn (áp dụng cho export/import) nằm ở **Task References — xem README.md, mục
"Lưu ý quan trọng khi export/import"** (nguồn: `WSL-DEV-ENVIRONMENT-GUIDE.md` § Lưu ý quan trọng khi
export/import).
