# Task 13: Docker Engine (optional) cho service Go

**Loại:** Claude Code
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 13
**Phụ thuộc:** Task 01

## Mục đích

Cài Docker Engine trực tiếp trong WSL (không dùng Docker Desktop), chỉ dùng cho service Go phụ trợ —
**không** dùng cho PHP/MySQL. Khởi động theo yêu cầu (socket activation) để không tốn RAM khi không dùng.

## Các bước thực hiện

```bash
[WSL, ubuntu, sudo]
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu noble stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list >/dev/null

sudo apt update
sudo apt install -y --no-install-recommends \
  docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo usermod -aG docker ubuntu
```

Giới hạn dung lượng log container:

```bash
[WSL, ubuntu, sudo]
sudo tee /etc/docker/daemon.json >/dev/null <<'EOF'
{
  "log-driver": "json-file",
  "log-opts": { "max-size": "10m", "max-file": "2" }
}
EOF
```

Khởi động theo yêu cầu:

```bash
[WSL, ubuntu, sudo]
sudo systemctl disable --now docker.service containerd.service
sudo systemctl enable --now docker.socket
```

## Kiểm tra (Verify)

> Cần đăng xuất/đăng nhập lại phiên shell (hoặc `newgrp docker`) để nhóm `docker` có hiệu lực trước khi
> chạy lệnh `docker` không cần `sudo`.

```bash
[WSL, ubuntu]
newgrp docker <<'EOF'
docker run --rm hello-world
docker --version && docker compose version
EOF
```

## Ghi chú

- Task này là **optional** (chỉ cần nếu team dùng service Go phụ trợ — xem Task 17-8).
- Nếu máy member có **Docker Desktop**, cần TẮT WSL integration cho distro `flow-dev` (Docker Desktop →
  Settings → Resources → WSL integration) để không xung đột với Docker Engine trong WSL — đây là thao tác
  trên Windows, thuộc phạm vi **Task 20 (human)** khi member tự thiết lập máy mình, không lặp lại ở đây.
- Mã nguồn mẫu của service Go (Dockerfile, `main.go`, `docker-compose.yml`) được tạo ở Task 17-8.
