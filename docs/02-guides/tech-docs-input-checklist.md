[📚 Docs](../README.md) › [Guides](README.md) › Checklist Input Tech-Docs (BE)

# Checklist Input Tech-Docs (BE) — Để API Contract Chuẩn Ngay Lần Đầu

> Áp cho `/generate-tech-docs` trên một file **System BDD** (`@trace.platform: system`) — sinh ra **BE tech-doc = API contract**. Chuẩn bị đúng đầu vào để FE không phải chờ sửa lại.

## Trước tiên: BDD khác tech-doc thế nào?

Hai cái trả lời **hai câu hỏi khác nhau** về cùng một tính năng BE:

| | **System BDD (BE)** | **BE Tech-docs (API contract)** |
|---|---|---|
| Trả lời | **CÁI GÌ** — hệ thống phải làm gì (hành vi nghiệp vụ) | **LÀM SAO** — hiện thực kỹ thuật |
| Viết bằng | sự kiện nghiệp vụ: "hệ thống nhận X → trả về Y → báo lỗi Z" | endpoint/method/path, **shape** request/response, HTTP status, data model, DB, error code |
| Chi tiết kỹ thuật? | **KHÔNG** | **CÓ** — nơi *duy nhất* chứa mấy thứ đó |
| Ai làm | PO (spec repo) | Dev (`/generate-tech-docs`) |
| Sinh từ đâu | từ PRD | **từ chính System BDD** |
| Dùng để | kiểm chứng hành vi (QC test trace về đây) | BE code theo; **FE đọc để gọi API thật** |

```
PRD  →  System BDD (CÁI GÌ)  →  /generate-tech-docs  →  BE tech-doc (LÀM SAO)  →  code
```

**Ví von:** System BDD = *"đủ tiền → máy nhả nước; thiếu tiền → báo lỗi, trả tiền lại"* (hành vi). Tech-doc = *bản vẽ kỹ thuật của máy*: `POST /vend` nhận `{coin}` trả `{drink_id, change}`, thiếu tiền trả `402`.

**Vì sao tách ra?** BDD không vỡ khi đổi chi tiết kỹ thuật (test ổn định); tech-doc là **contract liên đội** (FE phụ thuộc → cần sign-off T7); brownfield thì contract đã nằm sẵn trong PRD, tech-doc chỉ chép lại.

> Một câu: **BDD nói "đúng thì ra cái gì", tech-doc nói "gọi thế nào để ra cái đó".**

---

## 7 câu tự hỏi trước khi `/generate-tech-docs` (BE)

**1. System BDD đã duyệt + sạch lỗi chưa?**
BDD phải `@trace.status: approved` và **không còn finding critical** từ `/review-context`. Lệnh sẽ **dừng (HALT)** nếu BDD còn lỗi critical chưa xử lý.
→ *Tech-doc dẫn xuất từ BDD; BDD lỗi thì contract lỗi theo.*

**2. BDD đã mô tả đủ hành vi chưa?**
Mỗi scenario cần rõ: **kết quả trả về**, **các side-effect** (hệ thống lưu/đổi gì), và **tín hiệu lỗi** ("báo lỗi gì khi nào").
→ *Đây là nguyên liệu để suy ra endpoint, mã lỗi, và data model. BDD thiếu = contract thiếu.*

**3. API "đã có sẵn"? → bảng contract trong PRD phải đầy đủ**
Nếu là brownfield (`API Source: existing`), `/generate-tech-docs` chỉ **chép lại** contract từ PRD (reverse-document). Bảng "Existing API Contract" còn dấu "⛔ còn thiếu" → tech-doc sẽ hổng.
→ *Brownfield: đảm bảo contract đã trích đầy đủ từ nguồn thật.*

**4. Danh sách thực thể / dữ liệu đã chuẩn chưa?**
API contract cần **data model**: bảng/trường/kiểu/enum. Đảm bảo `core-entities.md` cập nhật đúng cho tính năng.
→ *Sai tên một trường ở đây là lệch cả request/response lẫn DB.*

**5. CLAUDE.md đã có kiến trúc + coding standard chưa?**
Tech-doc mô tả thiết kế theo **layer/kiến trúc của project**. Lệnh đọc `CLAUDE.md` (§architecture, §coding standards) để bám đúng quy ước.
→ *Thiếu → tech-doc tả kiến trúc chung chung, code sau lệch.*

**6. Đúng stack module cho service chưa?**
Service phải trỏ đúng module (java-spring / golang / dotnet / …) để lệnh lấy **mẫu layer** phù hợp.
→ *Sai module = mẫu thiết kế sai stack.*

**7. Phụ thuộc liên service đã ghi chưa?**
Nếu BE gọi/được gọi bởi service khác (§1c của PRD) → contract phải phản ánh các điểm tích hợp đó.
→ *Bỏ sót = contract thiếu chỗ nối, FE/đội khác tích hợp hụt.*

---

## Checklist nhanh — trước khi `/generate-tech-docs` (system/BE)

- [ ] System BDD `@trace.status: approved` + **0 finding critical** (không lệnh sẽ HALT)
- [ ] Mỗi scenario rõ: kết quả + side-effects + tín hiệu lỗi
- [ ] Brownfield → bảng Existing API Contract trong PRD **đầy đủ** (hết ⛔)
- [ ] `core-entities.md` cập nhật (data model: thực thể/trường/kiểu/enum)
- [ ] `CLAUDE.md` có kiến trúc layer + coding standards
- [ ] Service trỏ đúng stack module
- [ ] Phụ thuộc liên service (§1c PRD) ghi rõ nếu có

---

## Sau khi sinh tech-doc

`/review-tech-docs` chạy các check T1–T7. **T7 = cổng sign-off liên đội** (cross-team): greenfield/partner cần FE/App/SA confirm contract trước khi `@trace.status: approved`; brownfield (`existing`) thì skip T7 vì contract đã chốt sẵn. Contract chỉ thành "BE contract" mà FE chờ **sau khi approved**.

---

← [Guides](README.md) · Liên quan: [Checklist Input BDD (System/BE)](bdd-input-checklist.md) · [Checklist Input PRD](prd-input-checklist.md)
