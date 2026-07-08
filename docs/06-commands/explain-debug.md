[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /debug

# `/debug` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> Phân tích nhanh một lỗi/hành vi lạ — **chỉ phân tích, không sửa, không cần ticket**. Khác `/fix-bug` (workflow đầy đủ). Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Hình dung `/debug` như **hỏi nhanh một đồng nghiệp rành việc**: bạn đưa lỗi (stack trace, test fail, hoặc câu hỏi code), nó chỉ ra *nguyên nhân gốc* và *cách sửa gợi ý* — gọn, không lôi cả quy trình ra.

---

## Cách chạy

Đầu tiên nó hỏi **tình huống của bạn là gì**:
1. Đã có stack trace/log → dán vào.
2. Cần tái hiện trước → nó chỉ lệnh chạy service (từ `service_run`).
3. Test đang fail → nó chỉ lệnh test, bạn dán output fail.
4. Câu hỏi code (không cần chạy) → hỏi thẳng.

Rồi nó phân tích:
- **Stack trace:** đọc từ **dưới lên** (`Caused by:` mới là gốc thật).
- Đối chiếu **bảng lỗi thường gặp** theo platform (BE / web / mobile / AI-LLM) để đoán nguyên nhân + hướng fix.
- **Test fail:** so Expected vs Actual, kiểm mock setup, kiểm assertion.

Ra một báo cáo: Lỗi · Nguyên nhân gốc · Vị trí (file/dòng/layer) · Cách sửa · Rule liên quan (CLAUDE.md) · Bước kế (`/fix-bug` nếu muốn sửa trọn vẹn).

Nếu nguyên nhân là **lỗi AI hay lặp khi sinh code** → hỏi có ghi thành *lesson* không.

## Vị trí trong dây chuyền

`/debug` là công cụ **cầm tay dùng bất cứ lúc nào** — không gắn cứng vào một chặng. Thường gọi khi `/dev-run-test` hoặc `/dev-smoke-test` báo lỗi.

> **Một câu chốt:** `/debug` là "hỏi nhanh đồng nghiệp" — phân loại tình huống, đọc lỗi từ gốc, tra bảng lỗi theo platform, ra nguyên nhân + cách sửa; chỉ phân tích, muốn sửa trọn vẹn thì sang `/fix-bug`.
