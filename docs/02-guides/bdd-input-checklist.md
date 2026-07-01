[📚 Docs](../README.md) › [Guides](README.md) › Checklist Input BDD (System/BE)

# Checklist Input BDD (System / BE) — Để BDD Chuẩn Ngay Lần Đầu

> Áp cho `/generate-bdd` khi chọn platform **`system`** (BDD cho BE). Chuẩn bị đúng đầu vào để không phải sinh lại.

## Hiểu trước cho đúng: System BDD có HAI kiểu sinh

Khi chọn platform `system`, lệnh tự phân loại:

- **BE thuần** (feature không có web/app) → System BDD **sinh thẳng từ PRD** (AC / Business Rules / Business Logic).
- **BE trong feature đa-platform** → System BDD **tổng hợp từ web + app BDD đã có** (BE suy ra để phục vụ các luồng client).

→ Biết mình ở kiểu nào mới chuẩn bị đúng.

---

## Phần CHUNG — cả hai kiểu đều cần

**1. PRD đã duyệt + Luật/AC rõ ràng** *(đòn bẩy số 1)*
System BDD phủ **mỗi AC và mỗi Business Rule → ít nhất 1 scenario**. PRD mơ hồ ở Business Rules / Logic = BDD mơ hồ. Đây là chỗ quyết định nhiều nhất.

**2. Loại API đã chốt đúng trong PRD**
- **"Đã có sẵn"** (brownfield) → bảng **Existing API Contract trong PRD phải đầy đủ**, không còn dấu "⛔ còn thiếu". Lệnh dùng bảng này làm chuẩn và **bỏ qua bước tổng hợp**. Còn thiếu = BDD dễ bịa / sai shape.
- **"Tự làm mới"** → contract thiết kế sau; BDD chỉ tả hành vi nghiệp vụ.

**3. Từ điển + danh sách thực thể đã cập nhật**
System BDD viết bằng **ngôn ngữ sự kiện nghiệp vụ** ("hệ thống nhận X → trả về Y"), **không** dùng từ giao diện (click/tap), **không** chốt cứng shape JSON kỹ thuật. Tên thực thể / trường / enum phải đúng `core-entities.md`.

**4. Domain khớp cấu hình service** *(chế độ umbrella)*
Domain trong PRD phải khớp một service trong config; lệch là lệnh **dừng**.

---

## Phần RIÊNG theo kiểu

### Nếu BE trong feature đa-platform (tổng hợp)

**5. Đã sinh web + app BDD TRƯỚC — đúng thứ tự outside-in**
Nếu sinh `system` khi **chưa có** web/app BDD → lệnh tưởng là "BE thuần", sinh từ PRD và **bỏ lỡ** các kỳ vọng client thật (token, profile, redirect…). Phải theo thứ tự **web → app → system**.

**6. Sẵn sàng quyết "xung đột cross-platform"**
Nếu web và app **kỳ vọng khác nhau** (response / lỗi / luật) → lệnh **dừng ở CHECKPOINT** bắt PO chọn cách hoà: gộp chung / phân biệt theo platform / tách endpoint. Web+app BDD nên nhất quán, hoặc PO sẵn sàng quyết ngay — nếu không sẽ tắc.

### Nếu BE thuần

Bỏ qua câu 5–6. Dồn lực vào câu 1–3: PRD Business Rules / Logic + contract + thực thể.

---

## Checklist nhanh — trước khi `/generate-bdd` (system)

- [ ] PRD `approved`, mỗi AC/BR đủ rõ để suy ra scenario
- [ ] Loại API đã chốt; brownfield → bảng Existing API Contract **đầy đủ** (hết ⛔)
- [ ] `business-dictionary` + `core-entities` cập nhật (sự kiện / thực thể)
- [ ] Domain khớp cấu hình service (umbrella)
- [ ] *(đa-platform)* web + app BDD đã sinh + review **trước** system
- [ ] *(đa-platform)* sẵn sàng quyết xung đột cross-platform

---

## Sau khi sinh BDD

`/review-context` (BDD) bắt nốt sạn (coverage, Gherkin R1–R10, thuật ngữ); khi 0 critical → người duyệt đặt `# @trace.status: approved` rồi mới sang Tech Docs / Code / QC. Chuẩn bị tốt checklist trên thì bước review nhẹ.

---

← [Guides](README.md) · Liên quan: [Checklist Input PRD](prd-input-checklist.md) · [Checklist Input Tech-Docs (BE)](tech-docs-input-checklist.md)
