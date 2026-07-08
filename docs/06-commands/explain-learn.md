[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /learn

# `/learn` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> Ghi một **bài học của dự án** (guardrail) để AI **không lặp lại một lỗi**. Bài học được nạp vào context ở đầu mỗi lệnh. Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Hình dung `/learn` như **dán một tờ nhắc lên tường xưởng**: "lần trước AI hay làm sai X → từ nay phải làm Y". Mỗi lần chạy lệnh, AI đọc lại tấm bảng nhắc này nên không mắc lại.

> Đây là **bộ nhớ dự án**, không phải huấn luyện lại model — nó thành guardrail nhờ được inject vào context mỗi lần.

---

## Cách chạy

- Bạn gõ `/learn {mô tả tự do}` — lý tưởng kiểu *"AI làm X, nên làm Y"*.
- Nó tách thành **lỗi** (AI làm sai gì) + **quy tắc** (câu sửa mệnh lệnh "Luôn…/Không bao giờ…"), suy ra `category` (code-gen / bdd / tech-docs / tests / prd / general) và `scope` (domain / file glob / all).
- Hiện lại cho bạn xác nhận → ghi vào file lessons của dự án.

Guardrail có hiệu lực ngay từ lệnh kế. Nhiều lệnh khác (`/review-code`, `/debug`, `/fix-bug`) cũng **tự đề nghị** ghi lesson khi thấy một lỗi AI hay lặp.

> **Một câu chốt:** `/learn` dán tờ nhắc lên tường xưởng — biến một lỗi hay lặp thành quy tắc, được nạp vào context mỗi lần chạy để AI không mắc lại; commit file lessons để cả team dùng chung.
