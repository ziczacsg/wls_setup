# Task 05: PHP 8.3 + Composer

**Loại:** Claude Code
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 5
**Phụ thuộc:** Task 01

## Mục đích

Cài PHP 8.3 CLI + các extension cần cho Flow Framework, tinh chỉnh `php.ini` cho dev, và cài Composer.

## Các bước thực hiện

### 5.1. Cài PHP 8.3 (CLI) và extension

```bash
[WSL, ubuntu, sudo]
sudo apt install -y --no-install-recommends \
  php8.3-cli php8.3-common php8.3-opcache php8.3-readline \
  php8.3-mysql php8.3-mbstring php8.3-xml php8.3-curl php8.3-zip \
  php8.3-intl php8.3-gd php8.3-bcmath php8.3-redis php8.3-xdebug

php -v
php -m | grep -Ei 'mbstring|pdo_mysql|intl|xml|gd|redis|xdebug|opcache'
```

> **Không cài** metapackage `php` và **không cài** `php8.3-fpm`. Module Apache
> (`libapache2-mod-php8.3`) được cài ở Task 08, kèm bước kiểm tra extension đã được nạp cho Apache.

### 5.2. Tinh chỉnh `php.ini` cho dev (CLI và Apache mod_php)

```bash
[WSL, ubuntu, sudo]
sudo tee /etc/php/8.3/mods-available/zz-dev.ini >/dev/null <<'EOF'
; --- Dev overrides cho Flow Framework ---
date.timezone = UTC                ; đổi nếu cần, vd: Asia/Ho_Chi_Minh
memory_limit = 512M
max_execution_time = 120
upload_max_filesize = 64M
post_max_size = 64M
max_input_vars = 5000
error_reporting = E_ALL
display_errors = On

opcache.enable = 1
opcache.enable_cli = 0
opcache.memory_consumption = 128
opcache.validate_timestamps = 1
opcache.revalidate_freq = 0

; --- Tuỳ chọn cho code legacy (bỏ dấu ; ở đầu dòng nếu cần) ---
; short_open_tag = On
; error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT
EOF

# CLI: liên kết ngay. SAPI của Apache (mod_php) sẽ được liên kết ở Task 08
sudo ln -sf ../../mods-available/zz-dev.ini /etc/php/8.3/cli/conf.d/99-zz-dev.ini
```

### 5.4. Composer

```bash
[WSL, ubuntu]
curl -fsSL https://getcomposer.org/installer -o /tmp/composer-setup.php
EXPECTED="$(curl -fsSL https://composer.github.io/installer.sig)"
ACTUAL="$(php -r "echo hash_file('sha384', '/tmp/composer-setup.php');")"

if [ "$EXPECTED" = "$ACTUAL" ]; then
  sudo php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer
else
  echo "Checksum KHÔNG khớp – dừng lại!"
fi
rm -f /tmp/composer-setup.php

composer --version
```

## Kiểm tra (Verify)

```bash
[WSL, ubuntu]
php -v                              # PHP 8.3.x (cli)
php -m | grep -Ei 'mbstring|pdo_mysql|intl|xml|gd|redis|xdebug|opcache'
composer --version
```

## Ghi chú

- **Cách PHP chạy trong môi trường này:**
  - **CLI** (`php`, `composer`, `./flow ...`): dùng `/etc/php/8.3/cli/php.ini` + `conf.d/`.
  - **Web**: PHP chạy **bên trong tiến trình Apache** (`mod_php`), dùng `/etc/php/8.3/apache2/php.ini` +
    `conf.d/`. Không có pool/socket PHP-FPM.
  - User chạy web là user của Apache — đặt là `ubuntu` ở Task 08 để CLI và web cùng user, tránh lỗi phân
    quyền khi Flow ghi vào `Data/`.
- Hai môi trường (CLI và Apache) có `php.ini` **riêng**. Đổi cấu hình cho web phải kiểm tra thư mục
  `apache2` và chạy `sudo systemctl restart apache2` (xem Task 08).
- Nếu cần xử lý ảnh mạnh hơn, có thể thêm `php8.3-imagick` (tuỳ chọn, không có trong danh sách gốc).
