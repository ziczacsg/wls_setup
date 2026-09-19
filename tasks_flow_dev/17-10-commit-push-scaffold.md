# Task 17-10: Commit scaffold làm skeleton của team

**Loại:** Claude Code (⚠️ cần con người xác nhận trước khi `git push` — xem mục Ghi chú)
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 17.10
**Phụ thuộc:** Task 17-1 → 17-4 đã kiểm chứng đạt (tối thiểu; Task 17-5/17-7 là kiểm chứng bổ sung do con
người thực hiện, không bắt buộc phải xong trước khi commit)

## Mục đích

Commit toàn bộ scaffold (Task 14, 17-1, 17-2, 17-8) làm skeleton dùng chung cho team, đẩy lên remote Git để
member chỉ cần `git clone` sau khi import WSL — image (Task 19) không cần chứa mã nguồn.

## Các bước thực hiện

```bash
[WSL, ubuntu]
cd ~/projects/myproject
git add -A
git commit -m "chore: scaffold Flow (Acme.App) + Fluid wrapper + Vue 3"
```

`git push` **chỉ chạy sau khi có xác nhận của người dùng** và có URL repository thật:

```bash
[WSL, ubuntu]
git remote add origin <URL_REPO>
git push -u origin main
```

## Kiểm tra (Verify)

```bash
[WSL, ubuntu]
git log --oneline -1
git remote -v
```

## Ghi chú quan trọng

- ⚠️ `<URL_REPO>` phải do người dùng cung cấp — Claude Code không được tự đoán/tạo URL repository.
- ⚠️ Theo nguyên tắc an toàn khi thao tác Git, `git push` là hành động **hiển thị với người khác** (tạo
  remote, đẩy code lên) — cần xác nhận rõ ràng của người dùng trước khi thực hiện, kể cả khi task này được
  giao cho Claude Code.
- Sau bước này, member chỉ cần `git clone` skeleton sau khi import WSL (Task 20-5) — image xuất ra ở Task 19
  **không** chứa `~/projects/myproject` (xem Task 18, mục dọn dẹp).
