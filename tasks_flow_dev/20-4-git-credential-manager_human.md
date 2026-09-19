# Task 20-4 (human, tuỳ chọn): Git Credential Manager của Windows

**Loại:** Con người (Human)
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 20 (mục 20.4)
**Phụ thuộc:** Task 20-3

## Vì sao cần con người thực hiện

Đường dẫn tới `git-credential-manager.exe` khác nhau theo máy (tuỳ nơi cài Git for Windows) và liên quan
tới việc xác thực tài khoản Git cá nhân — nên để member tự cấu hình theo máy của họ.

## Các bước thực hiện

```bash
[WSL, ubuntu]
git config --global credential.helper "/mnt/c/Program\ Files/Git/mingw64/bin/git-credential-manager.exe"
```

(Đổi đường dẫn cho khớp nơi cài Git for Windows trên máy bạn.)

## Tiêu chí hoàn thành

`git push`/`git pull` tới remote riêng tư không còn hỏi lại mật khẩu mỗi lần.
