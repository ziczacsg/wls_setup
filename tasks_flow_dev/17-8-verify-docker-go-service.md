# Task 17-8: Kiểm chứng Docker + service Go (optional)

**Loại:** Claude Code
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 17.8
**Phụ thuộc:** Task 13 (Docker), Task 14 (thư mục `services/go-api` đã tạo)

## Mục đích

Tạo service Go tối giản (`/health` trả JSON), Dockerfile multi-stage, `docker-compose.yml`, build và chạy
thử qua Docker Engine trong WSL.

## Các bước thực hiện

`services/go-api/go.mod`:

```
module example.com/go-api

go 1.24
```

`services/go-api/main.go`:

```go
package main

import (
	"encoding/json"
	"log"
	"net/http"
	"time"
)

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/health", func(w http.ResponseWriter, r *http.Request) {
		w.Header().Set("Content-Type", "application/json")
		_ = json.NewEncoder(w).Encode(map[string]any{
			"status":  "ok",
			"service": "go-api",
			"time":    time.Now().UTC().Format(time.RFC3339),
		})
	})

	log.Println("go-api listening on :8080")
	log.Fatal(http.ListenAndServe(":8080", mux))
}
```

`services/go-api/Dockerfile`:

```dockerfile
FROM golang:1.24-alpine AS build
WORKDIR /src
COPY go.* ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o /out/api .

FROM alpine:3
COPY --from=build /out/api /usr/local/bin/api
EXPOSE 8080
ENTRYPOINT ["api"]
```

`docker-compose.yml` (ở gốc dự án `~/projects/myproject/`):

```yaml
services:
  go-api:
    build: ./services/go-api
    ports:
      - "127.0.0.1:8090:8080"
    restart: "no"
```

Chạy thử:

```bash
[WSL, ubuntu]
cd ~/projects/myproject
docker compose up -d --build
curl -s http://127.0.0.1:8090/health        # {"service":"go-api","status":"ok","time":"..."}
docker compose down
```

## Kiểm tra (Verify)

`curl` trả về JSON với `"status":"ok"` và `"service":"go-api"`.

## Ghi chú

- Task này chỉ áp dụng nếu Task 13 (Docker) đã được thực hiện — hoàn toàn optional theo thiết kế gốc.
- Service Go dùng HTTP trên `127.0.0.1` là đủ cho dev. Nếu Vue cần gọi trực tiếp, đặt sau proxy của Flow
  hoặc mở CORS cho `https://flow.localhost:8443`.
