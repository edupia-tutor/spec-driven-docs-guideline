[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /propose-scenario

# `/propose-scenario` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> Cho **tester & QC** đề xuất một **kịch bản BDD mới** cho edge case chưa được phủ — dạng phiếu đề xuất để PO/Dev duyệt. Không đụng `.feature` chính thức. Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Hình dung `/propose-scenario` như **bỏ phiếu đề xuất vào hòm góp ý**: tester thấy một tình huống chưa có test, viết draft kịch bản Gherkin và gửi đề xuất — PO/Dev là người quyết đưa nó vào BDD thật.

> **Không sửa BDD chính thức.** BDD do PO/Dev sở hữu; lệnh này chỉ ghi proposal.

---

## Cách chạy

1. **Phân giải UC + platform** (để khớp vocabulary + tag trace, tránh đề xuất trùng).
2. **Quyết định coverage (mấu chốt):**
   - **Case A — behavior NẰM TRONG một AC có sẵn** (chỉ thiếu scenario) → draft Gherkin, map tới AC đó, tag `@proposed @from-test`.
   - **Case B — behavior KHÔNG nằm trong AC nào** (yêu cầu mới) → **không** draft BDD (không có gì để trace); thay vào đó ghi **PRD change request** để PO thêm/mở rộng AC.
3. **Ghi proposal** vào `feedback/bdd-proposals/` với `Status: proposed`.
4. **Handoff:** commit + push lên spec repo → PO/Dev thấy khi `/sync`.

Vòng đời: `proposed → accepted/rejected` (PO/Dev duyệt) → `/generate-bdd` chèn cái `accepted` vào `.feature` rồi `incorporated`.

> **Một câu chốt:** `/propose-scenario` là hòm góp ý test — draft kịch bản cho gap coverage (Case A) hoặc chuyển thành PRD change request nếu là yêu cầu mới (Case B); PO/Dev duyệt mới vào BDD.
