[📚 Docs](../README.md) › [Guides](README.md) › Checklist Input Tech-Docs

# Checklist Input Tech-Docs — Tech Lead Cần Làm Gì Để Gen Chuẩn

> Áp cho `/generate-tech-docs`. Từ **một hoặc vài file BDD bạn (tech lead) trỏ vào**, lệnh sinh/bồi đắp **một** tech-doc full-stack cho cả PRD. Chuẩn bị đúng đầu vào để dev đọc là code được, và FE không phải chờ sửa lại.

## Mô hình (đọc trước để trỏ lệnh cho đúng)

- **1 PRD = 1 file tech-doc** `{TICKET-ID}-tech-design.md` — gộp cả BE (API, data model, DB) lẫn client (§4.5 component/state/Figma) + §5 sequence xuyên tầng.
- **Input = file BDD bạn chỉ định**, KHÔNG phải cả PRD. Lệnh chỉ nạp đúng file bạn trỏ (giữ context nhỏ), rồi **ghi/bồi đắp** vào doc chung của PRD.
- **Append là đường chính:** chạy lệnh trên BDD này → lần sau trỏ BDD khác cùng PRD → doc **lớn dần lên**, không ghi đè phần cũ. §10 UC Coverage là "sổ điểm danh" để lệnh biết cái gì đã có.

```
BDD (web/app/system) ──/generate-tech-docs {file(s) bạn trỏ}──▶ {TICKET}-tech-design.md
        run 1: system/UC1        →  tạo doc, phần BE của UC1
        run 2: web/UC1 app/UC1   →  append §4.5 client cho UC1
        run 3: system/UC2 …      →  append UC2
```

## Bạn trỏ lệnh thế nào (batching)

`/generate-tech-docs` nhận **1 hoặc nhiều** file/UC trong một lần (space-separated) — coi là **một batch**, gộp vào doc trong lần chạy đó.

| Cách trỏ | Khi nào dùng |
|----------|--------------|
| `/generate-tech-docs system/{UC}.feature` | Vẽ phần BE (API contract) của 1 UC trước |
| `/generate-tech-docs {UC}` (system+web+app cùng UC) | Muốn cả full-stack của 1 UC trong 1 lần — trỏ đủ 3 file platform |
| `/generate-tech-docs system/UC1 system/UC2` | Bồi thêm nhiều UC backend một lượt |

- **Cảnh báo mềm nếu > 5 file/lần** (không chặn) — batch to = nạp nhiều context, chất lượng thiết kế giảm. Nên tách nhỏ: **per UC**, hoặc **per platform**.
- **Kinh nghiệm:** greenfield nên đi **system trước** (chốt API contract) → rồi mới trỏ `web/`·`app/` để append §4.5 bám đúng endpoint. Không bắt buộc (BE+FE cùng file) nhưng ít rework hơn.
- Trỏ lại UC **đã có** trong doc → lệnh hỏi xác nhận trước khi cập nhật (thêm platform còn thiếu / refresh khi BDD bump version).

---

## BDD khác tech-doc thế nào?

Hai cái trả lời **hai câu hỏi khác nhau** về cùng một tính năng:

| | **BDD (web/app/system)** | **Tech-doc (design)** |
|---|---|---|
| Trả lời | **CÁI GÌ** — hệ thống/màn phải làm gì (hành vi) | **LÀM SAO** — hiện thực kỹ thuật |
| Chi tiết kỹ thuật? | **KHÔNG** | **CÓ** — nơi *duy nhất* chứa endpoint/shape/DB/component/state |
| Ai làm | PO (spec repo) | Tech lead/Dev (`/generate-tech-docs`) |
| Dùng để | kiểm chứng hành vi (QC trace về đây) | dev code theo; FE đọc để gọi API thật |

```
PRD → BDD (CÁI GÌ) → /generate-tech-docs → tech-doc (LÀM SAO) → code
```

