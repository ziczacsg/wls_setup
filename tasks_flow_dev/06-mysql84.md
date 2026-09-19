# Task 06: MySQL 8.4

**Loại:** Claude Code
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 6
**Phụ thuộc:** Task 01

## Mục đích

Cài MySQL 8.4 LTS (từ kho APT chính thức của MySQL — Ubuntu 24.04 chỉ có 8.0 mặc định), cấu hình nhẹ cho
dev, và tạo database/user cho Flow.

## Các bước thực hiện

### 6.1. Thêm kho MySQL 8.4 LTS

```bash
[WSL, ubuntu, sudo]
curl -fsSL https://repo.mysql.com/RPM-GPG-KEY-mysql-2023 \
  | sudo gpg --dearmor -o /usr/share/keyrings/mysql.gpg

echo "deb [signed-by=/usr/share/keyrings/mysql.gpg] http://repo.mysql.com/apt/ubuntu noble mysql-8.4-lts" \
  | sudo tee /etc/apt/sources.list.d/mysql.list

sudo apt update
apt-cache policy mysql-community-server     # Candidate phải là 8.4.x
```

> Nếu `apt update` báo `NO_PUBKEY` (MySQL đã đổi khoá ký), lấy khoá mới tại
> <https://dev.mysql.com/doc/refman/8.4/en/checking-gpg-signature.html>. Phương án thay thế: tải gói
> `mysql-apt-config` từ <https://dev.mysql.com/downloads/repo/apt/> rồi chọn **MySQL 8.4 LTS**.

### 6.2. Cài đặt

```bash
[WSL, ubuntu, sudo]
sudo DEBIAN_FRONTEND=noninteractive apt install -y --no-install-recommends mysql-community-server

mysql --version                 # mysql  Ver 8.4.x
sudo systemctl enable --now mysql
sudo systemctl is-active mysql
```

Mặc định `root` đăng nhập qua `auth_socket` → dùng `sudo mysql`.

### 6.3. Cấu hình nhẹ cho dev

```bash
[WSL, ubuntu, sudo]
sudo tee /etc/mysql/conf.d/99-dev.cnf >/dev/null <<'EOF'
[mysqld]
bind-address                   = 127.0.0.1
mysqlx                         = OFF
character-set-server           = utf8mb4
collation-server                = utf8mb4_unicode_ci
performance_schema             = OFF
max_connections                = 50
innodb_buffer_pool_size        = 256M
innodb_flush_log_at_trx_commit = 2

# MySQL 8.4 mặc định TẮT mysql_native_password. Chỉ bật nếu client cũ bắt buộc:
# mysql_native_password        = ON
EOF

sudo systemctl restart mysql
```

### 6.4. Tạo database & user cho Flow

```bash
[WSL, ubuntu, sudo]
sudo mysql <<'SQL'
CREATE DATABASE flow_dev  CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE DATABASE flow_test CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'flow'@'%' IDENTIFIED BY 'flow_dev_pw';
GRANT ALL PRIVILEGES ON flow_dev.*  TO 'flow'@'%';
GRANT ALL PRIVILEGES ON flow_test.* TO 'flow'@'%';
FLUSH PRIVILEGES;
SQL
```

## Kiểm tra (Verify)

```bash
[WSL, ubuntu]
mysql -h 127.0.0.1 -u flow -pflow_dev_pw -e "SELECT VERSION(); SHOW DATABASES;"
```

Kỳ vọng thấy phiên bản `8.4.x` và hai database `flow_dev`, `flow_test` trong danh sách.

## Ghi chú

- `'flow'@'%'` an toàn vì MySQL chỉ bind `127.0.0.1`. Dùng `'%'` để client trên Windows (DBeaver/HeidiSQL
  qua `localhost:3306`) vẫn kết nối được dù WSL forward từ IP khác.
- Mật khẩu `flow_dev_pw` chỉ dùng cho DEV, không dùng dữ liệu thật trong `flow_dev`/`flow_test`.
