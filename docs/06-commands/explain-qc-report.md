[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /qc-report

# `/qc-report` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> **Trạm 6 (cuối)** của dây chuyền QC tự động. Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Trạm cuối đóng vai **QC Report**: biến lần chạy gần nhất thành **báo cáo + bằng chứng** chia sẻ được, và chuyển các lỗi sản phẩm về cho PO/Dev.

---

## Cách chạy

- Định vị artifact lần chạy gần nhất, sinh `report.html` (pytest-html self-contained) + **Playwright Trace** (`trace.zip`, xem bằng `playwright show-trace`), đính screenshot/evidence cho mỗi FAIL/SKIP.
- Tóm tắt TOTAL / PASS / FAIL / SKIP; mỗi FAIL ghi lệnh xem trace + phân loại script-bug hay product-gap.
- **Bàn giao product-gap về spec (có nhắc):** với mỗi FAIL là *product-gap* (impl ≠ spec), in sẵn lệnh `/report-bug {UC-ID} {expected-vs-actual}` để QC file vào spec repo — để PO/Dev thấy trên `/sync`. **script-bug KHÔNG file** (QC tự sửa). Không bao giờ fake-pass.

## Vị trí trong dây chuyền

```
qc-run-test → [ qc-report ◀ trạm cuối ] → (product-gap) /report-bug → validate-traces → PR
```

> **Một câu chốt:** `/qc-report` là trạm cuối — gói report + trace/evidence, và đẩy product-gap về PO/Dev qua `/report-bug`; script-bug thì QC tự lo, product-gap thì giữ nguyên FAIL cho tới khi fix.
