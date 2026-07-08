[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /map-testids

# `/map-testids` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> Gán/backfill **test-id ổn định** lên các element FE **tái dùng hoặc đã có sẵn** — để QC định vị phần tử bằng id thay vì dò runtime. Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Hình dung `/map-testids` như **dán nhãn tên cố định** lên các nút/ô nhập trên giao diện. QC cần một cái "tay cầm" ổn định để tìm đúng element mà test; lệnh này lo phần khó: các component **dùng chung** (nhãn phải sống ở nơi sử dụng, và component phải *chuyển tiếp* được nhãn) và các màn **cũ/brownfield** đã code mà chưa có nhãn.

> `/generate-tech-docs` (FE) và `/generate-code` đã gán id cho code **mới**. Lệnh này lo phần **tái dùng + đã có sẵn**.

---

## Cách chạy (chỉ FE/App)

1. **Guard:** system/BE → dừng (không có UI test-id).
2. **Thu element có action** từ các step `When` của BDD + màn Design Spec (nút, ô nhập, link, chọn, toggle…), phân loại: *reused* (trong catalog dùng chung) / *existing* (đã code, brownfield) / *new* (để `/generate-code` lo).
3. **Gán id ổn định** theo quy ước `{uc}-{screen}-{element}-{type}` (vd `ft001-login-submit-btn`) — không nhúng số scenario; element đã có id thì dùng lại. Cùng element trên web & app dùng **cùng id value**.
4. **Đảm bảo component dùng chung chuyển-tiếp được id:** nếu chưa, patch component **một lần** để nhận + forward nhãn, rồi ghi vào figma-components catalog.
5. **Patch nơi sử dụng** (chỉ thêm nhãn, không refactor).
6. **Ghi map §4.5.6** vào tech-doc gộp (tạo file tối thiểu nếu chưa có).

## Vị trí trong dây chuyền

```
generate-tech-docs(FE) / generate-code → [ map-testids ◀ backfill id ] → review-tech-docs → qc-design-test
```

> **Một câu chốt:** `/map-testids` dán nhãn tên cố định cho element FE tái dùng/đã-có, đảm bảo component dùng chung chuyển-tiếp được nhãn, và ghi map §4.5.6 — để QC định vị bằng id thay vì dò runtime.
