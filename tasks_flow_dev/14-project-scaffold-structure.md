# Task 14: Kiến trúc frontend & khởi tạo cấu trúc thư mục dự án (monorepo)

**Loại:** Claude Code
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 14
**Phụ thuộc:** Task 03 (Node/nvm), Task 05 (PHP/Composer), Task 08 (Apache đã trỏ DocumentRoot tới
`~/projects/myproject/backend/Web`)

## Mục đích

Khởi tạo repo Git **monorepo** tại `~/projects/myproject` với 3 phần tách biệt: `backend/` (PHP Flow
Framework), `frontend/` (Vue 3 components — không phải SPA độc lập), `services/` (Go, optional),
`infra/` (script dùng chung).

### Nguyên tắc kiến trúc (ghi nhớ khi viết code ở các task sau)

- **Backend** (Flow) sở hữu routing, controller, dữ liệu và trang HTML (Fluid). Fluid chỉ tạo *thẻ gốc*
  có `data-*`.
- **Frontend** (Vue 3) chỉ là tập **component** được mount vào các thẻ gốc đó ("islands"). Không có
  `index.html`, không có Vue Router (routing do Flow lo).
- Mỗi thẻ gốc khai báo: `data-vue-component="<TênComponent>"` và `data-props='<JSON>'`.

## Cấu trúc thư mục mục tiêu

```
/home/ubuntu/projects/myproject/
├── backend/                        # PHP Flow Framework (DocumentRoot = backend/Web)
│   ├── Configuration/{Development,Production,Testing}/, Routes.yaml, Settings.yaml
│   ├── Data/                       # (gitignore)
│   ├── Packages/Application/Acme.App/   # code ứng dụng của team (commit) — tạo ở Task 17
│   ├── Web/                        # DocumentRoot
│   ├── composer.json
│   └── flow
├── frontend/                       # Vue 3 (component mount vào thẻ gốc)
│   └── src/{main.ts,env.d.ts,components/,composables/,api/,types/,assets/main.css}
├── services/go-api/                # service Go (Docker) — nội dung tạo ở Task 17-8
├── infra/{apache,mysql,php,wsl,scripts}/
├── database/
├── docs/
├── docker-compose.yml
├── .vscode/
├── .editorconfig  .gitattributes  .gitignore
└── README.md
```

## Các bước thực hiện

### 14.3. Khởi tạo repo & backend

```bash
[WSL, ubuntu]
mkdir -p ~/projects/myproject && cd ~/projects/myproject
git init

# Backend – Flow Framework
composer create-project neos/flow-base-distribution backend --remove-vcs

mkdir -p services/go-api infra/apache infra/mysql infra/php infra/wsl infra/scripts database docs .vscode
```

`backend/Configuration/Development/Settings.yaml` (kết nối MySQL — phần `Acme.App` sẽ được thêm ở
Task 17):

```yaml
Neos:
  Flow:
    persistence:
      backendOptions:
        driver: pdo_mysql
        host: 127.0.0.1
        port: 3306
        dbname: flow_dev
        user: flow
        password: flow_dev_pw
        charset: utf8mb4
```

```bash
[WSL, ubuntu]
cd ~/projects/myproject/backend
./flow flow:cache:flush
./flow doctrine:migrate
./flow help
```

Kiểm chứng: mở `https://flow.localhost:8443` (sau khi Task 09 xong) phải thấy trang chào của Flow.

> Mật khẩu DB dev dùng chung có thể commit trong `Configuration/Development`. **Không** commit secret của
> staging/production.

### 14.4. Khởi tạo frontend (tối giản, không dùng `create-vue`)

```bash
[WSL, ubuntu]
mkdir -p ~/projects/myproject/frontend && cd ~/projects/myproject/frontend

npm init -y >/dev/null
npm pkg set name=frontend type=module
npm pkg set --json private=true
npm pkg set scripts.dev="vite" scripts.build="vite build" scripts.typecheck="vue-tsc --noEmit"
npm pkg set engines.node=">=24.18"

npm install vue
npm install -D vite @vitejs/plugin-vue typescript vue-tsc @types/node

node -v | sed 's/^v//' > .nvmrc
mkdir -p src/components src/composables src/api src/types src/assets
```

`frontend/tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "strict": true,
    "isolatedModules": true,
    "skipLibCheck": true,
    "noEmit": true,
    "types": ["vite/client", "node"],
    "baseUrl": ".",
    "paths": { "@/*": ["src/*"] }
  },
  "include": ["src/**/*.ts", "src/**/*.vue", "vite.config.ts"]
}
```

`frontend/src/env.d.ts`:

```ts
/// <reference types="vite/client" />
```

### 14.5. File cấu hình gốc repo

`.gitattributes`:

```
* text=auto eol=lf
*.bat text eol=crlf
*.png binary
*.jpg binary
*.woff2 binary
```

`.editorconfig`:

```ini
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
indent_style = space
indent_size = 4

[*.{js,ts,vue,json,yaml,yml,css,scss,html}]
indent_size = 2
```

`.gitignore` (bổ sung cho `.gitignore` đi kèm Flow distribution):

```gitignore
# Backend
/backend/Data/
/backend/Web/_Resources/
/backend/Packages/*
!/backend/Packages/Application/
/backend/Packages/Application/*
!/backend/Packages/Application/Acme.App/
# Bản build của Vue (sinh ra từ Windows, không commit)
/backend/Packages/Application/Acme.App/Resources/Public/app/

# Frontend
/frontend/node_modules/

# Go / Docker
/services/**/bin/

# Editor / OS
.idea/
.DS_Store
Thumbs.db
```

### Cấu hình VSCode dùng chung (đặt trong repo — tham chiếu trước cho Task 15, human)

`.vscode/extensions.json`:

```json
{
  "recommendations": [
    "xdebug.php-debug",
    "bmewburn.vscode-intelephense-client",
    "Vue.volar",
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "editorconfig.editorconfig",
    "Anthropic.claude-code"
  ]
}
```

`.vscode/settings.json`:

```json
{
  "files.eol": "\n",
  "editor.formatOnSave": true,
  "php.validate.executablePath": "/usr/bin/php",
  "intelephense.files.exclude": ["**/Data/**", "**/node_modules/**"],
  "search.exclude": { "**/Data": true, "**/node_modules": true, "**/Resources/Public/app": true }
}
```

## Kiểm tra (Verify)

```bash
[WSL, ubuntu]
cd ~/projects/myproject
git status
ls backend frontend services infra
cat frontend/.nvmrc
cd frontend && npm run typecheck
```

## Ghi chú

- 14.6 (truy cập file từ Windows qua `\\wsl.localhost\flow-dev\home\ubuntu\projects`) là thông tin tham
  khảo cho member, không phải hành động cần Claude Code thực hiện — không cần task riêng.
- `.vscode/launch.json` (Xdebug) đã được tạo ở Task 10, mục 10.3.
