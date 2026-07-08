[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /dev-run-test

# `/dev-run-test` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> **Chạy** các test do `/dev-gen-test` sinh ra, phân tích lỗi, và ghi kết quả pass/fail vào trace. Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Hình dung `/dev-run-test` như **bấm nút chạy bộ tự-kiểm** rồi đọc kết quả: chạy test, cái nào fail thì tra bảng "lỗi thường gặp → nguyên nhân → cách sửa" theo đúng platform, và ghi lại "lần chạy gần nhất của dev: pass hay fail".

---

## Cách chạy

- **Chọn đúng lệnh test** theo module (`mvn test`, `go test`, `vitest`, `flutter test`…). Ở chế độ umbrella thì `cd` vào đúng service submodule trước (mỗi service có runner riêng).
- **Chạy** (có thể thu hẹp theo class cho nhanh).
- **Phân tích lỗi:** với mỗi test fail, đối chiếu bảng lỗi theo platform (vd BE "Expected 200 got 401 → thiếu setup auth"; FE "Unable to find role → element chưa render, bọc waitFor"; Flutter "pumpAndSettle timed out"…).
- **Ghi trace:** cập nhật `dev_selftest = pass/fail/not_run` + `dev_selftest_at`. **Không bao giờ** đụng `qc_status` — đó là tín hiệu QC chính thức, tách riêng.

## Vị trí trong dây chuyền

```
dev-gen-test → [ dev-run-test ◀ bạn ở đây ] → (pass) → review-code   |   (fail) → fix-bug / sửa test
```

> **Một câu chốt:** `/dev-run-test` chạy bộ tự-kiểm của dev, chẩn lỗi theo bảng-lỗi-từng-platform, và ghi kết quả `dev_selftest` — một tín hiệu riêng, độc lập với kết quả QC chính thức.
