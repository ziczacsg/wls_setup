# Task 21 (human, tuỳ chọn): `.wslconfig` — giới hạn RAM/CPU cho WSL2

**Loại:** Con người (Human)
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Phụ lục A
**Phụ thuộc:** Task 20-1 (đã import)

## Vì sao cần con người thực hiện

File `%USERPROFILE%\.wslconfig` nằm ở **phía Windows**, mỗi máy một file riêng theo cấu hình phần cứng
thật của máy đó, và **không** nằm trong image dùng chung (không được đóng gói/đồng bộ qua Task 18/19).

## Các bước thực hiện

Tạo/sửa `%USERPROFILE%\.wslconfig`:

```ini
[wsl2]
memory=4GB              # chỉnh theo RAM máy (vd 4–6GB)
processors=2
swap=2GB
localhostForwarding=true
# networkingMode=NAT là mặc định – giữ nguyên. Nếu dùng mirrored, cổng WSL trùng cổng Windows sẽ xung đột.

[experimental]
autoMemoryReclaim=gradual   # trả RAM cho Windows khi rảnh
sparseVhd=true              # VHDX tự thu nhỏ khi xoá file
```

Áp dụng:

```powershell
[host]
wsl --shutdown
```

Rồi mở lại WSL.

## Tiêu chí hoàn thành

WSL khởi động lại bình thường, và `Get-Process Vmmem`/`Task Manager` trên Windows cho thấy RAM WSL bị giới
hạn đúng theo `memory=` đã đặt.
