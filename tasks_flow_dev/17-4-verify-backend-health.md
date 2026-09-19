# Task 17-4: Kiểm chứng backend qua `/api/health`

**Loại:** Claude Code
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 17.4
**Phụ thuộc:** Task 17-1, Task 09 (HTTPS), Task 06 (MySQL), Task 07 (Redis), Task 10 (Xdebug)

## Mục đích

Xác minh toàn bộ chuỗi Apache + mod_php + Flow + MySQL 8.4 + Redis + HTTPS hoạt động đúng bằng endpoint
`/api/health`.

## Các bước thực hiện / Kiểm tra

```bash
[WSL, ubuntu]
# HTTP → HTTPS
curl -sI http://localhost:8080/demo | head -3            # 301 → https://flow.localhost:8443/demo

# Sức khoẻ môi trường
curl -s https://localhost:8443/api/health | python3 -m json.tool
```

Kết quả kỳ vọng (giá trị phiên bản có thể khác):

```json
{
    "flowContext": "Development",
    "php": { "version": "8.3.x", "sapi": "apache2handler", "xdebug": true },
    "https": true,
    "mysql": { "ok": true, "version": "8.4.x" },
    "redis": { "ok": true, "version": "7.x.x" }
}
```

| Trường | Ý nghĩa nếu đạt |
|---|---|
| `php.sapi = apache2handler` | Web đang chạy bằng **mod_php** (không phải FPM/CLI) |
| `php.version = 8.3.x` | Đúng phiên bản PHP |
| `https = true` | Request đi qua TLS trực tiếp tại Apache |
| `mysql.version = 8.4.x` | Flow (mod_php) kết nối được MySQL 8.4 |
| `redis.ok = true` | `ext-redis` nạp cho Apache và Redis phản hồi |
| `xdebug = true` | Xdebug đã nạp cho Apache |

## Tiêu chí hoàn thành

Toàn bộ 6 trường trên đạt giá trị kỳ vọng. Nếu không, tra bảng xử lý sự cố ở **Task 17-9 (tham khảo)**.

## Ghi chú

Đây là bước kiểm chứng **thuần API/CLI**, không cần trình duyệt — phù hợp để Claude Code tự xác minh mà
không cần con người can thiệp.
