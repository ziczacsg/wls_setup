# Task 00: Cấu hình WSL nền tảng (wsl.conf, sudo, thư mục dự án)

**Loại:** Claude Code
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 0 (mục 0.1, 0.2, 0.4, 0.5)
**Phụ thuộc:** — (chạy đầu tiên, sau khi Task 04 đã cài xong Claude Code)

> Ký hiệu lệnh: `[WSL, ubuntu]` = chạy trong shell WSL với user `ubuntu`; `[WSL, ubuntu, sudo]` = cần sudo.

## Mục đích

Xác nhận đúng bản Ubuntu 24.04, ghi `/etc/wsl.conf` (bật systemd, user mặc định `ubuntu`, hostname
`flow-dev`), bật sudo không mật khẩu cho dev, đặt mật khẩu tạm, và tạo thư mục dự án.

## Các bước thực hiện

### 0.1. Xác nhận đúng Ubuntu 24.04

```bash
[WSL, ubuntu]
. /etc/os-release && [ "$VERSION_ID" = "24.04" ] && echo "OK: Ubuntu 24.04" || echo "SAI: cần Ubuntu 24.04 (hiện là $VERSION_ID)"
```

Nếu in ra "SAI", dừng lại và báo cho người dùng — không tiếp tục các task sau vì toàn bộ tài liệu giả định
Ubuntu 24.04 (noble).

### 0.2. Tạo `/etc/wsl.conf`

```bash
[WSL, ubuntu, sudo]
sudo tee /etc/wsl.conf >/dev/null <<'EOF'
[boot]
systemd=true

[user]
default=ubuntu

[network]
hostname=flow-dev
generateResolvConf=true

[interop]
enabled=true
appendWindowsPath=true
EOF
```

### 0.4. Sudo không cần mật khẩu (dev) & mật khẩu user tạm

```bash
[WSL, ubuntu, sudo]
echo 'ubuntu ALL=(ALL) NOPASSWD:ALL' | sudo tee /etc/sudoers.d/90-ubuntu-dev
sudo chmod 440 /etc/sudoers.d/90-ubuntu-dev

# Mật khẩu tạm (dùng cho SSH) – member PHẢI đổi sau khi import (xem Task 20-3, human)
echo 'ubuntu:changeme' | sudo chpasswd
```

### 0.5. Tạo thư mục dự án

```bash
[WSL, ubuntu]
mkdir -p ~/projects
```

## Kiểm tra (Verify)

```bash
[WSL, ubuntu]
cat /etc/wsl.conf
sudo -n true && echo "OK: sudo không cần mật khẩu"
ls -d ~/projects
```

## Ghi chú

- 📌 Luôn để mã nguồn trong `/home/ubuntu/...`, **không** để trong `/mnt/c/...` (hiệu năng I/O, quyền file,
  inotify, symlink).
- `/etc/wsl.conf` chỉ có hiệu lực đầy đủ sau khi restart WSL — xem **Task 00b (human)**.
- Đây là cấu hình DEV. `NOPASSWD:ALL` và mật khẩu mặc định `changeme` **không** dùng cho production.
