[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /review-context

# `/review-context` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> Mô tả cách `/review-context` chạy bằng lời dễ hiểu + ví von đời thường. Bổ sung cho đặc tả chính thức ở [Reference › Commands](../05-reference/commands.md) và [command-cheatsheet](../05-reference/command-cheatsheet.md) — trang này ưu tiên *trực giác*, không phải đặc tả.

Hình dung `/review-context` như một **cửa kiểm định chất lượng cuối** trước khi tài liệu được "cho qua" sang bước sau. Nó **không sửa bài của bạn** — chỉ soi và ghi ra một danh sách lỗi để bạn tự quyết.

---

## Ví von: một tổ thanh tra trước cổng

Bạn đưa một tài liệu tới cổng (PRD hoặc BDD). Ở cổng có một **tổ thanh tra**. Quy trình như sau:

### 1. Bảo vệ nhìn tài liệu, biết ngay nó là loại gì
- Đuôi `.md` (ở gốc thư mục feature) → đây là **PRD** → dùng bộ tiêu chí **P** (P1–P5).
- Đuôi `.feature` → đây là **BDD** → dùng bộ tiêu chí **B** (B1–B6).

Hai loại giấy tờ, hai bảng kiểm khác nhau, nhưng **cùng một cái cổng**.

### 2. Chia nhỏ việc soi cho nhiều thanh tra — mỗi người một "cặp kính", mỗi người một phần
Thay vì một người ôm cả tài liệu soi mọi thứ cùng lúc (dễ sót), tổ trưởng **chia mỗi thanh tra soi đúng một khía cạnh, trên đúng một Use Case**. Người chỉ soi thuật ngữ, người chỉ soi câu mơ hồ, người chỉ soi cấu trúc… Ai cũng có "đầu óc tươi" nên soi kỹ hơn, ít sót hơn.

### 3. Một người canh "đã đủ chưa?" — chống bắt-cóc-bỏ-dĩa
Vấn đề muôn thuở: soi một lượt thì không bao giờ ra hết lỗi cùng lúc; chạy lại lần sau lại lòi lỗi mới (đập chuột chũi). Nên có một **người canh độ đầy đủ**: sau khi cả tổ soi xong, người này đọc lại **toàn bộ** tài liệu hỏi *"còn sót gì không?"*. Lặp tới khi **hai vòng liền không ra thêm gì mới** → mới chốt. Nhờ vậy **một lần chạy là đủ**, chạy lại sẽ ra 0 lỗi mới.

### 4. Hai việc tổ trưởng tự làm (không giao ai)
Có 2 kiểm tra cần nhìn ra ngoài tài liệu này nên tổ trưởng tự làm:
- **Định tuyến (P0):** PRD có ghi đúng "Domain" để sau này BDD/code chảy về đúng service không (chỉ ở chế độ umbrella).
- **Xung đột với PRD khác (P3):** feature này có mâu thuẫn luật với một PRD anh em nào không.

### 5. Soi hết → viết ra một tờ "biên bản lỗi", KHÔNG sửa bài
Kết quả là một file `…-findings.yaml`: mỗi lỗi ghi rõ *ở đâu, trích nguyên văn câu lỗi, vấn đề gì, đề xuất sửa thế nào, có tự sửa được không*. **Tài liệu gốc không bị đụng.** An toàn chạy bất cứ lúc nào.

---

## Sau khi có biên bản, bạn làm gì với nó?

Đây là chỗ **con người quyết**, có 2 đường:

- **Đường nhanh — `--fix`:** "cứ mấy lỗi máy tự sửa an toàn được (đổi thuật ngữ cấm, thêm khung section thiếu…) thì sửa ngay đi." Mấy lỗi cần đầu người vẫn để đó chờ.
- **Đường cẩn thận — Review Board + `--resume`:** bạn mở biên bản, duyệt **từng lỗi**: ✓Nhận / ✎Sửa lại đề xuất / ✗Bỏ. Xong bấm `--resume` để **áp những cái đã nhận**.

Áp xong, nó **tự tăng version** và **hạ trạng thái về `draft`** — vì tài liệu vừa đổi thì con dấu "đã duyệt" cũ hết hiệu lực, phải duyệt lại.

---

## Một mẹo tiết kiệm: lần đầu soi hết, lần sau chỉ soi chỗ đổi

- **Lần đầu** (chưa có biên bản cũ) → soi **toàn bộ** (full).
- **Lần sau** → chỉ soi những Use Case **đã thay đổi** (delta) cho nhanh. NHƯNG người-canh-đầy-đủ **vẫn đọc cả tài liệu** như lưới an toàn — phòng khi sửa chỗ này làm lộ lỗi chỗ kia.
- Nếu phát hiện tài liệu bị **người/lệnh khác** sửa ngoài tầm theo dõi → tự động quay lại soi **toàn bộ** cho chắc.

---

## Nó đứng ở đâu trong dây chuyền?

```
generate-prd → refine-prd → [ review-context: cổng chất lượng cuối ] → PO duyệt → generate-bdd
```

Với BDD thì nó là cổng trước `generate-tech-docs`.

> **Một câu chốt:** `refine-prd` lo *làm nội dung tốt hơn*; `review-context` là *cửa gác cuối* — soi thật kỹ, ghi biên bản, để con người quyết sửa gì; bản thân nó **không bao giờ tự ý sửa bài**.

---

*Muốn giải thích dễ hiểu cho một lệnh khác → thêm một trang `explain-{command}.md` trong thư mục này.*
