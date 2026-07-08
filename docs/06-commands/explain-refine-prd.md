[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /refine-prd

# `/refine-prd` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> Soi PRD để **làm nội dung tốt hơn** trước khi qua cổng chất lượng cuối. Đọc read-only, ghi biên bản lỗi, con người quyết. Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Hình dung `/refine-prd` như một **tổ soi bài** ngồi đọc PRD của bạn qua nhiều "cặp kính" khác nhau, tìm chỗ mô tả thiếu/mơ hồ/mâu thuẫn, rồi **ghi ra một danh sách góp ý** (findings). Nó **không tự sửa bài** — bạn duyệt từng góp ý rồi mới áp.

Nó là "anh em" với [`/review-context`](explain-review-context.md): dùng **chung một bộ máy soi** (fan-out nhiều lăng kính + người canh độ đầy đủ), nhưng `refine-prd` chạy **sớm hơn** — lo *cải thiện nội dung*; còn `review-context` là *cổng gác cuối*.

---

## Ba "cặp kính" (lăng kính)

Tổ soi bài gồm 3 vai, mỗi vai một góc nhìn:

- **DEV** — đọc như một dev sắp build: luồng đã đủ rõ để triển khai chưa, hay còn phải đoán?
- **SA** — đọc như một architect: các thực thể/luồng nghiệp vụ có thông suốt, nhất quán không?
- **PO** — đọc như chủ sản phẩm: mục tiêu, phạm vi, tiêu chí thành công có rõ không?

> **Quan trọng:** DEV và SA **đọc bằng mắt kỹ thuật nhưng VIẾT bằng lời nghiệp vụ** — chỉ ra chỗ nghiệp vụ mô tả thiếu để sau này xử lý được về kỹ thuật, chứ **không** kê giải pháp/cơ chế kỹ thuật vào PRD.
>
> *(Từng có lăng kính QA nhưng đang tạm tắt — vẫn giữ trong file dưới dạng block DISABLED kèm hướng dẫn bật lại.)*

---

## Cách soi cho kỹ (giống review-context)

- **Chia nhỏ:** mỗi lăng kính soi trên từng Use Case → nhiều "người soi đầu óc tươi", ít sót.
- **Người canh "đã đủ chưa?":** sau khi soi, đọc lại toàn bộ hỏi "còn sót gì không", lặp tới khi hai vòng liền không ra gì mới → **một lần chạy là đủ**.
- **Gộp & xử mâu thuẫn:** khử trùng lặp, hai góp ý chọi nhau thì nêu cả hai cho người chọn.

Kết quả là một file `…-findings.yaml`: mỗi góp ý ghi rõ ở đâu, trích nguyên văn, vấn đề gì, đề xuất sửa sao. Có xếp mức độ (critical/major/minor) và khuyến nghị (BLOCKED / NEEDS_REVISION / APPROVED).

---

## Sau khi có biên bản

Mở **Review Board**, duyệt từng góp ý (✓Nhận / ✎Sửa / ✗Bỏ) → chạy `--resume` để **áp những cái đã nhận**. Áp xong tự tăng version PRD + hạ về draft (phải duyệt lại).

Khi áp, nó **giữ đúng tầng**: cơ chế → BR/BL, AC chỉ giữ outcome + ref (không nhồi vào AC).

---

## Vị trí trong dây chuyền

```
generate-prd → [ refine-prd ◀ bạn ở đây ] → review-context (cổng cuối) → PO duyệt → generate-bdd
```

> **Một câu chốt:** `/refine-prd` là tổ soi bài 3 lăng kính (DEV/SA/PO) — đọc bằng mắt kỹ thuật, góp ý bằng lời nghiệp vụ — chạy sớm để *nâng chất nội dung PRD*; ghi biên bản, con người quyết, không tự sửa bài.
