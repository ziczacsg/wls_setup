# Task 20-2 (human): Tin cậy CA HTTPS của máy bạn (một lần, sau import)

**Loại:** Con người (Human)
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 20 (mục 20.2)
**Phụ thuộc:** Task 20-1

## Vì sao cần con người thực hiện

Giống Task 09b: import CA vào kho chứng chỉ Windows hiện hộp thoại xác nhận cần bấm "Yes" thủ công.

## Các bước thực hiện

Lần khởi động đầu, service `local-dev-certs` (đã cấu hình sẵn trong image, Task 09) tự sinh **CA riêng cho
máy bạn** và chứng chỉ (mất vài giây). Kiểm tra trong WSL:

```bash
[WSL, ubuntu]
ls /etc/ssl/local-dev ~/.local/share/mkcert
```

Rồi import CA vào Windows (PowerShell, **không** cần Administrator; chọn *Yes* ở hộp thoại xác nhận):

```powershell
[host]
certutil -user -addstore Root "\\wsl.localhost\flow-dev\home\ubuntu\.local\share\mkcert\rootCA.pem"
```

Đóng hẳn và mở lại trình duyệt. Firefox: đặt `security.enterprise_roots.enabled = true` trong
`about:config`.

## Tiêu chí hoàn thành

Trình duyệt không còn cảnh báo chứng chỉ khi mở `https://flow.localhost:8443` (sau khi dự án đã chạy —
Task 20-5).
