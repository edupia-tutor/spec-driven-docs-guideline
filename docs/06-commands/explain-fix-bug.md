[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /fix-bug

# `/fix-bug` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> Workflow **fix bug đầy đủ**: tìm nguyên nhân → sửa → test hồi quy → build → commit → đóng bug. Khác `/debug` (chỉ phân tích). Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Hình dung `/fix-bug` như **quy trình xử lý sự cố có hồ sơ**: không chỉ vá, mà còn truy nguyên nhân gốc, thêm test chống tái phát, build kiểm, commit đúng nhánh, và đóng phiếu bug.

---

## Sáu chặng

1. **Gom thông tin:** nhận ticket / một `{BUG-ID}` đã file (từ `/report-bug` — đọc luôn spec context, AC bị vi phạm) / hoặc mô tả (hỏi: xảy ra ở đâu, tái hiện sao, expected vs actual, log).
2. **Truy nguyên nhân gốc:** tra bảng "loại bug → chỗ hay gặp → cách kiểm" theo platform. Ra **CHECKPOINT** (root cause, file ảnh hưởng, fix đề xuất, mức rủi ro hồi quy) → chờ Y.
3. **Sửa:** tạo branch `fix/...` (umbrella thì `cd` vào service submodule), áp fix, gắn nhãn `@trace.fixes`.
4. **Test hồi quy:** thêm một test tái hiện bug (chống tái phát), chạy tới khi xanh (tối đa 3 vòng).
5. **Build & commit:** build kiểm, commit; umbrella thì **push 2 tầng** (service submodule → bump pointer umbrella).
6. **Đóng bug (nếu fix một `{BUG-ID}`):** đặt `State: Fixed` + ghi Resolution. **Chưa Closed** — QC sở hữu việc verify: khi `/qc-run-test` chạy lại pass thì mới `Closed`.

Cuối cùng có thể hỏi ghi *lesson* nếu nguyên nhân là lỗi AI hay lặp.

> **Ranh giới:** `/fix-bug` chỉ sửa **code**, không bao giờ sửa PRD/BDD (đổi spec là việc PO/Dev theo bug-flow).

## Vị trí trong dây chuyền

`/fix-bug` gọi khi review/test/QC phát hiện bug thật (từ `/review-code`, `/dev-run-test`, hoặc `/report-bug` của QC).

> **Một câu chốt:** `/fix-bug` là xử lý sự cố có hồ sơ — truy gốc, sửa code (không đụng spec), thêm test hồi quy, build + commit 2 tầng, và chuyển bug sang Fixed để QC verify đóng.
