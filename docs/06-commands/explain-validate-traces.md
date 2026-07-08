[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /validate-traces

# `/validate-traces` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> Kiểm **độ phủ** giữa spec ↔ code ↔ test (read-only), phát hiện lệch version, và làm mới dashboard Living Docs. Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Hình dung `/validate-traces` như một **đợt kiểm kê tồn kho**: đọc sổ theo dõi (`.trace/*.tsv`), đối chiếu với thực tế (file `.feature`, code, tech-doc), rồi báo mỗi kịch bản đang ở tình trạng nào — và có chỗ nào "lệch version" không.

---

## Cách chạy

- Nạp mọi trace `.tsv` (umbrella: gộp một chỗ trong spec repo).
- **Đối chiếu** với `.feature` hiện tại; tính **trạng thái mỗi scenario**:
  - `UNTRACKED` — chưa có code.
  - `GAP` — có code nhưng chưa có test.
  - `DRIFT` — spec đã đổi nhưng code sinh từ bản cũ.
  - `OK` — đủ code + test, khớp version.
- **Kiểm lệch version** upstream: PRD drift (code viết theo PRD cũ), tech-doc revision drift.
- **Ghi `trace-report.json`** — nguồn dữ liệu duy nhất cho web dashboard.
- **(umbrella)** Sync Living Docs + mirror panel để VS Code hiện đúng.

Ra một bảng tổng quan (PRD/UC/Scenario, % code cov, % test cov, drift, untracked, gap) + gợi ý lệnh sửa cho từng loại lệch. **Không sửa gì** ngoài status trong `.tsv` (và không đụng `dev_selftest`/`qc_status` — chỉ đọc).

> **Một câu chốt:** `/validate-traces` là đợt kiểm kê — đối chiếu spec↔code↔test, gắn nhãn OK/GAP/DRIFT/UNTRACKED, bắt lệch version, và xuất report cho dashboard; chỉ đọc, không sửa code.
