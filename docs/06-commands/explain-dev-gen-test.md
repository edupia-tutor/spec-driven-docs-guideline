[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /dev-gen-test

# `/dev-gen-test` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> Sinh **test tự-kiểm của dev** (smoke) từ các kịch bản BDD — để dev tự tin code chạy trước khi review. Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Hình dung `/dev-gen-test` như **bộ tự-kiểm nhanh của thợ**: nhìn từng kịch bản BDD của một UC + code đã viết, rồi sinh các bài test tương ứng (unit / component / widget / e2e tuỳ platform) để dev tự chạy.

> **Quan trọng — đây KHÔNG phải bộ test chính thức của QC.** Nó là *dev self-check*: một tín hiệu lên Living Docs cho QC biết "dev đã tự chạy kiểm rồi". Bộ test authoritative có flow riêng của QC.

---

## Cách chạy

- **Nhận diện platform** từ `@trace.module` (backend / web-frontend / mobile) để chọn đúng khuôn test (java-spring, golang, react, flutter, swift…).
- Hiện **plan** (bao nhiêu scenario, file impl nào) → chờ Y.
- Sinh test, mỗi test gắn nhãn `@trace.verifies={UC-ID}`, mỗi scenario BDD có ≥1 test.
- Cập nhật trace `.tsv`: `test_count`, `test_classes`, và `dev_selftest = not_run` (đã có test nhưng chưa chạy — `/dev-run-test` mới set pass/fail).

Có quy tắc chất lượng theo platform: BE mock đúng layer, FE/Mobile không hardcode delay, dùng accessible query…

## Vị trí trong dây chuyền

```
generate-code → [ dev-gen-test ◀ bạn ở đây ] → dev-run-test → review-code
```

> **Một câu chốt:** `/dev-gen-test` sinh bộ tự-kiểm nhanh cho dev (theo platform, bám từng scenario BDD) — một tín hiệu self-check, không phải bộ test QC chính thức.
