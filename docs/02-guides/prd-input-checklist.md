[📚 Docs](../README.md) › [Guides](README.md) › Checklist Input PRD

# Checklist Input PRD — Để PRD Chuẩn Ngay Lần Đầu

> Muốn `/generate-prd` ra PRD tốt **ngay lần đầu**, đừng trông vào `/refine-prd` để vá — hãy chuẩn bị đúng đầu vào trước.

## Hiểu trước cho đúng

- **PRD sinh ra từ một file khảo sát (product-definition)** — output của `/define-product`. Bạn **không gõ thẳng** PRD.
- **Không có "generate-prd cho BE" riêng.** PRD là tài liệu nghiệp vụ **dùng chung cho mọi platform** (một bản phục vụ cả FE/App/BE). Cái "system/BE" chỉ xuất hiện ở bước sau (`/generate-bdd`).
- Vì vậy "PRD chuẩn ngay lần đầu" = **chuẩn bị tốt 7 thứ dưới đây trước khi chạy `/generate-prd`**.

---

## 7 câu tự hỏi trước khi bấm `/generate-prd`

**1. Bản khảo sát đã làm xong chưa?**
Đã chạy `/define-product` đi hết 7 bước, không bỏ dở. Máy sẽ **không cho** sinh PRD nếu còn dang dở.
→ *Vì PRD là tài liệu để ký duyệt — không dựng từ thứ chưa chốt.*

**2. "Luật" và "cách xử lý" của tính năng đã viết rõ chưa?** *(quan trọng nhất)*
Hệ thống **phải làm gì / không được làm gì** (Business Rules), và **xử lý ra sao** trong từng trường hợp (Business Logic). Viết cụ thể, đừng để câu chung chung kiểu "xử lý hợp lý".
→ *Đây là phần dev dựa vào để code. Mơ hồ ở đây = code sai ở đó.*

**3. Đã liệt kê các trường hợp hỏng chưa?**
Không chỉ lúc chạy ngon, mà cả lúc lỗi: thiếu dữ liệu, điều kiện không thoả, hai người bấm cùng lúc, service khác chết.
→ *Hệ thống sống nhờ xử lý đúng mấy ca lỗi này; quên là lỗ hổng.*

**4. Tính năng này cần gì từ service/đội khác không?**
Ghi rõ: **cần dữ liệu/khả năng gì, lấy từ ai, để làm gì**. Nếu không cần thì ghi "Không có".
→ *Tính năng hiếm khi đứng một mình; thiếu mục này dễ tắc giữa chừng.*

**5. API của tính năng thuộc loại nào? — phải quyết trước**
Chọn 1 trong 3:
- **Đã có sẵn** (API chạy production rồi) → **đưa đường dẫn tới tài liệu API thật** (file swagger/openapi, link, hoặc thư mục code BE) **mà máy mở được**. Mở không được → PRD sẽ ghi "còn thiếu", chưa chuẩn.
- **Tự làm mới** → để trống, thiết kế sau ở `/generate-tech-docs`.
- **Đối tác đang làm song song** → chọn loại này (đừng chọn "đã có sẵn").
→ *Đây là chỗ quyết định độ chuẩn nhiều nhất cho tính năng BE.*

**6. Mỗi tiêu chí nghiệm thu (AC) có kiểm được đúng/sai rõ ràng không?**
Tránh kiểu "nhanh", "ổn". Phải đo được.
→ *AC mơ hồ thì sau này không ai biết tính năng "đạt" hay "chưa".*

**7. Tên gọi và dữ liệu đã dùng đúng từ điển chung chưa?**
Đảm bảo `business-dictionary.md` (từ chuẩn) và `core-entities.md` (danh sách thực thể/trường dữ liệu) đã cập nhật cho tính năng này.
→ *Gọi sai tên một bảng/trường là lệch cả chuỗi sau.*

---

## Checklist nhanh

- [ ] Khảo sát `/define-product` đã xong (7 bước, không gap)
- [ ] Business Rules + cách xử lý viết rõ, không mơ hồ *(đòn bẩy lớn nhất)*
- [ ] Đã liệt kê đủ các ca lỗi / trường hợp hỏng
- [ ] Phụ thuộc service/đội khác ghi rõ (hoặc "Không có")
- [ ] Đã quyết **loại API**; nếu "đã có sẵn" → đường dẫn contract **mở được**
- [ ] Mỗi AC kiểm được đúng/sai rõ ràng
- [ ] Từ điển chung + danh sách thực thể đã cập nhật
- [ ] Định danh đúng: Domain (khớp cấu hình service), PO, mã Ticket

---

## Riêng tính năng BE — chú ý nhất ở đâu?

BE không có Design Spec / màn hình đỡ phía trước, nên **PRD chính là spec**. Hai thứ quyết định độ chuẩn:

1. **Business Rules + Business Logic (câu 2)** — vì BE đọc PRD thẳng rồi sinh System BDD từ đây.
2. **Loại API (câu 5)** — BE "đã có sẵn" mà đường dẫn contract mở không được thì PRD sẽ kẹt ở trạng thái "còn thiếu".

> Tính năng BE thuần **không có màn hình** → phần Wireframe để trống/N/A là đúng; đừng để máy bịa màn hình.

---

## Sau khi sinh PRD

`/refine-prd` và `/review-context` là để **nhặt sạn** (làm rõ, bắt mâu thuẫn, thêm edge case) — **không phải** để vá cái thiếu từ khâu khảo sát. Chuẩn bị tốt 7 thứ trên thì hai bước này nhẹ tênh.

---

## Map sang các bước của `/define-product` *(cho ai muốn chính xác)*

| Câu hỏi | Nằm ở bước nào của discovery |
|---|---|
| 1. Khảo sát xong | toàn bộ Phase 1–7, `Status: completed` |
| 2. Luật + cách xử lý | Phase 4 (Business Rules) + Phase 5 (Business Logic) |
| 3. Ca lỗi | Phase 2 (Edge Cases) |
| 4. Phụ thuộc liên service | Phase 1, câu 8 |
| 5. Loại API | hỏi lúc `/generate-prd` (API Source) |
| 6. AC kiểm được | Phase 6 (Acceptance Criteria) |
| 7. Từ điển / thực thể | Phase 0 (Knowledge Sync) + `business-dictionary.md` / `core-entities.md` |

---

← [Guides](README.md) · Liên quan: [Checklist Input BDD (System/BE)](bdd-input-checklist.md) · [Quy tắc viết PRD](product-owner/prd-writing-rules.md) · [Checklist handoff](product-owner/handoff-checklist.md)
