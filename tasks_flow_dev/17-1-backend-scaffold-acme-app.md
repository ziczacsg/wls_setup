# Task 17-1: Backend — tạo package Flow `Acme.App`

**Loại:** Claude Code
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 17.1
**Phụ thuộc:** Task 14 (backend Flow đã khởi tạo, DB kết nối được)

## Mục đích

Tạo package `Acme.App` gồm: service nội bộ (`ProductProvider`), controller trang Fluid (`DemoController`),
controller API JSON (`ApiController`, đồng thời làm health-check), ViewHelper nạp JS/CSS Vue
(`ViteViewHelper`), template Fluid, và route.

## Các bước thực hiện

```bash
[WSL, ubuntu]
cd ~/projects/myproject/backend
./flow kickstart:package Acme.App
mkdir -p Packages/Application/Acme.App/Classes/{Controller,Service,ViewHelpers}
mkdir -p Packages/Application/Acme.App/Resources/Private/Templates/Demo
```

`Packages/Application/Acme.App/Configuration/Settings.yaml`:

```yaml
Acme:
  App:
    frontend:
      devServer: ''                # rỗng = nạp bản build trong Resources/Public/app
      entry: 'src/main.ts'         # entry của Vite (chỉ dùng khi có devServer)
      builtJs: 'app/app.js'
      builtCss: 'app/app.css'
```

Thêm vào `backend/Configuration/Development/Settings.yaml` (giữ nguyên phần MySQL đã có từ Task 14) để bật
Vite dev server ở Development:

```yaml
Acme:
  App:
    frontend:
      devServer: 'https://localhost:5173'
```

`Classes/Service/ProductProvider.php`:

```php
<?php
declare(strict_types=1);

namespace Acme\App\Service;

/**
 * Nguồn dữ liệu mẫu. Thực tế sẽ là Repository/Doctrine.
 * Cố ý KHÔNG có DI/AOP để Flow không tạo proxy → breakpoint Xdebug dừng chắc chắn (Task 17-7).
 */
class ProductProvider
{
    /** @return array<int, array{id:int, name:string, price:int}> */
    public function all(): array
    {
        $products = [
            ['id' => 1, 'name' => 'Bàn phím cơ', 'price' => 1200000],
            ['id' => 2, 'name' => 'Chuột không dây', 'price' => 450000],
            ['id' => 3, 'name' => 'Màn hình 27"', 'price' => 5600000],
        ];
        return $products;   // ← đặt breakpoint ở dòng này (Task 17-7)
    }
}
```

`Classes/Controller/DemoController.php`:

```php
<?php
declare(strict_types=1);

namespace Acme\App\Controller;

use Acme\App\Service\ProductProvider;
use Neos\Flow\Annotations as Flow;
use Neos\Flow\Mvc\Controller\ActionController;

class DemoController extends ActionController
{
    #[Flow\Inject]
    protected ProductProvider $productProvider;

    public function indexAction(): void
    {
        $this->view->assign('propsJson', json_encode([
            'initialProducts' => $this->productProvider->all(),
            'apiUrl' => '/api/products',
        ], JSON_THROW_ON_ERROR | JSON_UNESCAPED_UNICODE));
    }
}
```

`Classes/Controller/ApiController.php`:

```php
<?php
declare(strict_types=1);

namespace Acme\App\Controller;

use Acme\App\Service\ProductProvider;
use Doctrine\ORM\EntityManagerInterface;
use Neos\Flow\Annotations as Flow;
use Neos\Flow\Mvc\Controller\ActionController;
use Neos\Flow\Mvc\View\JsonView;

class ApiController extends ActionController
{
    protected $defaultViewObjectName = JsonView::class;
    protected $supportedMediaTypes = ['application/json'];

    #[Flow\Inject]
    protected ProductProvider $productProvider;

    #[Flow\Inject]
    protected EntityManagerInterface $entityManager;

    public function productsAction(): void
    {
        $this->view->assign('value', $this->productProvider->all());
    }

    public function healthAction(): void
    {
        $this->view->assign('value', [
            'flowContext' => getenv('FLOW_CONTEXT') ?: 'Development',
            'php' => [
                'version' => PHP_VERSION,
                'sapi' => PHP_SAPI,                        // mod_php → "apache2handler"
                'xdebug' => extension_loaded('xdebug'),
            ],
            'https' => $this->request->getHttpRequest()->getUri()->getScheme() === 'https',
            'mysql' => $this->checkMysql(),
            'redis' => $this->checkRedis(),
        ]);
    }

    private function checkMysql(): array
    {
        try {
            $version = $this->entityManager->getConnection()->fetchOne('SELECT VERSION()');
            return ['ok' => true, 'version' => $version];
        } catch (\Throwable $e) {
            return ['ok' => false, 'error' => $e->getMessage()];
        }
    }

    private function checkRedis(): array
    {
        if (!extension_loaded('redis')) {
            return ['ok' => false, 'error' => 'ext-redis chưa được nạp cho SAPI này'];
        }
        try {
            $redis = new \Redis();
            $redis->connect('127.0.0.1', 6379, 1.0);
            $redis->ping();
            $info = $redis->info('server');
            return ['ok' => true, 'version' => $info['redis_version'] ?? null];
        } catch (\Throwable $e) {
            return ['ok' => false, 'error' => $e->getMessage()];
        }
    }
}
```

