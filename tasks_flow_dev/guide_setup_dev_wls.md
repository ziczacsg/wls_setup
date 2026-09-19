# Hướng dẫn thiết lập môi trường Dev trên WSL2 — MKintai_Setup_WLS

Tài liệu này **gộp lại** các bước rải rác trong nhiều file task (`19-export-wsl_human.md`,
`20-1` → `20-5`, `12b`, `21`, `16`, `17-6`, `17-6b`) thành một quy trình liền mạch, **điền sẵn giá trị
thật** của dự án (tên project, tên package, URL repo, cổng, mật khẩu dev...) thay vì placeholder
(`myproject`, `Acme.App`, `<URL_REPO>`...). Dùng khi:

- Bạn (hoặc một member khác) **clear/xoá WSL hiện tại rồi import lại** — cần làm lại toàn bộ thiết lập cá
  nhân trên cả WSL lẫn máy Windows host.
- Có người mới join team, nhận image `.tar.gz` lần đầu.
- Cần build bản production hoặc cập nhật/phát hành image mới.

File task lẻ trong `tasks_flow_dev/` (đặc biệt các file `_human`) vẫn là **nguồn tham chiếu chi tiết** khi
cần đối chiếu từng bước hoặc khi Claude Code cần tick tiến độ vào `PROGRESS.md` — tài liệu này chỉ tổng hợp
lại theo trình tự thực tế, không thay thế `PROGRESS.md`/README.md`.

> 📅 Viết lần đầu: 2026-09-19, dựa trên trạng thái thật của WSL đã dựng trong repo này (đã hoàn tất
> HTTPS/mkcert, Xdebug, Adminer; đã **bỏ qua** Docker và một phần Task 02/18-VSCode-extensions — xem
> Phần D).

---

## 0. Giá trị thật cần nhớ (tra cứu nhanh)

| Mục | Giá trị thật | Ghi chú |
|---|---|---|
| Tên dự án (thư mục) | `MKintai_Setup_WLS` | thay cho placeholder `myproject` trong mọi file task |
| Package Flow | `MKintai.App` (composer `mkintai/app`) | thay cho placeholder `Acme.App` |
| Vị trí code package thật | `backend/DistributionPackages/MKintai.App/` | **không phải** `backend/Packages/Application/...` như tài liệu gốc giả định — `./flow kickstart:package` trên Flow 9.1.2 tạo ở `DistributionPackages/` (composer path-repository, xem `backend/composer.json`) |
| URL repo Git | `https://github.com/ziczacsg/MKintai_Setup_WLS.git` | remote `origin`, nhánh `main` |
| Hostname WSL (trong `/etc/wsl.conf`) | `flow-dev` | chỉ là hostname nội bộ Linux, **khác** với tên đăng ký distro trong `wsl -l -v` |
| Tên đăng ký distro (máy dựng image gốc) | `Ubuntu-24.04` | ⚠️ xem cảnh báo ở Phần A.3 — tài liệu gốc dùng ví dụ `flow-dev`, máy dựng image này thực tế tên khác |
| User Linux | `ubuntu`, mật khẩu tạm `changeme` | **phải đổi** ở Task 20-3 (`passwd`) |
| Sudo | `NOPASSWD:ALL` cho `ubuntu` | chỉ dùng cho dev |
| Node.js | `v24.18.1` (nvm) | phải cài **đúng bản này** trên Windows khi build production (Phần C) |
| PHP | `8.3.6` CLI + mod_php | Xdebug 3.2.0 đã bật sẵn |
| MySQL | `8.4.11`, DB `flow_dev`/`flow_test`, user `flow` / pass `flow_dev_pw`, bind `127.0.0.1:3306` | |
| Redis | mặc định, bind `127.0.0.1:6379` | |
| SSH | cổng **2222** (không phải 22), không SFTP | `ssh -p 2222 ubuntu@localhost` |
| Apache HTTP | `:8080` | redirect 301 toàn bộ sang HTTPS 8443 |
| Apache HTTPS (Flow) | `:8443`, vhost `flow.localhost` (alias `localhost`) | `https://flow.localhost:8443/demo` |
| Apache HTTPS (Adminer) | `127.0.0.1:8444`, vhost `adminer.localhost` | chỉ truy cập được từ chính máy đó |
| Vite dev server | `https://localhost:5173` | tự dùng HTTPS vì cert mkcert đã có sẵn trong `/etc/ssl/local-dev/` |
| Xdebug | mode `debug`, trigger, port `9003`, idekey `VSCODE` | `.vscode/launch.json` đã có trong repo |
| Adminer đăng nhập | System **MySQL**, Server `127.0.0.1`, User `flow`, Pass `flow_dev_pw`, DB `flow_dev` | |

