# Task 09: HTTPS cho môi trường local (mkcert) — phần trong WSL

**Loại:** Claude Code
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 9 (mục 9.1–9.3, 9.5)
**Phụ thuộc:** Task 08

## Mục đích

Dùng `mkcert` để tạo CA cục bộ và chứng chỉ cho `localhost`, `flow.localhost`, `adminer.localhost`,
`127.0.0.1`, `::1`. Mỗi máy tự sinh CA riêng lúc WSL khởi động lần đầu (service `local-dev-certs`) —
**không đóng gói CA/khoá riêng vào image**. Bật Apache SSL trên cổng 8443 (Flow, redirect từ 8080) và
127.0.0.1:8444 (dành cho Adminer, xem Task 11).

## Các bước thực hiện

### 9.1. Cài mkcert

```bash
[WSL, ubuntu, sudo]
sudo apt install -y --no-install-recommends mkcert || {
  # Dự phòng: tải binary chính thức
  sudo curl -fsSL "https://dl.filippo.io/mkcert/latest?for=linux/$(dpkg --print-architecture)" -o /usr/local/bin/mkcert
  sudo chmod +x /usr/local/bin/mkcert
}
mkcert -version
```

### 9.2. Service tự sinh chứng chỉ khi thiếu (chạy trước Apache)

```bash
[WSL, ubuntu, sudo]
sudo mkdir -p /etc/ssl/local-dev
sudo chown ubuntu:ubuntu /etc/ssl/local-dev

sudo tee /etc/systemd/system/local-dev-certs.service >/dev/null <<'EOF'
[Unit]
Description=Generate local dev TLS certificates (mkcert) if missing
Before=apache2.service
ConditionPathExists=!/etc/ssl/local-dev/local-dev.pem

[Service]
Type=oneshot
User=ubuntu
Environment=HOME=/home/ubuntu
Environment=CAROOT=/home/ubuntu/.local/share/mkcert
ExecStart=/usr/bin/env mkcert \
  -cert-file /etc/ssl/local-dev/local-dev.pem \
  -key-file  /etc/ssl/local-dev/local-dev-key.pem \
  localhost flow.localhost adminer.localhost 127.0.0.1 ::1
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
EOF

sudo mkdir -p /etc/systemd/system/apache2.service.d
sudo tee /etc/systemd/system/apache2.service.d/local-dev-certs.conf >/dev/null <<'EOF'
[Unit]
Wants=local-dev-certs.service
After=local-dev-certs.service
EOF

sudo systemctl daemon-reload
sudo systemctl enable local-dev-certs.service
sudo systemctl start local-dev-certs.service

ls -l /etc/ssl/local-dev                       # local-dev.pem  local-dev-key.pem
ls ~/.local/share/mkcert                       # rootCA.pem  rootCA-key.pem  (CA riêng của máy này)
```

### 9.3. Bật SSL cho Apache: 8443 (Flow) và redirect từ 8080

```bash
[WSL, ubuntu, sudo]
sudo a2enmod ssl

sudo tee /etc/apache2/ports.conf >/dev/null <<'EOF'
Listen 8080
Listen 8443
Listen 127.0.0.1:8444
EOF

sudo tee /etc/apache2/sites-available/flow.conf >/dev/null <<'EOF'
# HTTP: chuyển hướng toàn bộ sang HTTPS
<VirtualHost *:8080>
    ServerName flow.localhost
    ServerAlias localhost
    Redirect permanent / https://flow.localhost:8443/
</VirtualHost>

# HTTPS: ứng dụng Flow
<VirtualHost *:8443>
    ServerName flow.localhost
    ServerAlias localhost
    Include /etc/apache2/snippets/flow-app.conf

    SSLEngine on
    SSLCertificateFile    /etc/ssl/local-dev/local-dev.pem
    SSLCertificateKeyFile /etc/ssl/local-dev/local-dev-key.pem

    ErrorLog  ${APACHE_LOG_DIR}/flow-error.log
    CustomLog ${APACHE_LOG_DIR}/flow-access.log combined
</VirtualHost>
EOF

sudo apache2ctl configtest
sudo systemctl restart apache2
ss -tlnp | grep -E ':(8080|8443)\b'
```

### 9.5. Tin cậy CA trong WSL + kiểm tra

```bash
[WSL, ubuntu, sudo]
sudo install -m 644 "$HOME/.local/share/mkcert/rootCA.pem" /usr/local/share/ca-certificates/flow-dev-local-ca.crt
sudo update-ca-certificates

# Node.js không dùng kho CA của hệ thống → khai báo riêng
cat >> ~/.bashrc <<'EOF'

# Tin CA mkcert cục bộ cho Node.js (chỉ khi file đã tồn tại)
[ -f "$HOME/.local/share/mkcert/rootCA.pem" ] && export NODE_EXTRA_CA_CERTS="$HOME/.local/share/mkcert/rootCA.pem"
EOF
source ~/.bashrc
```

## Kiểm tra (Verify)

```bash
[WSL, ubuntu]
openssl s_client -connect localhost:8443 -servername flow.localhost \
  -CAfile ~/.local/share/mkcert/rootCA.pem </dev/null 2>/dev/null \
  | grep -E 'subject=|Verify return code'
# Kỳ vọng: Verify return code: 0 (ok)

curl -sI http://localhost:8080 | head -3        # 301 → https://flow.localhost:8443/
```

## Ghi chú

- Việc **import CA vào kho chứng chỉ của Windows** (để Chrome/Edge tin cậy) là bước riêng, phải chạy trên
  Windows host và có hộp thoại xác nhận — xem **Task 09b (human)**.
- Đây là ứng dụng Flow chưa có (sẽ tạo ở Task 14/17) nên chỉ kiểm tra bằng `openssl` ở bước này.
