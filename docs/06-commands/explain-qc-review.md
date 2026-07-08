[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /qc-review

# `/qc-review` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> **Trạm 4** của dây chuyền QC tự động — một **cổng review chạy hai lần**. Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Trạm 4 đóng vai **QC Reviewer**: một **cổng kiểm** đặt ở hai chỗ trong dây chuyền, soi trước khi cho đi tiếp.

---

## Chạy hai lần, hai mode

1. **Sau Trạm 3 (design)** → review **test case** `.Test.md`: đã phủ mọi scenario chưa, có đủ happy + negative + boundary, expected có cụ thể, trace đầy đủ, không có TC "mồ côi".
2. **Sau Trạm 5 (run)** → review **script Python / Page Object**: khớp `.Test.md` 1-1 chưa, Page Object gọn 3 lớp, dùng `expect()` không phải assert trần, không hardcode URL/cred/timeout, không `time.sleep`, selector theo đúng thứ tự ưu tiên (test-id trước).

Nó tự **phát hiện mode** theo artifact target (`.Test.md` hay file Python). Ra findings + phán quyết **APPROVED** / **NEEDS_FIX**.

## Vị trí trong dây chuyền

```
qc-design-test → [ qc-review · lần 1: test case ] → qc-run-test → [ qc-review · lần 2: script ] → qc-report
```

> **Một câu chốt:** `/qc-review` là cổng kiểm hai chiều — lần 1 soi test case (sau thiết kế), lần 2 soi script (sau khi chạy) — phán APPROVED/NEEDS_FIX trước khi cho qua trạm kế.