---

## Phần A — Dành cho người duy trì image (xuất bản / cập nhật bản WSL dùng chung)

Bỏ qua phần này nếu bạn chỉ là member nhận image có sẵn — sang thẳng **Phần B**.

### A.1. Điều kiện trước khi export

- Toàn bộ scaffold đã **commit & push** lên Git (`17-10-commit-push-scaffold.md`) — image xuất ra
  **không** chứa mã nguồn dự án, member sẽ `git clone` lại sau khi import.
- Đã kiểm tra kỹ các task chính (build/test cơ bản chạy được) trước khi dọn dẹp — sau khi dọn dẹp gần như
  không thể debug lại trên chính máy đó (mất `~/.claude`, lịch sử shell...).

### A.2. Dọn dẹp trước export (⚠️ phá huỷ, không thể hoàn tác)

Chạy `tasks_flow_dev/18-pre-export-cleanup.md` — script `pre-export-cleanup.sh` sẽ:

1. Dọn Docker (nếu có), dừng toàn bộ service.
2. Xoá cache gói (`apt`, `npm`, `composer`).
3. Xoá log, file tạm.
4. **Xoá CA mkcert + chứng chỉ HTTPS** (`/etc/ssl/local-dev/*`, `~/.local/share/mkcert`) — mỗi máy sau khi
   import sẽ tự sinh CA **riêng** (service `local-dev-certs`, Task 09 đã cấu hình sẵn trong image).
5. Xoá `~/.claude`, `~/.claude.json`, SSH host key, SSH key cá nhân, git identity, lịch sử shell,
   `~/.vscode-server`.
6. Xoá hẳn `~/projects/MKintai_Setup_WLS` (đã push Git rồi, member `git clone` lại ở Phần B.7).

**Chỉ chạy sau khi bạn xác nhận rõ ràng** — đặc biệt chắc chắn bước push (Task 26) đã xong.

### A.3. Export (PowerShell trên Windows, ngoài phiên WSL)

```powershell
[host]
wsl -l -v                                   # ⚠️ xem TÊN DISTRO THẬT của bạn — KHÔNG mặc định là "flow-dev"
                                             #    (máy dựng image gốc của repo này đăng ký tên "Ubuntu-24.04")
wsl --terminate <TEN_DISTRO_THAT_CUA_BAN>

mkdir D:\wsl-images -ErrorAction SilentlyContinue
wsl --export <TEN_DISTRO_THAT_CUA_BAN> D:\wsl-images\flow-dev-1.0.0.tar.gz --format tar.gz
Get-FileHash D:\wsl-images\flow-dev-1.0.0.tar.gz -Algorithm SHA256
```

Phát hành kèm: số phiên bản + ngày, SHA256, link tài liệu này + URL repo
(`https://github.com/ziczacsg/MKintai_Setup_WLS.git`).

---

## Phần B — Dành cho member: nhận máy mới / thiết lập lại sau khi clear

### B.1. Chuẩn bị trên Windows host

- Cài/di chuyển Node **không cần** ở bước này (chỉ cần khi build production — Phần C).
- Có sẵn file `flow-dev-<version>.tar.gz` + SHA256 do người duy trì image gửi.

### B.2. Import WSL (Task 20-1)

```powershell
[host]
wsl --update
wsl --version                                          # cần bản có systemd (≥ 2.0)

Get-FileHash .\flow-dev-1.0.0.tar.gz -Algorithm SHA256  # đối chiếu với checksum được gửi kèm

mkdir D:\WSL\flow-dev
wsl --import flow-dev D:\WSL\flow-dev .\flow-dev-1.0.0.tar.gz --version 2
# 👆 đặt tên distro là "flow-dev" ở đây (khuyến nghị, khớp với ví dụ UNC path \\wsl.localhost\flow-dev\...
#    dùng ở các bước sau) — nếu bạn đặt tên khác, nhớ đổi lại "flow-dev" thành tên bạn chọn trong MỌI
#    lệnh \\wsl.localhost\... và ssh config bên dưới.

wsl -l -v
wsl -d flow-dev
```

Kiểm tra: vào WSL phải bằng user `ubuntu` (không phải `root`) — nếu sai, xem
`tasks_flow_dev/20-1-import-wsl_human.md` mục khắc phục.

