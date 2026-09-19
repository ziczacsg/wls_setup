# Task 08: Apache (mod_php)

**Loại:** Claude Code
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 8
**Phụ thuộc:** Task 05 (PHP đã cài)

## Mục đích

Cài Apache với `mod_php` (không dùng PHP-FPM) vì web app legacy phụ thuộc chỉ thị `php_value`/`php_flag`
trong `.htaccess` — chỉ hoạt động với mod_php. Dùng `mpm_prefork`, chạy bằng user `ubuntu`, giới hạn số
tiến trình.

## Các bước thực hiện

### 8.1. Cài đặt và bật module

```bash
[WSL, ubuntu, sudo]
sudo apt install -y --no-install-recommends apache2 libapache2-mod-php8.3

# mod_php bắt buộc dùng mpm_prefork (không tương thích mpm_event/worker)
sudo a2dismod mpm_event 2>/dev/null || true
sudo a2enmod mpm_prefork php8.3 rewrite headers expires
sudo a2dissite 000-default

echo "ServerName localhost" | sudo tee /etc/apache2/conf-available/servername.conf
sudo a2enconf servername

# Áp dụng php.ini dev (từ Task 05) cho SAPI của Apache
sudo ln -sf ../../mods-available/zz-dev.ini /etc/php/8.3/apache2/conf.d/99-zz-dev.ini

apache2ctl -M | grep -E 'mpm|php|rewrite'     # phải thấy mpm_prefork_module, php_module, rewrite_module
ls /etc/php/8.3/apache2/conf.d | grep -E 'mysql|mbstring|intl|redis|xdebug|zz-dev'
```

Nếu thiếu extension nào trong danh sách trên, bật lại cho SAPI Apache:

```bash
[WSL, ubuntu, sudo]
sudo phpenmod -v 8.3 -s apache2 pdo_mysql mysqli mbstring intl redis xdebug
```

### 8.2. Cổng, user chạy Apache và giới hạn tiến trình

```bash
[WSL, ubuntu, sudo]
# Chỉ dùng cổng cao (8080 ở bước này; 8443/8444 được thêm ở Task 09)
sudo tee /etc/apache2/ports.conf >/dev/null <<'EOF'
Listen 8080
EOF

# Chạy worker (kèm PHP) bằng user ubuntu:
#  - tránh 403 do /home/ubuntu quyền 750
#  - CLI (./flow) và web ghi file Flow cùng một user → không lỗi phân quyền
sudo sed -i \
  -e 's/^export APACHE_RUN_USER=.*/export APACHE_RUN_USER=ubuntu/' \
  -e 's/^export APACHE_RUN_GROUP=.*/export APACHE_RUN_GROUP=ubuntu/' \
  /etc/apache2/envvars

# Giới hạn số tiến trình prefork để tiết kiệm RAM
sudo tee /etc/apache2/conf-available/dev-prefork.conf >/dev/null <<'EOF'
<IfModule mpm_prefork_module>
    StartServers              2
    MinSpareServers           1
    MaxSpareServers           3
    MaxRequestWorkers        10
    MaxConnectionsPerChild  500
</IfModule>
KeepAliveTimeout 2
EOF
sudo a2enconf dev-prefork
```

### 8.3. Snippet dùng chung + virtual host Flow (tạm thời HTTP)

```bash
[WSL, ubuntu, sudo]
sudo mkdir -p /etc/apache2/snippets

sudo tee /etc/apache2/snippets/flow-app.conf >/dev/null <<'EOF'
DocumentRoot /home/ubuntu/projects/myproject/backend/Web

# Ngữ cảnh chạy của Flow: Development | Production | Testing ...
SetEnv FLOW_CONTEXT Development
SetEnv FLOW_REWRITEURLS 1

<Directory /home/ubuntu/projects/myproject/backend/Web>
    Options FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>

# Chỉ hoạt động với mod_php – dùng khi code legacy cần (bỏ dấu # nếu cần):
# php_admin_value memory_limit 768M
# php_flag short_open_tag on
EOF

sudo tee /etc/apache2/sites-available/flow.conf >/dev/null <<'EOF'
<VirtualHost *:8080>
    ServerName flow.localhost
    ServerAlias localhost
    Include /etc/apache2/snippets/flow-app.conf

    ErrorLog  ${APACHE_LOG_DIR}/flow-error.log
    CustomLog ${APACHE_LOG_DIR}/flow-access.log combined
</VirtualHost>
EOF

sudo a2ensite flow
sudo apache2ctl configtest
sudo systemctl enable apache2
sudo systemctl restart apache2      # đổi MPM/module phải RESTART, không dùng reload
```

## Kiểm tra (Verify)

```bash
[WSL, ubuntu]
apache2ctl -M | grep -E 'mpm_prefork|php_module|rewrite'
sudo apache2ctl configtest          # Syntax OK (cảnh báo DocumentRoot thiếu là bình thường trước Task 14)
systemctl is-active apache2
curl -sI http://localhost:8080 | head -3
```

## Ghi chú

- Đường dẫn `/home/ubuntu/projects/myproject/...` phải khớp thư mục dự án thật (Task 14). Khi thư mục
  chưa tồn tại, `configtest` chỉ cảnh báo, không lỗi — bình thường ở giai đoạn này.
- `AllowOverride All` là bắt buộc vì Flow dùng `Web/.htaccess` để rewrite URL, và cho phép
  `php_value`/`php_flag` trong `.htaccess`.
- Handler PHP đã được `mods-enabled/php8.3.conf` cấu hình sẵn, không cần `SetHandler` trong vhost.
- **Đánh đổi:** mỗi kết nối chiếm một tiến trình Apache có nạp sẵn PHP (~30–60 MB RAM khi đang xử lý). Đã
  giới hạn ở mục 8.2; nếu trang tải chậm/treo, tăng `MaxRequestWorkers` (vd 20).
