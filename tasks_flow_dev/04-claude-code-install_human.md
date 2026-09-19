# Task 04 (human): Cài đặt Claude Code trong WSL (bootstrap)

**Loại:** Con người (Human)
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 4
**Phụ thuộc:** — (task đầu tiên; tiền đề duy nhất: WSL + distro Ubuntu 24.04 đã được cài sẵn)

## Vì sao cần con người thực hiện

Đây là task **bootstrap**, chạy **trước tất cả** các task khác trong `tasks_flow_dev/`. Toàn bộ các task
còn lại (kể cả Task 00) được giao cho Claude Code thực hiện, nhưng Claude Code chưa tồn tại trong WSL cho
tới khi task này hoàn tất — vì vậy bước cài đặt ban đầu bắt buộc một người thực hiện thủ công (mở terminal
WSL mặc định của distro và chạy các lệnh dưới đây). Sau task này, tất cả các task khác (00, 00b, 01, 02,
03, 05, ...) có thể giao cho Claude Code chạy trong WSL.

> Lưu ý: Task 00b (restart WSL) sẽ làm ngắt phiên `claude` đang chạy. Sau mỗi lần `wsl --shutdown`/mở lại
> distro, người dùng cần tự gõ `claude` lại trong terminal WSL để tiếp tục giao task — đây là thao tác thủ
> công nhỏ, không cần lặp lại toàn bộ Task 04.

## Các bước thực hiện

```bash
[WSL, ubuntu]
curl -fsSL https://claude.ai/install.sh | bash

# Bảo đảm ~/.local/bin nằm trong PATH
grep -q '.local/bin' ~/.bashrc || echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

claude --version
claude doctor        # chẩn đoán cài đặt
```

Tài liệu tham khảo: <https://code.claude.com/docs/en/setup>

## Tiêu chí hoàn thành

- `claude --version` chạy được, không lỗi.
- `claude doctor` không báo lỗi nghiêm trọng.

## Ghi chú quan trọng

- ⚠️ **KHÔNG đăng nhập (`claude` → login) trên image dùng chung.** Đây là image sẽ được export và chia sẻ
  cho cả team (Task 18/19) — nếu đăng nhập ở bước này, tài khoản cá nhân sẽ lộ ra trong image. Việc đăng
  nhập chỉ thực hiện bởi từng member sau khi họ tự import WSL về máy mình (xem Task 20-3, human).
- Nếu đã lỡ đăng nhập để thử nghiệm, phải xoá `~/.claude` và `~/.claude.json` trước khi export — Task 18
  (pre-export-cleanup) đã tự động làm việc này, nhưng nên biết trước để không phụ thuộc hoàn toàn vào nó.
- Đừng đặt `ANTHROPIC_API_KEY` vào `~/.bashrc` của image dùng chung.
- Từ đây trở đi, có thể chạy `claude` trong thư mục dự án và giao các task còn lại (Task 00, 01, 02, 03,
  05, 06, ...) cho Claude Code thực hiện tuần tự theo `README.md`.
