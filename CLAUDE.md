# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Dự án này là gì

Đây **không phải** một codebase ứng dụng — đây là bộ **task hướng dẫn cài đặt môi trường dev** trên
WSL2 (Ubuntu 24.04) cho dự án **`MKintai_Setup_WLS`** (stack Flow Framework/PHP + Vue 3), được breakdown
thành các file Markdown độc lập trong [`tasks_flow_dev/`](tasks_flow_dev/) để giao cho Claude Code thực
thi tuần tự.

**Vai trò của Claude Code trong repo này:** thực thi trực tiếp — đọc từng file task, chạy thật các lệnh
(`[WSL, ubuntu]`, `[WSL, ubuntu, sudo]`) lên chính máy WSL đang chạy phiên này (cài package, sửa file cấu
hình hệ thống, khởi tạo repo dự án...), rồi tự kiểm chứng bằng mục "Kiểm tra (Verify)" của từng file
trước khi coi task đã xong. Việc sửa nội dung các file `.md` trong `tasks_flow_dev/` chỉ cần thiết khi
người dùng yêu cầu chỉnh sửa quy trình, không phải hoạt động chính.

## Theo dõi tiến độ

[`tasks_flow_dev/PROGRESS.md`](tasks_flow_dev/PROGRESS.md) là checklist trạng thái từng task. Khi bắt đầu
phiên mới trong repo này: đọc file đó trước để biết đang ở task nào. Sau khi một task chạy **và** verify
đạt, tick `[x]` cho task đó trong `PROGRESS.md` trước khi chuyển sang task tiếp theo — không tick trước.

## Thứ tự thực hiện & phụ thuộc

Toàn bộ thứ tự, bảng phụ thuộc giữa các task, và ký hiệu ngữ cảnh lệnh (`[WSL, ubuntu]` /
`[WSL, ubuntu, sudo]` / `[host]`) nằm trong [`tasks_flow_dev/README.md`](tasks_flow_dev/README.md) —
tham chiếu file đó, không tự suy luận thứ tự khác. Số trong tên file bám theo số "Bước" của tài liệu gốc,
**không** phải lúc nào cũng là thứ tự thực hiện thực tế (ví dụ Task 04 chạy đầu tiên dù mang số 04).

## Quy ước phân công Claude Code / con người

- File có hậu tố `_human` trong tên → **bắt buộc con người thực hiện** (thao tác Windows host, hộp thoại
  GUI, đăng nhập, hoặc bootstrap cài Claude Code). Claude Code không tự chạy các bước trong file này —
  chỉ nhắc người dùng thực hiện và chờ xác nhận trước khi tiếp tục sang task phụ thuộc.
- File không có hậu tố `_human` → mặc định Claude Code thực thi.

## Task cần xác nhận rõ ràng trước khi chạy (dù được giao cho Claude Code)

- **`17-10-commit-push-scaffold.md`** — `git add`/`commit` chạy bình thường, nhưng `git push` **chỉ chạy
  sau khi người dùng xác nhận và cung cấp URL repository thật**. Không tự đoán/tạo URL repo.
- **`18-pre-export-cleanup.md`** — script xoá dữ liệu, lịch sử shell, SSH host key, `~/.claude`, và cuối
  cùng `rm -rf ~/projects/MKintai_Setup_WLS`. Đây là hành động **phá huỷ, không thể hoàn tác** trên chính
  WSL instance đang chạy — luôn hỏi xác nhận rõ ràng của người dùng trước khi chạy script hoặc trước bước
  xoá thư mục dự án, đặc biệt phải chắc Task 26 (push lên Git) đã hoàn tất trước.

## Tên dự án thật (thay placeholder trong tài liệu task)

Các file task (đặc biệt Task 14, 17-1...) dùng placeholder `myproject` cho thư mục gốc. Tên dự án thật
của repo này là **`MKintai_Setup_WLS`** — khi thực thi các lệnh có `~/projects/myproject`, thay bằng
`~/projects/MKintai_Setup_WLS`. Tên package backend `Acme.App` trong Task 17-1 chưa được xác nhận đổi —
nếu người dùng chưa nói rõ tên package thật, hỏi lại trước khi chạy Task 17-1 thay vì tự đoán.

## Ngôn ngữ làm việc

Phản hồi và commit message trong repo này dùng **tiếng Việt**, nhất quán với toàn bộ tài liệu task hiện có.

## Kiến trúc dự án đích (áp dụng từ Task 14 trở đi)

Dự án đích là **monorepo** tại `~/projects/MKintai_Setup_WLS` gồm `backend/` (PHP Flow Framework),
`frontend/` (Vue 3), `services/go-api/` (Go, optional), `infra/` (script dùng chung). Nguyên tắc kiến
trúc cần nhớ khi tạo code ở các task sau:

- **Backend (Flow)** sở hữu routing, controller, dữ liệu, và trang HTML (Fluid). Fluid chỉ tạo *thẻ gốc*
  mang thuộc tính `data-*`.
- **Frontend (Vue 3)** chỉ là tập component được mount vào các thẻ gốc đó ("islands") — không có
  `index.html` riêng, không dùng Vue Router (routing do Flow đảm nhiệm).
- Mỗi thẻ gốc khai báo `data-vue-component="<TênComponent>"` và `data-props='<JSON>'`.
- Bản build production của Vue được build trên **Windows host** (Task 16, do Node trên WSL chậm khi ghi
  qua `\\wsl.localhost\...`), không build trong WSL.

## Lệnh thường dùng khi thực thi/kiểm chứng task

- Backend (sau Task 17-1, trong `backend/`): `./flow flow:cache:flush`, `./flow doctrine:migrate`,
  `./flow help`, `composer install`.
- Frontend (trong `frontend/`): `npm run dev` (Vite), `npm run build`, `npm run typecheck` (`vue-tsc`).
- Git dùng chung cho image: **không** set `user.name`/`user.email` global trong task cài đặt image (Task
  02) — mỗi member tự đặt identity sau khi import (Task 20-3, human).
