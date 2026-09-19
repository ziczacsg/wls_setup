# Task 10: Cấu hình Xdebug 3 cho Apache/CLI

**Loại:** Claude Code
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 10 (mục 10.1–10.3)
**Phụ thuộc:** Task 08

## Mục đích

Cấu hình Xdebug 3 để VSCode (chạy Remote - WSL, lắng nghe `127.0.0.1:9003` **trong WSL**) có thể bắt
breakpoint. Vì VSCode và PHP đều chạy trong WSL, `client_host` luôn là `127.0.0.1` cho mọi member.

## Các bước thực hiện

### 10.1. Cấu hình Xdebug 3

```bash
[WSL, ubuntu, sudo]
sudo tee /etc/php/8.3/mods-available/xdebug.ini >/dev/null <<'EOF'
zend_extension=xdebug.so
xdebug.mode=debug
xdebug.start_with_request=trigger
xdebug.client_host=127.0.0.1
xdebug.client_port=9003
xdebug.discover_client_host=false
xdebug.idekey=VSCODE
xdebug.log_level=0
EOF

sudo systemctl restart apache2     # mod_php nạp Xdebug khi Apache khởi động
php -v                             # CLI: thấy "with Xdebug v3.x"
apache2ctl -M | grep php           # web: php_module
```

### 10.2. Script bật/tắt Xdebug nhanh

```bash
[WSL, ubuntu, sudo]
sudo tee /usr/local/bin/xdebugctl >/dev/null <<'EOF'
#!/usr/bin/env bash
case "$1" in
  on)  sudo phpenmod  -v 8.3 xdebug ;;
  off) sudo phpdismod -v 8.3 xdebug ;;
  *)   php -m | grep -qi xdebug && echo "Xdebug: ON" || echo "Xdebug: OFF"; exit 0 ;;   # trạng thái theo CLI
esac
sudo systemctl restart apache2
echo "Xdebug: $1"
EOF
sudo chmod +x /usr/local/bin/xdebugctl
```

Dùng: `xdebugctl on|off|status`.

### 10.3. `.vscode/launch.json` (đặt trong repo dự án)

Tạo tại `~/projects/myproject/.vscode/launch.json` (thư mục dự án sẽ tồn tại từ Task 14; nếu chưa có, tạo
trước `mkdir -p ~/projects/myproject/.vscode`):

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "PHP: Listen for Xdebug",
      "type": "php",
      "request": "launch",
      "port": 9003
    }
  ]
}
```

(Không cần `pathMappings` vì VSCode và PHP cùng chạy trong WSL, đường dẫn giống nhau.)

## Kiểm tra (Verify)

```bash
[WSL, ubuntu]
php -v | grep -i xdebug             # "with Xdebug v3.x"
xdebugctl status
cat ~/projects/myproject/.vscode/launch.json
```

## Ghi chú

- `start_with_request=trigger` → Xdebug chỉ kết nối khi có "trigger", nên gần như không tốn tài nguyên khi
  không debug.
- Kích hoạt debug: thêm `?XDEBUG_TRIGGER=1` vào URL trình duyệt, hoặc `XDEBUG_TRIGGER=1 ./flow ...` cho
  CLI, hoặc dùng extension *Xdebug Helper* (IDE key: `VSCODE`).
- Việc kiểm chứng thực tế (đặt breakpoint, F5, dừng đúng chỗ) cần thao tác trong VSCode/trình duyệt — xem
  **Task 17-7 (human)**.
- ⚠️ Flow biên dịch *proxy class* (AOP/DI) vào `Data/Temporary/<Context>/Cache/Code/`. Với class bị proxy
  (vd controller có `#[Flow\Inject]`), breakpoint đặt ở file nguồn có thể không dừng như mong đợi — đặt
  breakpoint ở class thuần khi cần chắc chắn (xem Task 17-7).
