[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /generate-code

# `/generate-code` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> Từ **BDD + tech-design đã duyệt**, sinh **code thật**. Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Hình dung `/generate-code` như một **thợ thi công theo bản vẽ**: đọc kịch bản (BDD) + bản vẽ kỹ thuật (tech-design), rồi viết code đúng theo đó, có gắn "nhãn truy vết" để sau này biết dòng code nào phục vụ kịch bản nào.

---

## Kỷ luật số 1: chỉ làm đúng một hạng mục (Scope Lock)

Thợ này **chỉ đọc và implement đúng một file feature** được giao — **không** ngó sang các `.feature` khác cùng folder, kể cả khi dùng chung entity. Tránh làm lan man ngoài phạm vi.

---

## Có thể làm "phần thô" trước, "đường ống thật" sau (Phase)

Với FE/App, code chia 2 pha để không phải đợi backend:

- **`--phase=ui`** — dựng UI + một **lớp mock** (giả lập API) dựa trên hợp đồng/System BDD. Tester test được ngay, không cần BE deploy.
- **`--phase=integration`** — thay mock bằng **lời gọi API thật** theo bản vẽ FE (§4). Giữ lại mock cho unit test.
- *(không flag)* — làm đầy đủ (thường cho BE).

**Bám Figma thật khi dựng UI:** nếu bật được **Figma Dev Mode MCP local** (app desktop), thợ đọc layout/token/component thật để code chính xác hơn nhiều link web trần; chưa bật thì gợi ý bật, hoặc skip (fallback text spec).

---

## Trước khi gõ code

- **Guard:** BDD/Design Spec chưa duyệt → cảnh báo mềm.
- **Đọc trace:** xem scenario nào chưa có code (UNTRACKED), scenario nào đã đổi (DRIFT) cần làm lại, cái nào bỏ qua.
- **File Scan:** phân loại mỗi file cần **CREATE** (mới) / **EXTEND** (chỉ thêm method, không viết lại code cũ) / **SKIP**.
- Hiện **plan** + chờ Y, rồi tạo **branch** riêng.

---

## Khi gõ code

- Viết theo **đúng thứ tự layer** trong CLAUDE.md (DTO → Entity → Repository → Service → Facade → Controller).
- Gắn **nhãn `@trace.implements`** ở layer entry-point (Controller/handler) kèm version PRD/BDD/tech-doc — để `/validate-traces` bắt được drift sau này.
- **(UI FE/App)** phát **test-id ổn định** cho mỗi element có action (theo §4.5.6 của tech-doc gộp) để QC định vị.
- Tự review 3 vòng, **build thử** (tối đa 3 lần retry), cập nhật trace `.tsv`, rồi commit.

---

## Vị trí trong dây chuyền

```
tech-docs approved → [ generate-code ◀ bạn ở đây ] → review-code → dev-gen-test → dev-run-test → QC
```

> **Một câu chốt:** `/generate-code` là thợ thi công bám bản vẽ — chỉ làm đúng một feature, có thể dựng UI-với-mock trước rồi lắp API thật sau, gắn nhãn truy vết từng dòng, và luôn build thử trước khi commit.
