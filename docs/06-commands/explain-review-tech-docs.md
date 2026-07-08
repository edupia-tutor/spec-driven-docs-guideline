[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /review-tech-docs

# `/review-tech-docs` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> Soi **bản thiết kế kỹ thuật** trước khi cho sinh code. Read-only, ghi biên bản, con người quyết. Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Hình dung `/review-tech-docs` như một **hội đồng nghiệm thu bản vẽ**: soi bản vẽ kỹ thuật đúng chuẩn kiến trúc chưa, có khớp entity/BDD không, có đụng bản vẽ khác không — và với hợp đồng API liên team thì còn **thu đủ chữ ký** trước khi đóng dấu.

Giống các lệnh review khác: **không sửa bài**, chỉ ghi `…-tech-review-findings.yaml`; bạn duyệt ở Review Board rồi `--resume` để áp.

---

## Bảy mặt soi (T1–T7)

| Mặt | Soi gì |
|---|---|
| **T1 Architecture** | Có đúng thứ tự layer trong CLAUDE.md không (Controller không gọi thẳng Repository, logic không nằm ở Controller/DTO…). Vi phạm = **critical**, không auto-fix. |
| **T2 Entity** | Tên entity/field có khớp core-entities không. |
| **T3 BDD Traceability** | Thiết kế có bám scenario BDD không, có mâu thuẫn scenario không. |
| **T4 Domain Conflict** | Có đụng bản vẽ khác cùng domain không (trùng endpoint khác shape, trùng trách nhiệm…). |
| **T5 Internal Consistency** | Trong nội bộ doc có tự mâu thuẫn không (sơ đồ ≠ mô tả, return type ≠ code sketch…). |
| **T6 Completeness** | Đủ các section chuẩn chưa (thiếu = auto-fix thêm skeleton). |
| **T7 Cross-Team Sign-off** | *(chỉ hợp đồng API system, không phải brownfield)* thu chữ ký BE/FE/App/SA và đối chiếu contract với `Then` của web+app BDD. |

---

## Cổng chữ ký (chỉ hợp đồng API liên team)

Đây là nét riêng: một hợp đồng API mới **không được đóng dấu approved** cho tới khi **cả BE, FE, App (nếu có) và SA/Tech Lead đều ký "done"**. Còn ai `pending` → trạng thái giữ `in-review` → `/generate-code` bị chặn. Cơ chế này ép cả liên team đồng thuận contract *trước khi* ai đó bắt đầu code.

*(Nếu là brownfield — API đã tồn tại — thì bỏ qua T7: contract đã do PO chốt trong PRD, không có gì để đàm phán.)*

---

## Áp fix & đóng dấu

`--resume` áp các finding đã nhận (theo critical → major → minor), tăng `@trace.revision`, và đặt trạng thái: đủ chữ ký → **approved**; chưa đủ → **in-review**. Đồng thời cập nhật cột revision trong trace `.tsv`.

## Vị trí trong dây chuyền

```
generate-tech-docs → [ review-tech-docs ◀ bạn ở đây ] → (approved) → generate-code
```

> **Một câu chốt:** `/review-tech-docs` là hội đồng nghiệm thu bản vẽ kỹ thuật (7 mặt soi) — và với hợp đồng API liên team, nó còn là **cổng chữ ký**: chưa đủ BE/FE/App/SA ký thì chưa approved, chưa cho sinh code.
