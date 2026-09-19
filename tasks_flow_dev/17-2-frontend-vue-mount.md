# Task 17-2: Frontend — mount component Vue vào thẻ gốc

**Loại:** Claude Code
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 17.2
**Phụ thuộc:** Task 14 (frontend đã khởi tạo)

## Mục đích

Viết `main.ts` (tìm mọi thẻ `[data-vue-component]`, đọc `data-props` làm props ban đầu rồi mount),
component `ProductList.vue`, CSS cơ bản, và `vite.config.ts` (dev HTTPS + CORS; build xuất vào package
Flow).

## Các bước thực hiện

`frontend/src/main.ts`:

```ts
import { createApp, type Component } from 'vue'
import './assets/main.css'
import ProductList from './components/ProductList.vue'

const registry: Record<string, Component> = { ProductList }

function mountAll(): void {
  document.querySelectorAll<HTMLElement>('[data-vue-component]').forEach((el) => {
    const name = el.dataset.vueComponent ?? ''
    const component = registry[name]
    if (!component) {
      console.warn(`[vue] Không tìm thấy component "${name}"`)
      return
    }

    let props: Record<string, unknown> = {}
    try {
      props = JSON.parse(el.dataset.props ?? '{}')
    } catch (e) {
      console.error(`[vue] data-props của "${name}" không phải JSON hợp lệ`, e)
    }

    createApp(component, props).mount(el)
  })
}

mountAll()
```

`frontend/src/components/ProductList.vue`:

```vue
<script setup lang="ts">
import { ref } from 'vue'

interface Product {
  id: number
  name: string
  price: number
}

const props = defineProps<{ initialProducts: Product[]; apiUrl: string }>()

const products = ref<Product[]>(props.initialProducts)
const loading = ref(false)
const error = ref<string | null>(null)

async function reload(): Promise<void> {
  loading.value = true
  error.value = null
  try {
    const res = await fetch(props.apiUrl, { headers: { Accept: 'application/json' } })
    if (!res.ok) throw new Error(`HTTP ${res.status}`)
    products.value = await res.json()
  } catch (e) {
    error.value = e instanceof Error ? e.message : String(e)
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <section class="product-list">
    <p class="badge">Vue 3 đã mount ✔</p>
    <ul>
      <li v-for="p in products" :key="p.id">{{ p.name }} — {{ p.price.toLocaleString('vi-VN') }} ₫</li>
    </ul>
    <button :disabled="loading" @click="reload">{{ loading ? 'Đang tải…' : 'Tải lại từ API' }}</button>
    <p v-if="error" class="error">Lỗi: {{ error }}</p>
  </section>
</template>
```

`frontend/src/assets/main.css`:

```css
body { font-family: system-ui, sans-serif; margin: 2rem; }
.badge { display: inline-block; padding: .2rem .6rem; border-radius: 999px; background: #d1fae5; color: #065f46; }
.error { color: #b91c1c; }
button { margin-top: .5rem; padding: .4rem .9rem; }
```

`frontend/vite.config.ts`:

```ts
import fs from 'node:fs'
import { fileURLToPath, URL } from 'node:url'
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// Chứng chỉ sinh bởi service local-dev-certs (Task 09; chỉ có trong WSL, Windows sẽ bỏ qua)
const CERT_DIR = process.env.DEV_CERT_DIR ?? '/etc/ssl/local-dev'
const keyPath = `${CERT_DIR}/local-dev-key.pem`
const certPath = `${CERT_DIR}/local-dev.pem`
const hasCert = fs.existsSync(keyPath) && fs.existsSync(certPath)

export default defineConfig({
  plugins: [vue()],
  resolve: { alias: { '@': fileURLToPath(new URL('./src', import.meta.url)) } },

  server: {
    host: true,
    port: 5173,
    strictPort: true,
    origin: `${hasCert ? 'https' : 'http'}://localhost:5173`,
    https: hasCert ? { key: fs.readFileSync(keyPath), cert: fs.readFileSync(certPath) } : undefined,
    cors: { origin: /^https?:\/\/([\w-]+\.)*localhost(:\d+)?$/ },
  },

  build: {
    outDir: '../backend/Packages/Application/Acme.App/Resources/Public/app',
    emptyOutDir: true,
    cssCodeSplit: false,
    rollupOptions: {
      input: 'src/main.ts',
      output: {
        entryFileNames: 'app.js',
        chunkFileNames: 'chunks/[name]-[hash].js',
        assetFileNames: (asset) => {
          const name = asset.names?.[0] ?? asset.name ?? ''
          return name.endsWith('.css') ? 'app.css' : 'assets/[name]-[hash][extname]'
        },
      },
    },
  },
})
```

## Kiểm tra (Verify)

```bash
[WSL, ubuntu]
cd ~/projects/myproject/frontend
npm run typecheck
```

## Ghi chú

- `outDir` trong `vite.config.ts` trỏ tới package Flow `Acme.App` được tạo ở **Task 17-1** — hai task này
  phụ thuộc lẫn nhau về đường dẫn, nên chạy sau khi Task 17-1 đã tạo cấu trúc `Packages/Application/Acme.App/`.
