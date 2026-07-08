[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /sync

# `/sync` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> Một lệnh **đồng bộ dự án** umbrella: git pull + xử lý submodule + nổi feedback tester + bootstrap config service + làm mới Living Docs. Chạy hằng ngày. Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Hình dung `/sync` như **nghi thức đầu ngày**: kéo về cái mới nhất, dọn dẹp, và báo cho bạn "có gì mới cần để mắt" — nhưng **cẩn thận không đụng vào chỗ bạn đang làm dở**.

> Khác `/update-framework`: `/sync` lo **nội dung dự án** (code/specs submodule + Living Docs); `/update-framework` lo **nâng cấp bản thân framework**.

---

## Cách chạy

1. **Pre-flight:** kiểm là git repo, đọc config (spec_source, services), quét trạng thái submodule (conflict → dừng).
2. **Pull umbrella** + init submodule chưa clone.
3. **Xử lý từng submodule theo trạng thái** — điểm tinh tế nhất: **không bao giờ ép branch**. Submodule bạn đang đứng trên một branch (active) → chỉ fetch, để nguyên; detached sạch (passive) → align về pointer; đang sửa dở (dirty) → cảnh báo, skip. Riêng **spec submodule** thì cố ý advance tới branch mới nhất (PO push liên tục).
4. **Nổi feedback tester/QC** vừa pull về (bug report *Open*, scenario proposal, PRD change request) — để PO/Dev được thông báo qua routine bình thường.
5. **Bootstrap config service** (tự tạo `.agent/project-context.yaml` cho service thiếu, đoán module từ `pom.xml`/`go.mod`/`package.json`…).
6. **Làm mới Living Docs** + spec-manifest.

Báo cáo cuối gom mọi thứ: git, từng submodule, feedback mới, config service, Living Docs.

> **Một câu chốt:** `/sync` là nghi thức đầu ngày cho umbrella — pull + đồng bộ submodule (tôn trọng branch đang làm), nổi feedback tester, tự tạo config service, làm mới dashboard; an toàn chạy lại.
