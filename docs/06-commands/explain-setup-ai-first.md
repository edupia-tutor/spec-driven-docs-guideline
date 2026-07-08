[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /setup-ai-first

# `/setup-ai-first` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> Khởi tạo **một-lần** framework trong dự án: tạo thư mục, `CLAUDE.md`, config, từ điển, rồi verify. Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Hình dung `/setup-ai-first` như **dựng khung một xưởng mới**: đặt sẵn các phòng (thư mục), bảng nội quy (`CLAUDE.md`), sổ cấu hình (`project-context.yaml`), và từ điển thuật ngữ — để mọi lệnh sau có chỗ mà chạy.

---

## Cách chạy

1. **Kiểm đã setup chưa** (an toàn chạy lại — mỗi bước đề nghị merge/skip).
2. **Hỏi loại dự án:**
   - **Single-service** — một codebase (setup chuẩn đầy đủ).
   - **Umbrella** — repo chứa nhiều service submodule (chỉ tạo `.trace/` + `.agent/review/`; spec sống trong spec submodule).
   - **PO Spec repo** — chỉ docs, không code (tạo `specs/`, `feedback/`; `CLAUDE.md` tối thiểu).
3. **Tạo cấu trúc** thư mục theo loại (folder từng-feature thì tạo **on demand** bởi các lệnh generate, không tạo trước).
4. **Tạo file nền:** `CLAUDE.md` (nội quy: layer, coding standards, git), `project-context.yaml` (đường dẫn, tech stack), `business-dictionary.md`, `core-entities.md`.
5. **Gợi ý cài extension** VS Code (Review Board + Living Docs).
6. **Verify** checklist theo loại dự án.

Điểm nhấn: mọi PRD phải có row **Domain** đúng — team dev route BDD/code theo domain đó.

> **Một câu chốt:** `/setup-ai-first` dựng khung xưởng một-lần — thư mục + nội quy + config + từ điển theo 3 loại dự án (single / umbrella / PO spec) — để cả pipeline có nền mà chạy.