`Classes/ViewHelpers/ViteViewHelper.php`:

```php
<?php
declare(strict_types=1);

namespace Acme\App\ViewHelpers;

use Neos\Flow\Annotations as Flow;
use Neos\Flow\ResourceManagement\ResourceManager;
use Neos\FluidAdaptor\Core\ViewHelper\AbstractViewHelper;

/**
 * Xuất thẻ nạp frontend Vue:
 *  - devServer có giá trị → nạp module trực tiếp từ Vite dev server (hot-reload)
 *  - devServer rỗng       → nạp bản build trong Resources/Public/app (production)
 */
class ViteViewHelper extends AbstractViewHelper
{
    protected $escapeOutput = false;

    #[Flow\InjectConfiguration(path: 'frontend', package: 'Acme.App')]
    protected array $config = [];

    #[Flow\Inject]
    protected ResourceManager $resourceManager;

    public function render(): string
    {
        $devServer = rtrim((string)($this->config['devServer'] ?? ''), '/');

        if ($devServer !== '') {
            $server = htmlspecialchars($devServer, ENT_QUOTES);
            $entry = htmlspecialchars(ltrim((string)($this->config['entry'] ?? 'src/main.ts'), '/'), ENT_QUOTES);
            return '<script type="module" src="' . $server . '/@vite/client"></script>' . "\n"
                 . '<script type="module" src="' . $server . '/' . $entry . '"></script>';
        }

        $css = $this->publicUri((string)($this->config['builtCss'] ?? 'app/app.css'));
        $js = $this->publicUri((string)($this->config['builtJs'] ?? 'app/app.js'));
        return '<link rel="stylesheet" href="' . htmlspecialchars($css, ENT_QUOTES) . '">' . "\n"
             . '<script type="module" src="' . htmlspecialchars($js, ENT_QUOTES) . '"></script>';
    }

    private function publicUri(string $path): string
    {
        return $this->resourceManager->getPublicPackageResourceUriByPath(
            'resource://Acme.App/Public/' . ltrim($path, '/')
        );
    }
}
```

`Resources/Private/Templates/Demo/Index.html`:

```html
{namespace app=Acme\App\ViewHelpers}
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Acme.App – Fluid wrapper + Vue 3</title>
    <app:vite />
</head>
<body>
    <div data-vue-component="ProductList" data-props="{propsJson}"></div>
</body>
</html>
```

Nối thêm vào cuối `backend/Configuration/Routes.yaml`:

```bash
[WSL, ubuntu]
cd ~/projects/myproject/backend
printf '\n' >> Configuration/Routes.yaml
cat >> Configuration/Routes.yaml <<'EOF'
-
  name: 'Acme.App - demo (Fluid + Vue)'
  uriPattern: 'demo'
  defaults:
    '@package': 'Acme.App'
    '@controller': 'Demo'
    '@action': 'index'
    '@format': 'html'

-
  name: 'Acme.App - api products'
  uriPattern: 'api/products'
  defaults:
    '@package': 'Acme.App'
    '@controller': 'Api'
    '@action': 'products'
    '@format': 'json'
  httpMethods: ['GET']

-
  name: 'Acme.App - api health'
  uriPattern: 'api/health'
  defaults:
    '@package': 'Acme.App'
    '@controller': 'Api'
    '@action': 'health'
    '@format': 'json'
  httpMethods: ['GET']
EOF

./flow flow:cache:flush
./flow flow:package:list | grep -i 'Acme.App'
```

## Kiểm tra (Verify)

```bash
[WSL, ubuntu]
cd ~/projects/myproject/backend
./flow flow:package:list | grep -i 'Acme.App'
```

Kiểm tra HTTP đầy đủ sẽ thực hiện ở **Task 17-4** (sau khi frontend cũng đã sẵn sàng — Task 17-2, 17-3).

## Ghi chú

- Mã scaffold viết theo Flow 9.x (PHP attributes). Nếu bản Flow của bạn khác, đối chiếu cú pháp
  annotation/attribute và API với tài liệu Flow.
- Nếu gặp lỗi *Access Denied*, kiểm tra `Policy.yaml`.
- Fluid tự escape `{propsJson}` (dấu `"` → `&quot;`) nên an toàn khi đặt trong attribute; trình duyệt giải
  mã lại trước khi Vue đọc bằng `dataset.props`.
