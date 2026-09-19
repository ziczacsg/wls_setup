# Task 19 (human): Export WSL

**Loại:** Con người (Human)
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 19
**Phụ thuộc:** Task 18 (đã dọn dẹp và `exit` khỏi WSL)

## Vì sao cần con người thực hiện

`wsl --export` phải chạy từ **PowerShell trên Windows host**, sau khi distro đã dừng hẳn — không thể chạy
từ tiến trình đang sống bên trong chính distro đó (Claude Code trong WSL không thể tự export chính mình).

## Các bước thực hiện

```powershell
[host]
# 1) Xem tên distro & bảo đảm nó đã dừng hẳn
wsl -l -v
wsl --terminate <TEN_DISTRO_GOC>          # hoặc: wsl --shutdown

# 2) Tạo thư mục chứa image
mkdir D:\wsl-images -ErrorAction SilentlyContinue

# 3a) Export dạng .tar
wsl --export <TEN_DISTRO_GOC> D:\wsl-images\flow-dev-1.0.0.tar

# 3b) (WSL mới) export nén trực tiếp – nhỏ hơn nhiều
wsl --export <TEN_DISTRO_GOC> D:\wsl-images\flow-dev-1.0.0.tar.gz --format tar.gz
#     Nếu bản WSL không hỗ trợ --format: nén file .tar bằng 7-Zip (.tar.gz/.tar.xz) rồi phát hành.

# 4) Tạo checksum để member kiểm tra tính toàn vẹn
Get-FileHash D:\wsl-images\flow-dev-1.0.0.tar.gz -Algorithm SHA256
```

## Tiêu chí hoàn thành

- File `.tar`/`.tar.gz` được tạo trong `D:\wsl-images\`.
- Có SHA256 checksum.

## Ghi chú

Phân phối file qua ổ chia sẻ nội bộ/NAS/Drive kèm:

- Số phiên bản image + ngày (`1.0.0`, nội dung thay đổi – changelog)
- SHA256
- Link tới `WSL-DEV-ENVIRONMENT-GUIDE.md` và URL repository skeleton (Task 17-10)

💡 Tar chỉ chứa file thực tế (không chứa vùng trống của VHDX), nên kích thước bằng dung lượng dữ liệu
thật; thường chỉ vài GB, nén xuống còn khoảng một nửa hoặc hơn tuỳ nội dung.
