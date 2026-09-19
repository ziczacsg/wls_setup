# Task 12: SSH server (cổng 2222, không SFTP) — phần trong WSL

**Loại:** Claude Code
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 12 (mục 12.1–12.3)
**Phụ thuộc:** Task 01

## Mục đích

Cài SSH server dùng cho shell từ xa (cổng **2222**, không dùng 22 để tránh xung đột giữa các distro WSL2
dùng chung một VM/mạng). **SFTP bị tắt hoàn toàn**; truyền file dùng `\\wsl.localhost\...` hoặc git.

## Các bước thực hiện

### 12.1. Cài và cấu hình (cổng 2222, không SFTP)

```bash
[WSL, ubuntu, sudo]
sudo apt install -y --no-install-recommends openssh-server

sudo tee /etc/ssh/sshd_config.d/00-dev.conf >/dev/null <<'EOF'
Port 2222
PermitRootLogin no
AllowUsers ubuntu
PubkeyAuthentication yes
PasswordAuthentication yes
ClientAliveInterval 60
EOF

# Tắt SFTP: vô hiệu hoá dòng "Subsystem sftp ..." trong cấu hình chính
sudo sed -i -E 's/^(Subsystem[[:space:]]+sftp)/#\1/' /etc/ssh/sshd_config
```

### 12.2. Ubuntu 24.04 dùng `ssh.socket` → phải tắt để đổi cổng có hiệu lực

```bash
[WSL, ubuntu, sudo]
sudo systemctl disable --now ssh.socket
sudo systemctl daemon-reload
sudo systemctl enable --now ssh.service

sudo sshd -t && ss -tlnp | grep 2222
sudo sshd -T | grep -i '^subsystem' || echo "OK: không có subsystem SFTP"
```

### 12.3. Tự sinh lại SSH host key sau khi import

```bash
[WSL, ubuntu, sudo]
sudo tee /etc/systemd/system/ssh-hostkeys-regen.service >/dev/null <<'EOF'
[Unit]
Description=Regenerate SSH host keys if missing
Before=ssh.service
ConditionPathExistsGlob=!/etc/ssh/ssh_host_*_key

[Service]
Type=oneshot
ExecStart=/usr/bin/ssh-keygen -A

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable ssh-hostkeys-regen.service
```

## Kiểm tra (Verify)

```bash
[WSL, ubuntu]
sudo sshd -t
ss -tlnp | grep 2222
sudo sshd -T | grep -i '^subsystem' || echo "OK: không có subsystem SFTP"
systemctl is-enabled ssh-hostkeys-regen.service
```

## Ghi chú

- Có thể đổi `PasswordAuthentication no` khi mọi member đã thêm public key (xem Task 12b, human).
- 📝 **VSCode Remote - WSL không cần SSH** (nó dùng `wsl.exe` trực tiếp). SSH ở đây dành cho ai muốn dùng
  **Remote - SSH** hoặc shell từ xa thuần tuý.
- `scp` mặc định của OpenSSH hiện đại dùng giao thức SFTP nên sẽ không chạy khi SFTP đã tắt (dùng
  `scp -O` cho giao thức cũ, hoặc tốt hơn dùng `\\wsl.localhost\...`).
