# Task 03: Node.js 24.18 qua nvm

**Loại:** Claude Code
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 3
**Phụ thuộc:** Task 01

## Mục đích

Cài `nvm` và Node.js **24.18.x** trong WSL. Node ở đây chỉ dùng cho Vite dev server (hot-reload) và các
script hỗ trợ — build production chạy trên Windows host (xem Task 16, human).

## Các bước thực hiện

```bash
[WSL, ubuntu]
# Kiểm tra phiên bản nvm mới nhất tại https://github.com/nvm-sh/nvm/releases trước khi chạy
curl -fsSL https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash

# Nạp nvm vào shell hiện tại
export NVM_DIR="$HOME/.nvm"
. "$NVM_DIR/nvm.sh"

# Xác nhận 24.18.x có tồn tại rồi cài
nvm ls-remote 24 | grep -E 'v24\.18\.'
nvm install 24.18
nvm alias default 24.18
nvm use default

node -v     # v24.18.x
npm -v
```

Ghi lại đúng phiên bản patch để dùng cho `.nvmrc` (Task 14) và cho Windows host (Task 16, human):

```bash
[WSL, ubuntu]
node -v | sed 's/^v//'      # ví dụ 24.18.0
```

## Kiểm tra (Verify)

```bash
[WSL, ubuntu]
export NVM_DIR="$HOME/.nvm"; . "$NVM_DIR/nvm.sh"
node -v
npm -v
```

## Ghi chú

- **Mẹo:** nvm ghi vào `~/.bashrc` sau đoạn "return nếu non-interactive", nên phiên SSH không tương
  tác/VSCode task có thể không thấy `node`. Nếu gặp `node: command not found`, chuyển 3 dòng nvm lên
  **đầu** `~/.bashrc`.
- Ghi nhớ đúng phiên bản patch (vd `24.18.0`) — cần khớp chính xác với Node cài trên Windows host ở Task 16.
