[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /generate-spec-manifest

# `/generate-spec-manifest` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> Quét toàn bộ file spec và ghi một **sổ mục lục** `spec-manifest.yaml` — cho **agent bên ngoài** (vd agent của tester) biết feature nào có PRD/BDD/tech-doc ở đâu. Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Hình dung `/generate-spec-manifest` như **lập danh mục kho tài liệu**: đi một vòng, ghi lại "feature X có PRD ở đây, BDD ở kia, tech-doc chỗ nọ, trạng thái/version bao nhiêu" thành một file index để công cụ/agent khác tra nhanh mà không phải mò.

---

## Cách chạy

- Xác định gốc quét (umbrella: spec submodule; single: project).
- **Quét PRD** (mỗi feature) → lấy TICKET-ID, domain, status, version.
- **Khớp** file product-definition, BDD, tech-doc theo từng TICKET-ID (theo glob + `@trace.prd`).
- **Ghi** `spec-manifest.yaml` (index gọn theo feature) và đảm bảo nó nằm trong `.gitignore` (file sinh ra, không commit).

Report còn cảnh báo PRD nào chưa có BDD/tech-doc khớp.

> **Một câu chốt:** `/generate-spec-manifest` lập sổ mục lục toàn bộ spec (feature → PRD/BDD/tech-doc/version) cho agent ngoài tra cứu — file sinh ra, gitignored, chạy lại bất cứ lúc nào.
