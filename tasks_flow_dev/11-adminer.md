# Task 11: Adminer

**Loại:** Claude Code
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 11
**Phụ thuộc:** Task 09 (HTTPS/mkcert đã sẵn sàng), Task 06 (MySQL)

## Mục đích

Cài Adminer (1 file PHP duy nhất, không cài phpMyAdmin), chạy qua Apache/mod_php, chỉ lắng nghe
`127.0.0.1:8444` (HTTPS, dùng chung chứng chỉ ở Task 09).

## Các bước thực hiện

```bash
[WSL, ubuntu, sudo]
sudo mkdir -p /var/www/adminer
sudo curl -fsSL https://www.adminer.org/latest-mysql-en.php -o /var/www/adminer/index.php
sudo chown -R ubuntu:ubuntu /var/www/adminer

sudo tee /etc/apache2/sites-available/adminer.conf >/dev/null <<'EOF'
<VirtualHost *:8444>
    ServerName adminer.localhost
    DocumentRoot /var/www/adminer

    SSLEngine on
    SSLCertificateFile    /etc/ssl/local-dev/local-dev.pem
    SSLCertificateKeyFile /etc/ssl/local-dev/local-dev-key.pem

    <Directory /var/www/adminer>
        Require all granted
    </Directory>

    ErrorLog  ${APACHE_LOG_DIR}/adminer-error.log
    CustomLog ${APACHE_LOG_DIR}/adminer-access.log combined
</VirtualHost>
EOF

sudo a2ensite adminer
sudo apache2ctl configtest && sudo systemctl reload apache2
```

## Kiểm tra (Verify)

```bash
[WSL, ubuntu]
curl -sI https://localhost:8444 | head -1       # HTTP/1.1 200 OK  (không cần -k nếu Task 09 đã xong)
```

Truy cập từ trình duyệt (thao tác người dùng, không bắt buộc để coi task này "done" cho Claude Code):
<https://localhost:8444> → System: **MySQL** · Server: `127.0.0.1` · User: `flow` · Password:
`flow_dev_pw` · DB: `flow_dev`.

## Ghi chú

- Không mở cổng 8444 ra ngoài (đã bind `127.0.0.1`). Adminer không nên bị truy cập từ mạng LAN.
- `curl -sI https://localhost:8444` cũng gián tiếp xác nhận HTTPS + mod_php + CA đều đúng.
