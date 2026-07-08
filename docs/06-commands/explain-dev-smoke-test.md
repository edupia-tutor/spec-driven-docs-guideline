[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /dev-smoke-test

# `/dev-smoke-test` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> Kiểm nhanh một UC trên **service/app đang chạy thật**. Khác `/dev-run-test` (không cần server sống). Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Hình dung `/dev-smoke-test` như **thử máy tại chỗ**: service/app đã bật sẵn, ta gọi thẳng endpoint (BE) hoặc bấm thử luồng trên màn (FE/App) để xem "chạy thật có ra đúng không".

---

## Cách chạy (tuỳ platform)

- **Backend:** kiểm service đã chạy chưa (curl health endpoint) → tìm endpoint của UC (theo `@trace.implements`) → lấy token nếu cần → gọi `curl` GET/POST → đọc mã trạng thái (200 OK, 200-nhưng-sai-data → lỗi logic, 401/403/500 → theo bảng).
- **Web:** kiểm dev server → chạy E2E smoke (Playwright/Cypress lọc theo UC), hoặc thử tay trên browser.
- **Mobile:** cài build lên device/emulator → đi qua màn của UC, làm hành động chính (When) → xác nhận kết quả (Then) → hốt log nếu crash.

Kết quả là một bảng ✅ (ổn) / ⚠️ (chạy nhưng sai data → lỗi logic) / ❌ (lỗi/chưa chạy). Có vấn đề → dán vào `/debug` hoặc mở `/fix-bug`.

## Vị trí trong dây chuyền

```
generate-code → dev-gen-test → dev-run-test → [ dev-smoke-test ◀ thử máy thật ] → PR
```

> **Một câu chốt:** `/dev-smoke-test` là "thử máy tại chỗ" trên service/app đang chạy thật — gọi endpoint hoặc bấm luồng để xác nhận kết quả thực, khác với `/dev-run-test` chạy test không cần server sống.
