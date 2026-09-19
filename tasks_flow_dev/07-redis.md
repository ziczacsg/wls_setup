# Task 07: Redis

**Loại:** Claude Code
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 7
**Phụ thuộc:** Task 01

## Mục đích

Cài Redis, cấu hình làm cache (tắt persistence), khởi động và kiểm tra.

## Các bước thực hiện

```bash
[WSL, ubuntu, sudo]
sudo apt install -y --no-install-recommends redis-server redis-tools

sudo tee -a /etc/redis/redis.conf >/dev/null <<'EOF'

# --- Dev overrides ---
maxmemory 128mb
maxmemory-policy allkeys-lru
save ""
appendonly no
EOF

sudo systemctl enable --now redis-server
redis-cli ping           # PONG
redis-server --version
```

## Kiểm tra (Verify)

```bash
[WSL, ubuntu]
redis-cli ping                       # PONG
systemctl is-active redis-server     # active
```

## Ghi chú (tuỳ chọn — không bắt buộc trong scaffold)

Dùng Redis làm cache backend cho Flow, thêm vào `backend/Configuration/Development/Caches.yaml` (sẽ tồn
tại từ Task 14 trở đi):

```yaml
Flow_Mvc_Routing_Route:
  backend: Neos\Cache\Backend\RedisBackend
  backendOptions:
    hostname: 127.0.0.1
    port: 6379
    database: 0
```