### B.3. Tin cậy CA HTTPS trên máy bạn (Task 20-2)

Lần khởi động WSL đầu tiên, service `local-dev-certs` (đã có sẵn trong image) tự sinh **CA + chứng chỉ
riêng cho máy bạn** (vài giây). Kiểm tra:

```bash
[WSL, ubuntu]
ls /etc/ssl/local-dev ~/.local/share/mkcert
```

Rồi import CA vào Windows (PowerShell, không cần Administrator, chọn **Yes** ở hộp thoại xác nhận):

```powershell
[host]
certutil -user -addstore Root "\\wsl.localhost\flow-dev\home\ubuntu\.local\share\mkcert\rootCA.pem"
```

Đóng hẳn và mở lại trình duyệt. Firefox: vào `about:config`, đặt `security.enterprise_roots.enabled = true`.

> Không hoàn tất bước này thì `https://flow.localhost:8443` sẽ báo `NET::ERR_CERT_AUTHORITY_INVALID`.

### B.4. Thiết lập cá nhân sau import (Task 20-3)

```bash
[WSL, ubuntu]
# 1) Đổi mật khẩu Linux — KHÔNG dùng "changeme" lâu dài
passwd

# 2) Git identity của chính bạn (Task 02 KHÔNG set sẵn global trong image, chủ đích)
git config --global user.name  "<Tên bạn>"
git config --global user.email "<email bạn>"

# 3) Tin cậy CA mkcert trong WSL (cho curl, PHP curl...)
sudo install -m 644 "$HOME/.local/share/mkcert/rootCA.pem" /usr/local/share/ca-certificates/flow-dev-local-ca.crt
sudo update-ca-certificates

# 4) Kiểm tra 4 dịch vụ đều active (tự khởi động sẵn — xem Phần F, không cần start thủ công)
for s in apache2 mysql redis-server ssh; do printf '%-14s %s\n' "$s" "$(systemctl is-active $s)"; done

# 5) Đăng nhập Claude Code bằng tài khoản cá nhân của bạn
claude
```

### B.5. (Tuỳ chọn) Thêm SSH public key theo máy — Task 12b

Chỉ cần nếu bạn muốn SSH/Remote-SSH vào WSL thay vì dùng VSCode Remote-WSL (không bắt buộc, Remote-WSL
không cần SSH).

```powershell
[host]
ssh-keygen -t ed25519
type $env:USERPROFILE\.ssh\id_ed25519.pub | wsl -d flow-dev -u ubuntu -- bash -c "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

`C:\Users\<user>\.ssh\config`:

```
Host flow-dev
    HostName localhost
    Port 2222
    User ubuntu
    IdentityFile ~/.ssh/id_ed25519
```

Kiểm tra: `ssh flow-dev` kết nối được; `sftp flow-dev` phải báo lỗi (SFTP tắt theo thiết kế).

### B.6. (Tuỳ chọn) Git Credential Manager — Task 20-4

```bash
[WSL, ubuntu]
git config --global credential.helper "/mnt/c/Program\ Files/Git/mingw64/bin/git-credential-manager.exe"
```

(Đổi đường dẫn theo nơi cài Git for Windows trên máy bạn.)

### B.7. Clone dự án, cài dependency, chạy dev (Task 20-5)

```bash
[WSL, ubuntu]
cd ~/projects
git clone https://github.com/ziczacsg/MKintai_Setup_WLS.git MKintai_Setup_WLS
cd MKintai_Setup_WLS
code .
```

```bash
[WSL, ubuntu]
cd ~/projects/MKintai_Setup_WLS/backend && composer install && ./flow doctrine:migrate
```

```bash
[WSL, ubuntu]
cd ~/projects/MKintai_Setup_WLS/frontend
nvm use          # đọc .nvmrc (24.18.1)
npm ci
npm run dev      # nền, https://localhost:5173
```

Mở trình duyệt: `https://flow.localhost:8443/demo` (phải thấy "Vue 3 đã mount ✔" + danh sách sản phẩm) và
`https://flow.localhost:8443/api/health` (6/6 trường `ok`/`true`).