> Một câu: **BDD nói "đúng thì ra cái gì", tech-doc nói "gọi/dựng thế nào để ra cái đó".**

---

## Chuẩn bị trước khi `/generate-tech-docs`

**1. BDD được trỏ đã duyệt + sạch lỗi chưa?**
Mỗi file trong batch nên `@trace.status: approved` và **0 finding critical** từ `/review-context`. Lệnh **HALT** nếu còn lỗi critical; chưa approved → cảnh báo mềm.
→ *Tech-doc dẫn xuất từ BDD; BDD lỗi thì design lỗi theo.*

**2. BDD đã mô tả đủ hành vi chưa?**
Mỗi scenario rõ: **kết quả trả về**, **side-effect** (lưu/đổi gì), **tín hiệu lỗi** (báo gì khi nào).
→ *Nguyên liệu để suy ra endpoint, mã lỗi, data model, và §5 sequence. Thiếu = design thiếu.*

**3. Có client (web/app) trong batch → có design-spec chưa?**
§4.5 (component, Figma mapping, test-id, state) lấy fidelity từ **design-spec**. Thiếu → cảnh báo mềm, §4.5 vẽ tạm từ BDD và đánh dấu `[DRAFT — no design-spec]`.
→ *Muốn §4.5 chuẩn: chạy `/generate-design-spec` trước.*

**4. `core-entities.md` đã chuẩn chưa?**
§3 Data Model cần bảng/trường/kiểu/enum đúng. Sai tên một trường = lệch cả request/response lẫn DB.

**5. `CLAUDE.md` có kiến trúc + coding standard chưa?**
§2 Architecture + service flow bám **layer/kiến trúc project**. Thiếu → design chung chung, code sau lệch.

**6. Service trỏ đúng stack module chưa?** (java-spring / golang / dotnet / angular / …)
→ *Sai module = mẫu layer/component sai stack.*

**7. Brownfield? → bảng "Existing API Contract" trong PRD phải đầy đủ.**
`API Source: existing` → lệnh **chép lại** contract (reverse-document). Còn "⛔ thiếu" → tech-doc hổng.

**8. Phụ thuộc liên service đã ghi chưa?** (§1c PRD)
→ *Bỏ sót = §6 thiếu chỗ nối, đội khác tích hợp hụt.*

---

## Checklist nhanh — trước khi gõ lệnh

- [ ] Đã chọn **đúng file BDD** cần vẽ lần này (per UC / per platform); batch **≤ 5** file
- [ ] Mỗi file trong batch `@trace.status: approved` + **0 finding critical** (không → HALT)
- [ ] Mỗi scenario rõ: kết quả + side-effects + tín hiệu lỗi
- [ ] Có `web/`·`app/` trong batch → **design-spec sẵn** (không → §4.5 degraded)
- [ ] `core-entities.md` cập nhật (data model)
- [ ] `CLAUDE.md` có kiến trúc layer + coding standards
- [ ] Service trỏ đúng stack module
- [ ] Brownfield → bảng Existing API Contract trong PRD **đầy đủ** (hết ⛔)
- [ ] Phụ thuộc liên service (§1c PRD) ghi rõ nếu có

---

## Sau khi sinh tech-doc

- Ra **một** file `{TICKET-ID}-tech-design.md` (hoặc doc cũ được bồi thêm), trạng thái `draft`.
- `/review-tech-docs` chạy các check T1–T7. **T7 = cổng sign-off liên đội**: greenfield/partner cần FE/App/SA confirm contract trước khi `@trace.status: approved`; brownfield (`existing`) skip T7 vì contract đã chốt.
- Nếu doc sống trong spec repo dùng chung → **commit + push** để cả team `/sync` đọc.
- Approved rồi mới `/generate-code`.

---

← [Guides](README.md) · Liên quan: [Checklist Input BDD (System/BE)](bdd-input-checklist.md) · [Checklist Input PRD](prd-input-checklist.md) · [Giải thích /generate-tech-docs](../06-commands/explain-generate-tech-docs.md)
