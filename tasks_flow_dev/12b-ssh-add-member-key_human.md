# Task 12b (human): Thêm SSH public key của member (per-máy, sau khi import)

**Loại:** Con người (Human)
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 12 (mục 12.4)
**Phụ thuộc:** Task 12, Task 20-1 (import WSL, human) — chỉ áp dụng cho từng member sau khi họ đã import
distro về máy mình, **không** chạy trong lúc dựng image dùng chung.

## Vì sao cần con người thực hiện

Đây là thao tác **cá nhân của từng member**, chạy từ **PowerShell trên máy Windows của họ**, tạo/sử dụng
SSH keypair riêng và đẩy public key vào WSL của chính họ — không phải một bước dựng image dùng chung, và
không nên tự động hoá bằng Claude Code chạy trong image chung (sẽ làm lộ/nhầm key giữa các máy).

## Các bước thực hiện

Trên **PowerShell** (Windows), sau khi đã import distro `flow-dev` (Task 20-1):

```powershell
[host]
# Tạo key nếu chưa có
ssh-keygen -t ed25519

# Đẩy public key vào WSL
type $env:USERPROFILE\.ssh\id_ed25519.pub | wsl -d flow-dev -u ubuntu -- bash -c "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

Tạo/sửa `C:\Users\<user>\.ssh\config`:

```
Host flow-dev
    HostName localhost
    Port 2222
    User ubuntu
    IdentityFile ~/.ssh/id_ed25519
```

## Tiêu chí hoàn thành

```powershell
[host]
ssh flow-dev            # kết nối thành công bằng public key
sftp flow-dev            # phải báo lỗi "subsystem request failed" (SFTP đã tắt — đúng như thiết kế)
```

## Ghi chú

Sau khi mọi member đã thêm public key, có thể cân nhắc đổi `PasswordAuthentication no` trong
`/etc/ssh/sshd_config.d/00-dev.conf` (Task 12) để chỉ cho phép đăng nhập bằng key.