Trong VSCode, cài các extension gợi ý trong `.vscode/extensions.json` khi được nhắc ("Install in WSL:
flow-dev") — xem Phần D về phần chưa cài sẵn trong image gốc.

### B.8. (Tuỳ chọn) Giới hạn RAM/CPU WSL2 — Task 21

`%USERPROFILE%\.wslconfig` (phía Windows, không nằm trong image, mỗi máy cấu hình riêng):

```ini
[wsl2]
memory=4GB
processors=2
swap=2GB
localhostForwarding=true

[experimental]
autoMemoryReclaim=gradual
sparseVhd=true
```

Áp dụng: `wsl --shutdown` rồi mở lại.

---

## Phần C — Build production Vue trên Windows (khi cần bản build tĩnh)

Vue **không build trong WSL** — Vite/esbuild/rollup cài binary theo hệ điều hành, và ghi qua
`\\wsl.localhost\...` từ WSL rất chậm. Luôn build từ một bản clone riêng trên ổ Windows.

### C.1. Chuẩn bị Node trên Windows (một lần)

```powershell
[host]
nvm install 24.18.1        # ĐÚNG bản trong frontend/.nvmrc
nvm use 24.18.1
node -v
```

### C.2. Build & đưa bản build sang WSL

```powershell
[host]
# Một lần
git clone https://github.com/ziczacsg/MKintai_Setup_WLS.git C:\build\MKintai_Setup_WLS

# Mỗi lần build
cd C:\build\MKintai_Setup_WLS\frontend
git pull
npm ci
npm run build
# → sinh backend/DistributionPackages/MKintai.App/Resources/Public/app/app.js, app.css
#   (⚠️ đường dẫn đã ĐƯỢC SỬA so với tài liệu gốc — xem cảnh báo Phần E)

robocopy "C:\build\MKintai_Setup_WLS\backend\DistributionPackages\MKintai.App\Resources\Public\app" `
         "\\wsl.localhost\flow-dev\home\ubuntu\projects\MKintai_Setup_WLS\backend\DistributionPackages\MKintai.App\Resources\Public\app" /MIR
```

### C.3. Chuyển Flow sang đọc bản build (Task 17-6)

```bash
[WSL, ubuntu]
cd ~/projects/MKintai_Setup_WLS/backend
# Sửa Configuration/Development/Settings.yaml: devServer: ''  (thay vì 'https://localhost:5173')
./flow flow:cache:flush
ls ~/projects/MKintai_Setup_WLS/backend/DistributionPackages/MKintai.App/Resources/Public/app/
```

Dừng `npm run dev` (Vite) nếu đang chạy nền — không cần thiết ở chế độ production tĩnh.

### C.4. Kiểm chứng (Task 17-6b, con người)

Tải lại `https://flow.localhost:8443/demo`: view source phải thấy `<link ... app.css>` và
`<script ... app.js>` trỏ `_Resources/Static/Packages/MKintai.App/app/...`, trang vẫn hoạt động đầy đủ,
không còn phụ thuộc Vite.

### C.5. Quay lại chế độ dev

```bash
[WSL, ubuntu]
cd ~/projects/MKintai_Setup_WLS/backend
# Sửa lại Settings.yaml: devServer: 'https://localhost:5173'
./flow flow:cache:flush
cd ../frontend && nvm use && npm run dev
```

---

## Phần D — Việc đã chủ động bỏ qua / chưa hoàn tất trong image (member cần biết)

Ghi theo `PROGRESS.md` tại thời điểm viết tài liệu này — kiểm tra lại file đó nếu tình trạng đã thay đổi:

- **Docker Engine + service Go** (`13-docker-optional.md`, `17-8-verify-docker-go-service.md`): bị bỏ qua.
  Thư mục `services/go-api/` tồn tại nhưng rỗng, không dùng được cho tới khi task này được làm.
- **Git config dùng chung** (`02-git-setup.md`): bị bỏ qua — mỗi member **bắt buộc** tự set
  `git config --global user.name/user.email` (đã có ở B.4, không phải bước thừa).
- **VSCode extensions trong WSL**: image chỉ có sẵn `xdebug.php-debug` + `Anthropic.claude-code` được cài
  qua phiên Claude Code khi dựng image; 5 extension còn lại trong `.vscode/extensions.json`
  (Intelephense, Volar, ESLint, Prettier, EditorConfig) **chưa cài sẵn** — VSCode sẽ tự gợi ý cài khi bạn
  mở project lần đầu (B.7), nhớ chấp nhận gợi ý đó.

---

## Phần E — Lỗi đã gặp & cách xử lý (rút ra từ quá trình dựng image này)

1. **`outDir` Vite sai chỗ (đã fix trong repo, commit `aab36e0`)** — bản cũ trỏ
   `backend/Packages/Application/MKintai.App/...` nhưng `./flow kickstart:package` thực tế tạo package ở
   `backend/DistributionPackages/MKintai.App/...`. Nếu bạn thấy trang production build ra trắng/404 sau
   Task 16, kiểm tra `frontend/vite.config.ts` có đúng `outDir` trỏ `DistributionPackages` không, và
   `git pull` để lấy fix mới nhất.
2. **`devServer` trong `Settings.yaml` phải khớp đúng protocol Vite đang chạy thật** (`http://` nếu chưa có
   mkcert, `https://` nếu đã có) — sai lệch khiến 2 thẻ `<script>` trong trang không load được, Vue không
   bao giờ mount. Sau khi Task 09/mkcert hoàn tất, giá trị đúng luôn là `https://localhost:5173`.
3. **Adminer/Flow dùng name-based virtual host theo `Host` header** — mở đúng domain (`flow.localhost`,
   `adminer.localhost`), không phải IP hay domain khác, nếu không sẽ vào nhầm vhost mặc định. Trình duyệt
   hiện đại (Chrome/Edge/Firefox bản mới) tự resolve `*.localhost` về `127.0.0.1` mà không cần sửa hosts
   file; nếu dùng tool không phải trình duyệt (curl.exe, Postman...), thêm vào
   `C:\Windows\System32\drivers\etc\hosts`:
   ```
   127.0.0.1 flow.localhost
   127.0.0.1 adminer.localhost
   ```
4. **Trang `/` (root) luôn là Flow Welcome mặc định** — route demo thật nằm ở `/demo`
   (`https://flow.localhost:8443/demo`), không phải domain gốc.
5. **Tên đăng ký WSL distro ≠ hostname nội bộ** — `\\wsl.localhost\<TÊN>\...` dùng tên distro lúc
   `wsl --import <TÊN> ...`, không phải giá trị `hostname=flow-dev` trong `/etc/wsl.conf`. Luôn `wsl -l -v`
   để biết tên thật nếu không chắc.
6. Bảng xử lý sự cố đầy đủ hơn: `tasks_flow_dev/17-9-troubleshooting-reference.md`.

---

## Phần F — Service tự khởi động cùng WSL (không cần start thủ công)

`apache2`, `mysql`, `redis-server`, `ssh` (Task 06/07/08/12) đều đã `systemctl enable` trong image, cộng
với `systemd=true` trong `/etc/wsl.conf` (Task 00) — nghĩa là **mỗi lần WSL instance khởi động, systemd tự
start toàn bộ service đã enable**, không cần chạy `sudo systemctl start ...` bằng tay. Hai service oneshot
`local-dev-certs` (Task 09) và `ssh-hostkeys-regen` (Task 12b) cũng theo cơ chế này — chạy một lần lúc
boot rồi tự thoát (`active=inactive` là bình thường, không phải lỗi).

Cách WSL instance "khởi động" theo cơ chế trên:

- Mở terminal chạy `wsl` / `wsl -d flow-dev`, hoặc mở project qua VSCode Remote-WSL — bất kỳ cách nào làm
  WSL instance chuyển từ trạng thái dừng sang chạy đều kích hoạt systemd, kéo theo toàn bộ service ở trên.
- **WSL instance không tự khởi động khi Windows khởi động** — khác với service Linux bên trong. Nếu muốn
  máy luôn sẵn sàng ngay khi mở Windows (không phải lúc mở terminal/VSCode lần đầu), cần tự cấu hình thêm
  ở phía Windows (ví dụ Task Scheduler chạy `wsl.exe -d flow-dev exit` lúc đăng nhập) — việc này **nằm
  ngoài phạm vi** các task trong `tasks_flow_dev/`, tuỳ chọn theo nhu cầu cá nhân.
- Nếu lệnh kiểm tra ở B.4 bước 4 hoặc Phụ lục bên dưới báo service chưa `active` ngay sau khi vừa mở WSL,
  đợi vài giây rồi kiểm tra lại (systemd cần thời gian khởi động các unit) trước khi nghi ngờ có lỗi cấu
  hình.

---

## Phụ lục — Lệnh kiểm tra nhanh sau khi thiết lập xong

```bash
[WSL, ubuntu]
for s in apache2 mysql redis-server ssh; do printf '%-14s %s\n' "$s" "$(systemctl is-active $s)"; done
curl -sk https://localhost:8443/api/health | python3 -m json.tool
curl -sk -I https://localhost:8444 | head -1
curl -sk -o /dev/null -w '%{http_code}\n' https://localhost:5173/@vite/client
```

Tất cả đạt như trên tức là môi trường đã sẵn sàng để phát triển.
