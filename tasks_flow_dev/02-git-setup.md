# Task 02: Cài đặt Git (cấu hình dùng chung)

**Loại:** Claude Code
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 2
**Phụ thuộc:** Task 01

## Mục đích

Cài Git và thiết lập các cấu hình **dùng chung cho image** (không phải identity cá nhân).

## Các bước thực hiện

```bash
[WSL, ubuntu, sudo]
sudo apt install -y --no-install-recommends git

git config --global init.defaultBranch main
git config --global core.autocrlf input      # luôn commit LF
git config --global core.editor nano
git --version
```

## Kiểm tra (Verify)

```bash
[WSL, ubuntu]
git --version
git config --global --get init.defaultBranch   # main
git config --global --get core.autocrlf        # input
```

## Ghi chú quan trọng

- ❌ **KHÔNG** cấu hình `git config --global user.name` / `user.email` trong task này — đây là image dùng
  chung cho cả team. Mỗi member tự đặt identity của mình sau khi import (xem **Task 20-3, human**).
