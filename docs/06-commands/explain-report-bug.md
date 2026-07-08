[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /report-bug

# `/report-bug` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> Cho **tester & QC** lập một **phiếu bug có hồ sơ spec** — gắn ngữ cảnh (PRD/BDD/AC), đoán layer khả nghi, đẩy về team dev. Read-only trên spec/code. Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Hình dung `/report-bug` như **lập phiếu sự cố có truy vết**: không chỉ ghi "lỗi ở đâu", mà còn móc thẳng vào spec (AC nào bị vi phạm) và **đoán nguyên nhân nằm ở tầng nào** để định tuyến đúng người xử lý.

> **Chỉ ghi phiếu — không bao giờ sửa PRD/BDD/tech-docs/code.** Fix là việc `/fix-bug` (dev); viết scenario là `/propose-scenario`.

---

## Cách chạy

1. **Phân giải ngữ cảnh spec:** tìm PRD/BDD/tech-doc của UC (ưu tiên `spec-manifest.yaml`), lấy scenario fail, version PRD. Không có scenario nào khớp → đánh dấu *coverage gap*.
2. **Thu chi tiết:** xảy ra ở đâu, tái hiện sao, expected (theo spec) vs actual, log, env.
3. **Xác định AC bị vi phạm:** trích nguyên văn AC. Không AC nào phủ → có thể là PRD gap.
4. **Phân loại layer khả nghi (BUG_FLOW):** Code bug → `/fix-bug`; BDD bug → Dev/PO; PRD mơ hồ → PO; coverage gap → `/propose-scenario`; Design Spec bug → Dev/Designer; Env → DevOps.
5. **Ghi phiếu** `BUG-{ngày}-{NN}` vào `feedback/bug-reports/`, **backfill trace** (`qc_blocked_by`, `qc_owner`) để view "đang chờ ai" của PM trỏ đúng.
6. **Handoff:** commit + push lên spec repo — để PO/Dev thấy khi `/sync`.

Vòng đời: `Open → Fixed` (bởi `/fix-bug`) `→ Closed` (khi `/qc-run-test` verify pass).

> **Một câu chốt:** `/report-bug` là phiếu sự cố có hồ sơ spec — móc vào AC, đoán tầng nguyên nhân, đẩy lên spec repo để đúng người xử lý; chỉ ghi phiếu, không sửa gì.
