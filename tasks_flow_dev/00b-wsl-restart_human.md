# Task 00b (human): Restart WSL để `wsl.conf` có hiệu lực

**Loại:** Con người (Human)
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 0 (mục 0.3)
**Phụ thuộc:** Task 00

## Vì sao cần con người thực hiện

`wsl --shutdown` phải chạy từ **PowerShell trên Windows**, ở ngoài phiên WSL hiện tại. Nếu để tiến trình
Claude Code (đang chạy bên trong WSL) tự tắt distro của chính nó thì phiên làm việc sẽ bị ngắt giữa chừng —
cần người dùng chủ động chạy lệnh này rồi mở lại terminal để Claude Code tiếp tục.

## Các bước thực hiện

```powershell
[host]
wsl --shutdown
wsl -d <TEN_DISTRO_HIEN_TAI>
```

Sau khi vào lại WSL, kiểm tra:

```bash
[WSL, ubuntu]
systemctl is-system-running     # "running" hoặc "degraded" đều được
whoami                          # phải là "ubuntu", không phải "root"
```

## Tiêu chí hoàn thành

- `systemctl is-system-running` trả về `running` hoặc `degraded` (không phải lỗi/treo).
- `whoami` in ra `ubuntu`.

## Ghi chú

Nếu `whoami` vẫn ra `root`, kiểm tra lại nội dung `/etc/wsl.conf` (Task 00) rồi lặp lại `wsl --shutdown`.
