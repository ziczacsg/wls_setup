# Task 17-7 (human): Kiểm chứng Xdebug + VSCode

**Loại:** Con người (Human)
**Nguồn:** `WSL-DEV-ENVIRONMENT-GUIDE.md` § Bước 17.7
**Phụ thuộc:** Task 10 (Xdebug config), Task 15 (VSCode Remote - WSL), Task 17-1 (`ProductProvider` tồn
tại)

## Vì sao cần con người thực hiện

Đặt breakpoint, bấm F5, và quan sát VSCode dừng đúng chỗ là thao tác tương tác trong GUI của VSCode —
không tự động hoá được từ Claude Code.

## Các bước thực hiện

1. Mở dự án bằng `code .` (Remote - WSL) — đã làm ở Task 15. Cài **PHP Debug** *trong WSL* nếu chưa có,
   `.vscode/launch.json` đã được tạo sẵn ở Task 10.
2. Đặt breakpoint tại dòng `return $products;` trong
   `backend/Packages/Application/Acme.App/Classes/Service/ProductProvider.php`.
3. Bấm **F5** (Listen for Xdebug).
4. Mở `https://flow.localhost:8443/demo?XDEBUG_TRIGGER=1`.
5. VSCode phải dừng tại breakpoint, xem được biến `$products`. Bấm F5 để tiếp tục.

## Tiêu chí hoàn thành

VSCode dừng đúng tại breakpoint và hiển thị đúng giá trị biến `$products`.

## Ghi chú

Nếu không dừng: chạy `xdebugctl status` trong WSL, kiểm tra `php -v` có Xdebug, extension PHP Debug đã cài
trong WSL (không phải trên Windows), và không có tiến trình nào khác đang giữ cổng 9003.

⚠️ Flow biên dịch *proxy class* (AOP/DI) vào `Data/Temporary/<Context>/Cache/Code/`. Breakpoint ở
`ProductProvider` (không có DI/AOP) được chọn có chủ đích để chắc chắn dừng; nếu cần debug class bị proxy
(vd controller có `#[Flow\Inject]`), đặt breakpoint ở file tương ứng trong thư mục cache.
