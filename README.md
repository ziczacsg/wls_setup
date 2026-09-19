# WSL Dev Environment Setup — Flow Framework + Vue 3

Dự án này dùng **Claude Code** để cài đặt và cấu hình môi trường phát triển trên **WSL2 (Ubuntu 24.04)**
cho stack Flow Framework (PHP) + Vue 3, theo hình thức giao từng task nhỏ, độc lập cho Claude Code thực
thi tuần tự trong terminal WSL.

## Cấu trúc thư mục

- [`tasks_flow_dev/`](tasks_flow_dev/) — toàn bộ danh sách task cài đặt, được đánh số và mô tả chi tiết
  từng bước (lệnh chạy, mục đích, phụ thuộc giữa các task). Xem
  [`tasks_flow_dev/README.md`](tasks_flow_dev/README.md) để biết thứ tự thực hiện đầy đủ và bảng phụ
  thuộc giữa các task.

## Nguyên tắc phân công Claude Code / con người

- Mỗi task là một file Markdown riêng trong `tasks_flow_dev/`.
- File có hậu tố `_human` trong tên (ví dụ `04-claude-code-install_human.md`) là task **bắt buộc con
  người thực hiện** — thường là thao tác trên Windows host, hộp thoại xác nhận GUI, đăng nhập/nhập thông
  tin cá nhân, hoặc chính là bước bootstrap cài Claude Code (vì tại thời điểm đó Claude Code chưa tồn tại
  trong WSL để nhận task).
- File **không** có hậu tố `_human` mặc định được giao cho **Claude Code** chạy trong WSL thực hiện.
- Một số task tuy phân loại là Claude Code nhưng cần **xác nhận rõ ràng của người dùng** trước khi thực
  thi (ví dụ: `git push`, xoá dữ liệu/lịch sử) — chi tiết nằm trong từng file task và trong ghi chú vận
  hành ở `tasks_flow_dev/README.md`.

## Bắt đầu từ đâu

1. Đảm bảo WSL2 (Ubuntu 24.04) đã được cài sẵn trên máy (không nằm trong bộ task này).
2. Thực hiện task đầu tiên (bootstrap, con người): xem bảng "Thứ tự thực hiện" trong
   [`tasks_flow_dev/README.md`](tasks_flow_dev/README.md).
3. Sau bootstrap, giao tuần tự các task còn lại cho Claude Code chạy trong WSL theo đúng thứ tự phụ thuộc
   được liệt kê trong `tasks_flow_dev/README.md`.