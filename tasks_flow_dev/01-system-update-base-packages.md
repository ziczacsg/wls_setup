# Task 01: Cập nhật hệ thống & gói nền

**Loại:** Claude Code
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 1
**Phụ thuộc:** Task 00b (human)

## Mục đích

Cập nhật toàn bộ gói hệ thống, cài các gói nền cơ bản, và bỏ các dịch vụ không cần thiết trên WSL để giảm
dung lượng và tăng tốc khởi động.

## Các bước thực hiện

```bash
[WSL, ubuntu, sudo]
sudo apt update && sudo apt full-upgrade -y

sudo apt install -y --no-install-recommends \
  ca-certificates curl gnupg lsb-release unzip zip rsync less nano

# Bỏ bớt thứ không cần trên WSL (giảm dung lượng, tăng tốc khởi động)
sudo apt purge -y snapd unattended-upgrades 2>/dev/null || true
sudo systemctl disable --now apt-daily.timer apt-daily-upgrade.timer 2>/dev/null || true
sudo systemctl mask systemd-networkd-wait-online.service   # tránh treo ~2 phút lúc boot
```

## Kiểm tra (Verify)

```bash
[WSL, ubuntu]
apt list --upgradable 2>/dev/null | grep -v '^Listing' | wc -l   # kỳ vọng: 0 (hoặc rất ít)
curl --version >/dev/null && echo "OK: curl"
```

## Ghi chú

- Luôn dùng `--no-install-recommends` cho mọi lệnh `apt install` trong toàn bộ quy trình (quy ước chung
  của tài liệu gốc).
- Không cài GUI/desktop package.
