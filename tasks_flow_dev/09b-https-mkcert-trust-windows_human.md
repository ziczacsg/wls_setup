# Task 09b (human): Tin cậy CA của mkcert trên Windows

**Loại:** Con người (Human)
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 9 (mục 9.4)
**Phụ thuộc:** Task 09

## Vì sao cần con người thực hiện

Việc import chứng chỉ gốc (root CA) vào kho chứng chỉ Windows hiển thị **hộp thoại xác nhận** cần người
dùng bấm "Yes" — không thể tự động hoá từ script. Trình duyệt (Chrome/Edge) chạy trên Windows nên phải tin
cậy CA ở phía Windows, không phải trong WSL.

## Các bước thực hiện

Chạy trong **PowerShell** trên Windows (đổi `flow-dev` thành tên distro thật nếu khác):

```powershell
[host]
certutil -user -addstore Root "\\wsl.localhost\flow-dev\home\ubuntu\.local\share\mkcert\rootCA.pem"
```

Windows sẽ hiện hộp thoại xác nhận cài chứng chỉ gốc → chọn **Yes**. Cách này chỉ ghi vào kho của *user
hiện tại*, không cần quyền Administrator.

Sau đó **đóng hẳn và mở lại trình duyệt**.

> Firefox dùng kho chứng chỉ riêng: vào `about:config` → đặt `security.enterprise_roots.enabled = true` để
> Firefox dùng kho của Windows.

## Tiêu chí hoàn thành

- `certutil` báo cài chứng chỉ thành công.
- Mở `https://localhost:8443` (sau khi có vhost ở Task 09/17) không còn cảnh báo
  `NET::ERR_CERT_AUTHORITY_INVALID`.

## Ghi chú

Nếu sau này chứng chỉ hết hạn hoặc CA bị xoá (Task 18 pre-export-cleanup xoá CA trước khi export), mỗi máy
sẽ tự sinh CA mới khi khởi động lần đầu (Task 09) — cần lặp lại task này một lần nữa trên máy đó.
