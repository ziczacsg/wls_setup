# Task 20-1 (human): Import WSL (dành cho member)

**Loại:** Con người (Human)
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 20 (mục 20.1)
**Phụ thuộc:** Task 19 (image `.tar.gz` + checksum đã được phát hành)

## Vì sao cần con người thực hiện

`wsl --import` chạy từ PowerShell trên máy Windows **của từng member**, trước khi có bất kỳ WSL/Claude Code
nào tồn tại trên máy đó để giao việc — bắt buộc thao tác thủ công, tương tự Task 04 nhưng ở phía member.

## Các bước thực hiện

```powershell
[host]
# Cập nhật WSL (cần ≥ 2.0 để có systemd)
wsl --update
wsl --version

# Kiểm tra checksum
Get-FileHash .\flow-dev-1.0.0.tar.gz -Algorithm SHA256

# Chọn thư mục cài trên ổ SSD nội bộ, đường dẫn ngắn, KHÔNG có dấu/khoảng trắng, KHÔNG nằm trong OneDrive
mkdir D:\WSL\flow-dev
wsl --import flow-dev D:\WSL\flow-dev .\flow-dev-1.0.0.tar.gz --version 2

wsl -l -v
wsl -d flow-dev
```

Kiểm tra vào phải là user `ubuntu` (nhờ `/etc/wsl.conf`). Nếu vào bằng `root`:

```powershell
[host]
wsl -d flow-dev -u ubuntu
# rồi kiểm tra /etc/wsl.conf có [user] default=ubuntu, sau đó: wsl --terminate flow-dev
```

## Tiêu chí hoàn thành

- SHA256 khớp với checksum được phát hành ở Task 19.
- `wsl -d flow-dev` vào bằng user `ubuntu`.

## Ghi chú

Nếu trùng tên distro (đã có `flow-dev` từ trước), đặt tên khác khi import (`wsl --import flow-dev-2 ...`),
thư mục cài riêng cho mỗi distro.
