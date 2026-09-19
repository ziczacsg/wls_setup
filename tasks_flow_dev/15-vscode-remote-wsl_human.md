# Task 15 (human): Cài VSCode + extension WSL, mở dự án qua Remote - WSL

**Loại:** Con người (Human)
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 15
**Phụ thuộc:** Task 14

## Vì sao cần con người thực hiện

Cài VSCode và extension trên **Windows host**, và cài extension "bên trong WSL" là thao tác GUI tương tác
(nút *Install in WSL: flow-dev*) — không tự động hoá được từ Claude Code chạy trong WSL.

## Các bước thực hiện

1. Cài VSCode trên Windows + extension **WSL** (`ms-vscode-remote.remote-wsl`).
2. Mở dự án từ WSL:
   ```bash
   [WSL, ubuntu]
   cd ~/projects/myproject
   code .
   ```
   Lần đầu VSCode tự tải `vscode-server` vào `~/.vscode-server` (không đóng gói sẵn trong image).
3. Trong cửa sổ VSCode vừa mở, cài các extension được gợi ý trong `.vscode/extensions.json` (đã tạo ở
   Task 14) bằng nút **Install in WSL: flow-dev** cho từng extension, hoặc chấp nhận gợi ý cài theo
   workspace.

## Tiêu chí hoàn thành

- VSCode kết nối được vào WSL (góc dưới trái hiển thị "WSL: flow-dev").
- Các extension trong `extensions.json` đã cài **bên trong WSL** (không phải chỉ trên Windows).

## Ghi chú

- Dùng **Remote - WSL** thì không cần SSH. Nếu muốn dùng **Remote - SSH**, chọn host `flow-dev` đã cấu
  hình ở Task 12b (human).
