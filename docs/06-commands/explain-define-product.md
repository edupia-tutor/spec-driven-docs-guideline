[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /define-product

# `/define-product` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> Bước **đầu tiên** của cả dây chuyền: biến một ý tưởng tính năng trong đầu PO thành một **hồ sơ khám phá** (product-definition) đủ rõ để sau này viết PRD. Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Hình dung `/define-product` như một **buổi phỏng vấn khai thác ý tưởng có người dẫn**. AI đóng vai người phỏng vấn: hỏi bạn từng câu, ghi biên bản, và **chốt từng chặng** trước khi đi tiếp.

---

## Ví von: một buổi phỏng vấn 8 chặng, mỗi chặng có "chốt"

Bạn tới với một ý tưởng còn mơ hồ. AI dẫn bạn qua 8 chặng, hỏi **từng câu một** (không dồn một lúc), và sau mỗi chặng quan trọng phải nghe bạn nói "✅ đúng rồi" mới đi tiếp (gọi là **CHECKPOINT**):

1. **Chặng 0 — Tự tra bối cảnh:** AI tự quét dự án xem có khái niệm/rule/feature liên quan nào (không cần bạn trả lời). Nếu thấy thuật ngữ lạ lặp lại → để dành hỏi ở Chặng 3.
2. **Chặng 1 — Định nghĩa tính năng:** vì sao cần, giải quyết vấn đề gì, mong muốn kết quả gì, ai dùng, làm gì / không làm gì, user story.
3. **Chặng 2 — Luồng người dùng:** vào từ đâu, đi qua các bước nào, có những màn hình chính nào, ra khỏi ra sao, các tình huống lỗi.
4. **Chặng 3 — Hỏi cho hết mơ hồ:** AI tự thấy chỗ nào còn hổng thì hỏi thêm, lặp tới khi **không còn mục tồn đọng**.
5. **Chặng 4 — Business Rules:** rút ra các luật "hệ thống PHẢI / KHÔNG được làm gì".
6. **Chặng 5 — Business Logic:** mỗi luật chạy theo logic nghiệp vụ nào (rẽ nhánh, công thức).
7. **Chặng 6 — Acceptance Criteria:** tiêu chí nghiệm thu, mỗi cái phải kiểm được pass/fail.
8. **Chặng 7 — Tự kiểm độ phủ:** AI tự lập bảng "mỗi hành động có đủ Rule + Logic + AC chưa?", còn thiếu (GAP) thì cảnh báo.

### Nói bằng lời nghiệp vụ
Suốt buổi phỏng vấn, AI tránh hỏi chuyện kỹ thuật (API, database…) — chỉ khai thác **WHAT** (cần gì), không phải **HOW** (làm bằng gì).

---

## Bỏ dở cũng không sao — resume được

Sau mỗi chặng được chốt, AI ghi lại "đã xong tới chặng mấy" vào file. Nếu buổi phỏng vấn bị ngắt, lần sau chạy lại **đi tiếp từ chặng kế** — không phải làm lại từ đầu.

---

## Kết quả & vị trí trong dây chuyền

Ra một file `product-definition/{TICKET-ID}-{slug}.md`. Cái **slug** (tên rút gọn của feature) sinh ở đây và **dùng nguyên xuyên suốt** PRD, BDD, tech-docs, design-spec sau này.

```
[ define-product ◀ bạn ở đây ] → generate-prd → design-spec → generate-bdd → …
```

> **Một câu chốt:** `/define-product` là buổi phỏng vấn có người dẫn để **moi ý tưởng ra thành hồ sơ rõ ràng** — hỏi từng câu, chốt từng chặng, tự kiểm độ phủ; PRD chỉ được sinh khi buổi này xong hẳn (đủ 7 chặng).
