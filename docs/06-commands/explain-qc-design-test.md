[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /qc-design-test

# `/qc-design-test` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> **Trạm 3** của dây chuyền QC tự động. Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Trạm 3 đóng vai **QC Designer**: viết ra các **test case** bằng Markdown (`.Test.md`) từ phân tích + kế hoạch. **Chưa** viết Python (đó là Trạm 5).

---

## Cách chạy

- Chọn **tầng test** (một màn / feature đa-màn / API / integration / e2e / non-functional / exploratory) và nạp đúng skill.
- Viết TC: mã `TC_<FEATURE>_<NNN>`, tách happy/negative, mỗi TC một mối quan tâm, **giá trị expected cụ thể**, priority P0/P1/P2, tags, status.
- **Tham chiếu test-id, không tả hình ảnh:** mỗi step GUI trỏ tới test-id ổn định trong §4.5.6 của tech-doc gộp (vd "click `ft001-login-submit-btn`") — để Trạm 5 dựng locator từ contract, không dò runtime.
- **Trace:** mỗi TC ghi `@trace.verifies={UC-ID}-SC{N}` (join key để Trạm 5 ghi `qc_status` theo scenario). TC bị chặn bởi gap vẫn viết + đánh dấu `🚫 Block: GAP-xx`.

Ra các file `.Test.md` trong `docs/{UC-ID}/test-cases/`.

> **Một câu chốt:** `/qc-design-test` là QC Designer — viết test case Markdown bám scenario, trỏ test-id thay vì tả hình ảnh; Python đến sau ở Trạm 5.
