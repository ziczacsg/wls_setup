# Task 18: Tối ưu kích thước & dọn dẹp trước khi export

**Loại:** Claude Code (⚠️ hành động **phá huỷ dữ liệu trong chính WSL này** — xin xác nhận người dùng
trước khi chạy script, xem Ghi chú)
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 18
**Phụ thuộc:** Task 17-10 (scaffold đã push lên Git)

## Mục đích

Dọn Docker, dừng dịch vụ, xoá cache gói/ngôn ngữ, xoá log, và **quan trọng nhất: xoá mọi thông tin cá
nhân/bí mật/định danh riêng của máy** (CA, SSH host key & key cá nhân, Claude Code login, git identity,
lịch sử shell, VSCode server) để image có thể chia sẻ an toàn cho cả team.

## Các bước thực hiện

Lưu script vào `infra/scripts/pre-export-cleanup.sh`:

```bash
[WSL, ubuntu]
mkdir -p ~/projects/myproject/infra/scripts
cat > ~/projects/myproject/infra/scripts/pre-export-cleanup.sh <<'SCRIPT'
#!/usr/bin/env bash
# Dọn dẹp WSL trước khi export. Chạy bằng user ubuntu.
set -u

echo "==> [1/8] Dọn Docker (image/volume/cache build)"
if command -v docker >/dev/null 2>&1; then
  sudo systemctl start docker 2>/dev/null || true
  sudo docker system prune -af --volumes 2>/dev/null || true
fi

echo "==> [2/8] Dừng dịch vụ để dữ liệu được flush sạch"
sudo systemctl stop apache2 mysql redis-server ssh ssh.socket \
                    docker docker.socket containerd 2>/dev/null || true

echo "==> [3/8] Dọn cache gói"
sudo apt-get clean
sudo apt-get autoremove --purge -y
sudo rm -rf /var/lib/apt/lists/*

echo "==> [4/8] Dọn cache ngôn ngữ (npm, composer, ...)"
export NVM_DIR="$HOME/.nvm"; [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
command -v npm >/dev/null && npm cache clean --force
command -v composer >/dev/null && composer clear-cache
rm -rf ~/.cache/* ~/.npm/_logs

echo "==> [5/8] Xoá log & file tạm"
sudo journalctl --rotate 2>/dev/null; sudo journalctl --vacuum-time=1s 2>/dev/null
sudo rm -rf /var/log/journal/*
sudo find /var/log -type f \( -name '*.gz' -o -name '*.[0-9]' -o -name '*.old' \) -delete
sudo find /var/log -type f -exec truncate -s 0 {} \;
sudo rm -rf /tmp/* /var/tmp/*

echo "==> [6/8] Xoá CA & chứng chỉ HTTPS (mỗi máy sẽ tự sinh CA riêng khi khởi động lần đầu)"
sudo rm -f /etc/ssl/local-dev/*.pem
rm -rf ~/.local/share/mkcert
sudo rm -f /usr/local/share/ca-certificates/flow-dev-local-ca.crt
sudo update-ca-certificates --fresh >/dev/null 2>&1

echo "==> [7/8] Xoá thông tin cá nhân / bí mật / định danh riêng của máy"
rm -rf ~/.claude ~/.claude.json
rm -f  ~/.ssh/id_* ~/.ssh/authorized_keys ~/.ssh/known_hosts
sudo rm -f /etc/ssh/ssh_host_*
git config --global --unset user.name  2>/dev/null
git config --global --unset user.email 2>/dev/null
rm -f ~/.git-credentials
rm -f ~/.bash_history ~/.mysql_history ~/.lesshst ~/.viminfo ~/.node_repl_history ~/.rediscli_history
rm -rf ~/.vscode-server
sudo rm -f /var/lib/mysql/auto.cnf

echo "==> [8/8] Ghi dấu phiên bản image"
echo "flow-dev image 1.0.0 - $(date -I)" | sudo tee /etc/flow-dev-release >/dev/null

echo
echo "Dung lượng các thư mục lớn nhất:"
sudo du -xh --max-depth=1 / 2>/dev/null | sort -h | tail -8
echo
echo "XONG. Chạy:  unset HISTFILE; exit   rồi export từ PowerShell."
SCRIPT

chmod +x ~/projects/myproject/infra/scripts/pre-export-cleanup.sh
```

**Sau khi người dùng xác nhận (xem Ghi chú), chạy:**

```bash
[WSL, ubuntu]
~/projects/myproject/infra/scripts/pre-export-cleanup.sh
```

Kiểm tra thủ công trước khi thoát:

```bash
[WSL, ubuntu]
grep -i 'ANTHROPIC_API_KEY' ~/.bashrc ~/.profile 2>/dev/null || echo "OK: không có ANTHROPIC_API_KEY"
ls -d ~/projects/myproject 2>/dev/null && echo "⚠️ Chưa xoá ~/projects/myproject — xem bước tiếp theo"
```

Xoá thư mục dự án khỏi image (đã push lên Git ở Task 17-10, member sẽ `git clone` lại sau khi import —
xem Task 20-5):

```bash
[WSL, ubuntu]
rm -rf ~/projects/myproject
```

Cuối cùng:

```bash
[WSL, ubuntu]
unset HISTFILE
exit
```

## Kiểm tra (Verify)

- Script chạy hết cả 8 bước, không lỗi nghiêm trọng.
- `~/.claude`, `~/.claude.json`, `~/.ssh/id_*`, `/etc/ssh/ssh_host_*`, `~/.local/share/mkcert`,
  `~/projects/myproject` đều không còn tồn tại.
- `~/.bashrc`/`~/.profile` không còn `ANTHROPIC_API_KEY` hay mật khẩu/token cá nhân.

## Ghi chú quan trọng

- ⚠️ **Đây là hành động phá huỷ** (xoá lịch sử shell, SSH host key, thư mục dự án...) đối với **chính WSL
  instance đang chuẩn bị export** — không ảnh hưởng tới máy Windows host hay các distro khác. Đúng theo
  mục đích của tài liệu gốc (chuẩn bị image dùng chung, không lộ thông tin cá nhân), nhưng vì tính không
  thể hoàn tác, **người vận hành nên xác nhận rõ ràng trước khi Claude Code chạy script này**, đặc biệt
  bước xoá `~/projects/myproject` — chỉ chạy khi đã chắc chắn Task 17-10 (push lên Git) đã hoàn tất.
- Không nên đóng gói mã nguồn trong image; scaffold đã được push lên Git (Task 17-10) nên xoá
  `~/projects/myproject` trước khi export và để member `git clone` sau khi import.
- Người tạo image cũng đã import CA vào kho chứng chỉ Windows của mình (Task 09b); CA đó chỉ thuộc máy đó,
  không nằm trong image.
