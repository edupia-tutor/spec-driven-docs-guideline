[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /generate-prd

# `/generate-prd` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> Biến **biên bản phỏng vấn** (product-definition) thành một **hồ sơ yêu cầu chuẩn** (PRD) — tài liệu chính thức để cả team dựa vào. Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Hình dung `/generate-prd` như một **thư ký biên tập**: nhận biên bản buổi khám phá, sắp xếp lại thành một hồ sơ đúng khuôn (Metadata, Use Case, AC, Business Rule, Wireframe…), đúng thuật ngữ chuẩn, thuần ngôn ngữ nghiệp vụ.

---

## Trước khi viết, thư ký kiểm mấy điều

- **Biên bản đã xong chưa?** Nếu product-definition chưa chốt đủ 7 chặng → **dừng**, bảo bạn chạy lại `/define-product` cho xong. PRD là hồ sơ ký duyệt, không dựng từ nguồn còn dang dở.
- **API thuộc loại nào?** Hỏi **một** câu để chốt loại nguồn API (bạn chỉ chọn loại, không gõ chi tiết):
  - *existing* — API đã chạy production (AI trích contract as-is vào Appendix, không bịa).
  - *greenfield* — tự thiết kế sau (ở tech-docs).
  - *partner* — đối tác làm song song, chưa có contract.
- **Có link ticket thật không?** AI không tự bịa URL Jira — hỏi bạn, có thì đặt link cạnh mã ticket, không thì để mã trơn.
- **Tên PO** lấy từ biên bản; thiếu thì hỏi.

---

## Viết PRD — kỷ luật quan trọng nhất: "đúng tầng"

Thư ký này rất kỹ về việc **cái gì viết ở đâu** (gọi là *altitude*):

- **AC** (nghiệm thu) = chỉ ghi *kết quả quan sát được* + trỏ số hiệu BR. KHÔNG nhồi cơ chế (số lần thử lại, timeout, nhánh lỗi chi tiết).
- **BR / Business Logic** = nơi chứa cơ chế "chạy thế nào".
- **Scope** = ranh giới (làm gì / không làm gì), không định nghĩa thuật ngữ.

Và **thuần nghiệp vụ**: chi tiết kỹ thuật (API, token, DB) → thuộc Tech Docs; chi tiết hình ảnh (màu, animation, pixel) → thuộc Design Spec. PRD chỉ giữ Wireframe ở mức nghiệp vụ (màn có gì, bấm ra kết quả gì).

Ngoài ra thư ký còn giữ **liên kết truy vết**: mỗi AC trỏ về ≥1 BR, mỗi UC liệt kê "AC liên quan", và hai chiều phải khớp nhau — lệch là lỗi, phải sửa trước khi ghi.

---

## Kết quả & vị trí trong dây chuyền

Ra file PRD `specs/{domain}/{prd-slug}/{TICKET-ID}-{prd-slug}.md` ở **version 1.0, trạng thái draft**.

```
define-product → [ generate-prd ◀ bạn ở đây ] → refine-prd → review-context → PO duyệt → generate-bdd
```

> **Một câu chốt:** `/generate-prd` là thư ký biến biên bản khám phá thành **hồ sơ chuẩn thuần nghiệp vụ**, canh đúng tầng AC/BR và giữ truy vết — rồi chuyển cho `refine-prd`/`review-context` soi lại trước khi PO duyệt.
