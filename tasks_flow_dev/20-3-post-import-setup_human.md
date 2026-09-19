# Task 20-3 (human): Thiết lập cá nhân sau khi import

**Loại:** Con người (Human)
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 20 (mục 20.3)
**Phụ thuộc:** Task 20-1, Task 20-2

## Vì sao cần con người thực hiện

Đổi mật khẩu Linux (`passwd`), nhập Git identity cá nhân, và đăng nhập Claude Code (`claude`) đều là thao
tác tương tác/nhập thông tin cá nhân — không nên và không thể giao cho một phiên Claude Code chưa đăng
nhập thực hiện thay.

## Các bước thực hiện

Lưu vào `infra/scripts/post-import.sh` (sau khi đã `git clone` ở Task 20-5) hoặc làm thủ công:

```bash
[WSL, ubuntu]
# 1) Đổi mật khẩu Linux (mật khẩu tạm "changeme" từ Task 00 KHÔNG được dùng lâu dài)
passwd

# 2) Git identity của chính bạn
git config --global user.name  "<Tên bạn>"
git config --global user.email "<email bạn>"

# 3) Tin cậy CA mkcert trong WSL (curl, PHP curl, ...) — nếu Task 20-2 đã sinh CA
if [ -f "$HOME/.local/share/mkcert/rootCA.pem" ]; then
  sudo install -m 644 "$HOME/.local/share/mkcert/rootCA.pem" /usr/local/share/ca-certificates/flow-dev-local-ca.crt
  sudo update-ca-certificates
else
  echo "Chưa thấy CA: chạy  sudo systemctl start local-dev-certs  rồi chạy lại."
fi

# 4) Kiểm tra dịch vụ (host key SSH và chứng chỉ HTTPS được tự sinh lúc boot đầu tiên)
for s in apache2 mysql redis-server ssh; do
  printf '%-14s %s\n' "$s" "$(systemctl is-active $s)"
done

# 5) Đăng nhập Claude Code bằng tài khoản CÁ NHÂN của bạn (binary đã có sẵn từ image, Task 04)
claude
```

## Tiêu chí hoàn thành

- Mật khẩu Linux đã đổi khỏi `changeme`.
- `git config --global user.name/user.email` là của chính bạn.
- `claude --version` / phiên đăng nhập Claude Code hoạt động bằng tài khoản cá nhân.
- 4 dịch vụ (`apache2`, `mysql`, `redis-server`, `ssh`) đều `active`.

## Ghi chú

- Sau khi hoàn tất, các bước còn lại — mục 3 và 4 ở trên — thực chất là lệnh shell thuần, **có thể** nhờ
  Claude Code (sau khi đã đăng nhập ở bước 5) chạy lại giúp nếu muốn, nhưng bước 1/2/5 bắt buộc do chính
  bạn thực hiện.
- Việc tiếp theo: thêm SSH public key (Task 12b, human, tuỳ chọn) và mở dự án (Task 20-5).
