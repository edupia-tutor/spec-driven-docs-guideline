[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /generate-design-spec

# `/generate-design-spec` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> Từ PRD, sinh **bản đặc tả thiết kế màn hình** cho FE/App — bám **Figma thật**. Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Hình dung `/generate-design-spec` như một người **biên soạn bản thiết kế màn hình**: đọc PRD để biết có những màn nào, rồi với **mỗi màn** mở đúng frame Figma tương ứng để tả layout, thành phần, các trạng thái (bình thường / đang tải / lỗi / trống) — dựa trên **thiết kế thật**, không bịa.

---

## Nét đặc thù: bắt buộc bám Figma theo từng màn

Đây là lệnh gen **duy nhất cần "vật liệu ngoài"** (Figma). AI **không đọc được link file Figma trần** — nó cần link **node-level tới từng frame** (URL có `?node-id=`). Nên nó thu **một link cho mỗi màn**, rồi fetch layout/component/token thật của frame đó về mà tả.

- Màn nào có link đọc được → tả dựa trên frame thật.
- Màn nào **chưa có link** → vẫn sinh nhưng đánh dấu ❌ Missing, và giữ **draft** (chưa cho ký duyệt) tới khi đủ link.

---

## Các bước kiểm trước khi sinh

- **PRD duyệt chưa?** Chưa → cảnh báo mềm (cho làm song song).
- **Platform nào?** backend → **dừng** (BE không có design spec). web / app / app-ios / app-android.
- **PRD đổi chưa (drift)?** Nếu design-spec cũ dựng từ PRD v cũ mà PRD đã đổi → hỏi cập nhật phần ảnh hưởng hay làm lại.
- **Chốt danh sách màn** với bạn, rồi thu link Figma từng màn, rồi CHECKPOINT xác nhận.

---

## Hai tầng ngôn ngữ (Design Language Guard)

Design Spec là **cầu nối PRD → code**, nên có hai loại bề mặt, ngôn ngữ khác nhau:

- **Tầng A — bề mặt đọc** (danh mục màn, mô tả layout, mô tả các trạng thái, hành động, AC-UI): **thuần nghiệp vụ/UX** — "thanh tiến độ", "nút chính", "trạng thái chưa chọn". KHÔNG nhét tên layer/variant Figma, mã màu hex, số đo px, hay ẩn dụ dữ liệu ("cờ").
- **Tầng B — cột/phụ lục kỹ thuật** (Component Inventory với tên code + import path, cột link Figma, bảng Design Token): **được giữ code/token** — vì đây là ngăn dành cho FE/Designer.

Nguyên tắc: **không xoá chi tiết kỹ thuật, dồn về đúng tầng.** (Xem thêm [Cơ chế · Business Language Guard](../03-concepts/mechanisms-explained.md).)

---

## Cổng tự-rà trước khi ghi

Trước khi ghi file, nó chạy một **checklist bắt buộc**: mọi màn có đủ đặc tả + 3 trạng thái tối thiểu, component đã map hoặc gắn cờ `[NEW]`, chỉ sinh phần đúng platform, AC-UI kiểm được, ngôn ngữ Tầng A sạch. Mục nào fail → ghi cảnh báo vào "Giả định AI" + giữ draft.

---

## Kết quả & vị trí trong dây chuyền

Ra `design-spec/{TICKET-ID}-design-spec-{platform}-{slug}.md`. Ký duyệt (PO + Designer đặt approved) **chỉ khi 0 màn Missing**.

```
PRD duyệt → [ generate-design-spec ◀ bạn ở đây ] → PO+Designer ký duyệt → generate-bdd (đọc AC-UI cho kịch bản FE)
```

> **Một câu chốt:** `/generate-design-spec` tả màn hình bám Figma thật (mỗi màn một link node-id), phân tầng ngôn ngữ (bề mặt đọc thuần nghiệp vụ, code/token dồn về cột kỹ thuật), và giữ draft cho tới khi đủ frame để ký duyệt.
