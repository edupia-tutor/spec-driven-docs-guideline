[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /generate-tech-docs

# `/generate-tech-docs` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> Từ **BDD đã duyệt** của một PRD, sinh **một** bản thiết kế kỹ thuật **full-stack** — chỗ chi tiết kỹ thuật mà PRD cố tình không chứa. Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Hình dung `/generate-tech-docs` như một **bản vẽ thi công**: PRD/BDD nói "cần làm gì", còn bản vẽ này nói "làm bằng gì" (API, data model, luồng service, component, state, xử lý lỗi…). Đây là nơi chi tiết kỹ thuật được phép sống.

---

## Một PRD → một bản vẽ (merged, full-stack)

- **1 tech-doc / 1 PRD**, KHÔNG phải 1 doc / 1 UC. File gộp **cả backend lẫn client** trong cùng một tài liệu: §1–§4.4 + §6–§8 là phần BE (API contract, data model, DB); §4.5 là phần client (component, state, Figma mapping, test-id) cho mỗi platform; **§5 sequence diagram** nối xuyên tầng (component → service → API → external → DB). Một dev đọc một file là hiểu và code được cả tính năng.
- **Input = file BDD tech lead trỏ vào** — 1 file, hoặc một batch nhỏ (vd `system/` + `web/` + `app/` của cùng 1 UC). Lệnh nạp **chỉ** các file đó (giữ context nhỏ), **KHÔNG** tự gom hết BDD của PRD. Cảnh báo mềm nếu > 5 file/lần (không chặn).
- **Output:** `.../{prd-slug}/tech-docs/{TICKET-ID}-tech-design.md` (không hậu tố UC/platform).

---

## Sinh mới vs bổ sung (append)

- Doc **chưa tồn tại** → **Fresh:** sinh đầy đủ cho mọi UC hiện có.
- Doc **đã tồn tại** → **Append:** đọc **§10 UC Coverage** + Changelog để biết UC nào đã có; chỉ **thêm** section/sequence-diagram cho UC mới, cập nhật ma trận coverage + Changelog, **giữ nguyên** phần cũ (tôn trọng chỉnh tay + chữ ký reviewer). Không có UC mới và BDD không đổi version → báo "đã đầy đủ", không ghi đè.

Nhờ vậy, mỗi lần PRD có thêm BDD (thêm UC, thêm platform), bản vẽ **lớn dần lên** thay vì đẻ ra nhiều file rời.

---

## Trước khi vẽ, kiểm cửa

- **BDD sạch chưa?** Quét review-BDD findings của **mọi** feature trong PRD. Còn lỗi critical chưa xử → **dừng** (bắt `/review-context --fix/--resume`). BDD chưa duyệt → cảnh báo mềm.
- **(Client) có design-spec chưa?** Nếu PRD có `web/`·`app/` BDD, §4.5 lấy fidelity từ design-spec (Figma/component/token). Thiếu → **cảnh báo mềm**, §4.5 vẽ text-only từ BDD và đánh dấu `[DRAFT — no design-spec]` (không chặn cứng — phần BE vẫn đầy đủ giá trị).
- **Brownfield?** API đã tồn tại (`API Source: existing`) → chế độ **reverse-document**: mô tả lại contract as-is, ghi chú chỗ vênh, không thiết kế mới.

Rồi hiện **plan** (mode fresh/append, platform, danh sách UC, các section sẽ vẽ) và chờ Y.

---

## Sau khi vẽ

- Ra **một** file `{TICKET-ID}-tech-design.md` cho cả PRD.
- Nếu tech-doc sống trong spec repo dùng chung → **publish** (commit + push) để cả team `/sync` đọc được.
- Trạng thái ban đầu là **draft** — phải qua `/review-tech-docs` duyệt mới cho sinh code.

§4.5 client có **Test Selectors** — quy ước test-id ổn định cho từng element có action, làm *hợp đồng* để QC định vị phần tử mà không phải dò runtime (cùng một id value dùng chung web ↔ app, chỉ khác attribute).

---

## Vị trí trong dây chuyền

```
BDD duyệt (web+app+system của 1 PRD)
     → [ generate-tech-docs ◀ bạn ở đây ] → review-tech-docs (duyệt) → generate-code
       (1 bản vẽ full-stack / PRD, append khi có UC mới)
```

> **Một câu chốt:** `/generate-tech-docs` gom mọi BDD của một PRD thành **một** bản vẽ full-stack (BE + client + sequence xuyên tầng), sinh mới hoặc **bổ sung** khi có UC mới; ra draft, phải review mới cho code.
