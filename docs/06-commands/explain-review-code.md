[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /review-code

# `/review-code` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> Soi **code** theo 4 mặt trước khi cho đi test. Read-only — chỉ report, không sửa. Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Hình dung `/review-code` như một **giám sát công trình** đi soi phần code vừa thi công: đúng bản vẽ chưa, đúng chuẩn chưa, có bỏ sót kịch bản nào không. Nó **không tự sửa** — chỉ ra báo cáo và phán "đạt" hay "cần sửa".

---

## Bốn mặt soi

| Mặt | Soi gì |
|---|---|
| **1. Traceability** | Mỗi endpoint có nhãn `@trace.implements` chưa; nhãn có đặt đúng layer không; trace `.tsv` có cập nhật không. |
| **2. Layer Architecture** | Mỗi class đúng layer chưa, phụ thuộc đi đúng chiều, không bypass layer (theo CLAUDE.md §2). |
| **3. Coding Standards** | Naming, response wrapper, exception không bị nuốt, không magic number, không log dữ liệu nhạy cảm, transaction đúng (CLAUDE.md §3). |
| **4. Spec Compliance** | Mỗi scenario BDD có code phủ; không có endpoint "lậu" (code không có spec đỡ lưng). |

Trước khi soi, nó liệt kê **scope** (file nào gắn `@trace.implements={UC-ID}`, bao nhiêu scenario) và chờ Y.

---

## Kết quả

Một báo cáo phân theo mức Critical / Major / Minor (file · dòng · vấn đề · gợi ý sửa), và **phán quyết**:

- **APPROVED ✅** → đi tiếp `/dev-gen-test`.
- **NEEDS_FIX ❌** → sửa nhỏ inline rồi review lại / lỗi logic-kiến trúc → `/fix-bug` / code lệch spec → `/generate-code` lại.

**Không tạo artifact** (read-only thuần).

---

## Học từ lỗi lặp

Nếu một finding Critical/Major trông như **lỗi AI hay lặp** khi sinh code, nó hỏi bạn có muốn ghi thành **project lesson** để `/generate-code` lần sau không lặp lại — một vòng phản hồi giúp framework tự khá lên.

## Vị trí trong dây chuyền

```
generate-code → [ review-code ◀ bạn ở đây ] → (APPROVED) → dev-gen-test → dev-run-test
```

> **Một câu chốt:** `/review-code` là giám sát công trình soi code 4 mặt (truy vết · kiến trúc · chuẩn · bám spec), chỉ báo cáo không sửa, phán APPROVED/NEEDS_FIX, và có thể ghi lại lỗi lặp thành bài học cho lần sinh code sau.
