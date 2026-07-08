[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /update-framework

# `/update-framework` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> Nâng cấp **bản thân bộ đồ nghề framework** (command, steps, modules, templates, skills) lên version mới trên npm. Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Hình dung `/update-framework` như **thay bộ đồ nghề mới**: các slash command và khung template là "dụng cụ"; lệnh này lấy bản mới nhất về, thay dụng cụ cũ — nhưng **giữ nguyên đồ của bạn** (config, CLAUDE.md, từ điển, trace).

> Khác `/sync`: `/sync` lo **nội dung dự án** (chạy hằng ngày); `/update-framework` lo **nâng cấp dụng cụ** (chạy thỉnh thoảng, khi có bản mới).

---

## Cách chạy

1. **Phát hiện trạng thái:** đọc `FRAMEWORK_VERSION`, mode, các module đã cài (phải truyền lại khi nâng để chúng cũng update).
2. **Kiểm bản mới nhất** trên npm; đã mới nhất → dừng; có bản mới → hỏi Y.
3. **(umbrella)** nhắc: bộ đồ nghề chỉ sống ở umbrella root, service submodule không cần update riêng.
4. **Pre-flight git:** có thay đổi chưa commit trong `.agent/` → khuyên commit/stash trước (để review diff sạch).
5. **Chạy nâng cấp** (`npx @latest --init`) — **ghi đè** file framework (`.agent/commands`, `steps`, `templates`, `skills`…) nhưng **KHÔNG đụng** `project-context.yaml`, `CLAUDE.md`, `domain-knowledge/`, `.trace/`.
6. **Review changes:** `git diff --stat`, nêu command mới/đổi/bị bỏ, rồi bạn commit.

> **Một câu chốt:** `/update-framework` thay bộ đồ nghề (command/steps/templates) lên bản npm mới nhất, giữ nguyên nội dung của bạn; chạy thỉnh thoảng, khác với `/sync` chạy hằng ngày.
