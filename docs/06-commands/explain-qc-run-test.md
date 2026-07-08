[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /qc-run-test

# `/qc-run-test` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> **Trạm 5** của dây chuyền QC tự động — nơi ghi **kết quả QC CHÍNH THỨC** (`qc_status`). Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Trạm 5 đóng vai **QC Runner**: biến test case Markdown (đã review) thành **script Python thật** (pytest-playwright + Page Object), chạy chúng, và ghi kết quả pass/fail *chính thức* vào trace.

---

## Cách chạy

- **Markdown-first:** không sinh Python khi chưa có `.Test.md` đã review.
- Dùng stack riêng **qc-playwright** (độc lập với module code của dev). Page Object gọn 3 lớp (locator / action / assertion), dùng `expect()`, không hardcode, không `time.sleep`.
- **Locator từ contract test-id** (§4.5.6), không dò runtime; fallback role/text chỉ khi element thiếu id.
- Mỗi test gắn `@trace.verifies={UC-ID}-SC{N}`, phủ 100% TC (mỗi cái kết Pass/Fail/Skip).
- **Phân loại mỗi FAIL:** *script-bug* (QC tự sửa selector/logic) vs *product-gap* (defect thật — **giữ FAIL**, không bao giờ fake-pass).

## Nét quan trọng nhất: ghi `qc_status` chính thức

Đây là trạm **duy nhất** ghi `qc_status` vào trace `.tsv` — tín hiệu QC authoritative (khác hẳn `dev_selftest` của dev). Kèm theo:
- `qc_owner` — SC đang chờ ai: `dev` (product-gap) / `po` (bị chặn bởi spec gap) / `—` (pass).
- `qc_blocked_by` — `GAP-{id}` / `BUG-{id}` liên kết.

**Không bao giờ** đụng `dev_selftest` — hai tín hiệu tách biệt.

> **Một câu chốt:** `/qc-run-test` là QC Runner — sinh & chạy script Playwright từ test case đã review (locator theo test-id), rồi ghi `qc_status` **chính thức** + ai đang chờ; product-gap giữ FAIL, không fake-pass.
