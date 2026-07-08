[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /qc-analyze

# `/qc-analyze` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> **Trạm 1** của dây chuyền QC tự động (`qc-analyze → qc-plan → qc-design-test → qc-review → qc-run-test → qc-report`). Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Hình dung dây chuyền QC như một **xưởng kiểm thử 6 trạm**. Trạm đầu — `/qc-analyze` — đóng vai **QC Analyst**: đọc spec chính thức (PRD + BDD + Design Spec) và **phân rã thành yêu cầu có cấu trúc** để các trạm sau dựa vào.

---

## Cách chạy

- **Guard:** BDD chưa duyệt → cảnh báo mềm.
- Phân rã spec thành: chức năng → **business rule** (`BR-xx`) → data flow → **acceptance criteria** (`AC-xx`); mỗi BR/AC **map tới scenario** `{UC-ID}-SC{N}` sở hữu nó.
- **Ghi DOC_GAPS:** mọi chỗ mơ hồ/thiếu/mâu thuẫn thành `GAP-xx` (không bao giờ tự bịa câu trả lời). Gap 🔴 Blocker → UC chưa sẵn sàng.
- **Đẩy defect thật lên PO:** blocker là lỗi trong spec chính thức → file qua `/report-bug` (hoặc `/propose-scenario` nếu thiếu coverage), không chỉ nằm local.

Ranh giới vai: `/qc-analyze` trả lời *"yêu cầu là gì?"*; trạm sau `/qc-plan` trả lời *"rủi ro ở đâu, hỏi dev gì?"*.

Ra 2 file: `REQUIREMENT_ANALYSIS.md` + `DOC_GAPS.md` (đặt ở `docs/{UC-ID}/` — folder QC nhìn thấy được, không phải spec repo của PO).

> **Một câu chốt:** `/qc-analyze` là QC Analyst — biến spec chính thức thành yêu cầu có cấu trúc (BR/AC map tới scenario) + danh sách gap, đẩy defect thật lên PO; nó *phân tích*, không viết test case.
