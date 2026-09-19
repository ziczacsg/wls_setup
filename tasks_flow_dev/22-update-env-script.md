# Task 22 (tuỳ chọn): Script cập nhật môi trường không cần export lại

**Loại:** Claude Code
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Phụ lục B
**Phụ thuộc:** Task 17-10 (repo skeleton đã tồn tại)

## Mục đích

Chuẩn bị nơi chứa các thay đổi môi trường nhỏ (thêm extension PHP, đổi cấu hình Apache/MySQL, ...) để
member chạy trên bản WSL của họ, thay vì phát hành lại toàn bộ image nặng vài GB mỗi lần.

## Các bước thực hiện

Tạo stub script `infra/scripts/update-env.sh` trong repo (nội dung cụ thể sẽ được bổ sung dần theo từng
thay đổi hạ tầng thực tế, không có sẵn trong tài liệu gốc — đây chỉ là khung):

```bash
[WSL, ubuntu]
mkdir -p ~/projects/myproject/infra/scripts
cat > ~/projects/myproject/infra/scripts/update-env.sh <<'EOF'
#!/usr/bin/env bash
# Chạy trên WSL của member để áp dụng các thay đổi môi trường nhỏ, không cần export/import lại image.
# Thêm các bước cụ thể vào đây khi có thay đổi (cài thêm ext PHP, đổi cấu hình Apache/MySQL, ...).
set -euo pipefail

echo "update-env.sh: chưa có thay đổi nào được định nghĩa."
EOF
chmod +x ~/projects/myproject/infra/scripts/update-env.sh
```

Cách member dùng về sau:

```bash
[WSL, ubuntu]
cd ~/projects/myproject && git pull
bash infra/scripts/update-env.sh
```

## Kiểm tra (Verify)

```bash
[WSL, ubuntu]
test -x ~/projects/myproject/infra/scripts/update-env.sh && echo OK
```

## Ghi chú

Chỉ phát hành image mới (`1.1.0`, `2.0.0`, ...) khi có thay đổi lớn: nâng phiên bản PHP/MySQL/Node, đổi
cấu trúc hệ thống — lặp lại từ Task 18/19 (dùng bản WSL hiện tại của người tạo image, không phải import
từ đầu).
