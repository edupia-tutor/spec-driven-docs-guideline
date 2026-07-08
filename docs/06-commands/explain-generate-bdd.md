[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /generate-bdd

# `/generate-bdd` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> Biến **PRD đã duyệt** (+ Design Spec) thành các **kịch bản kiểm thử** viết bằng ngôn ngữ nghiệp vụ (file `.feature` — Gherkin Given/When/Then). Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Hình dung `/generate-bdd` như một người **biến yêu cầu thành kịch bản diễn thử**: đọc PRD, viết ra từng tình huống "cho trạng thái này → khi làm hành động kia → thì kết quả phải thế này". Mỗi AC/BR đều phải có ít nhất một kịch bản phủ.

---

## Trước khi viết, hỏi/kiểm mấy điều

- **PRD duyệt chưa?** Chưa duyệt (draft) → cảnh báo mềm, hỏi có làm tiếp không (cho phép prototype).
- **Cho platform nào?** web / app / **system** (BDD tầng hệ thống). Từ vựng viết kịch bản đổi theo: web "clicks", app "taps", system "the system returns" (không dùng từ UI).
- **Có Design Spec chưa?** (chỉ FE/App) — nếu có và đã approved → rút thêm các **Screen State** (loading/error/empty) và **AC-UI** để phủ; chưa sẵn sàng → cảnh báo mềm.

---

## Nét đặc thù: 3 cơ chế đáng nhớ

**1. Việc lớn thì chia cho "thợ phụ" (orchestration).** PRD lớn (> 3 UC hoặc > 300 dòng) → session chính thành "sếp", **giao mỗi UC cho một sub-agent** làm song song cho nhanh và đỡ sót. PRD nhỏ thì làm một mình.

**2. System BDD = tổng hợp từ web + app.** Khi làm platform *system*, nó **đọc lại BDD web & app đã có** rồi gộp thành hợp đồng tầng hệ thống. Nếu web và app **mong đợi khác nhau** (vd web muốn A, app muốn B) → dừng ở CHECKPOINT bắt bạn chọn cách hoà giải (gộp chung / tách endpoint / theo header…). Nếu PRD là brownfield (API có sẵn) thì dùng thẳng contract trong PRD, khỏi tổng hợp.

**3. PRD đổi thì bắt kịp (version drift).** Nếu BDD cũ sinh từ PRD v1.0 mà PRD giờ v1.2 → đọc Change Log, hỏi cập nhật phần ảnh hưởng (Y) hay làm lại toàn bộ (F). Không chắc đổi ở đâu → khuyên làm lại toàn bộ cho an toàn.

---

## Luật viết kịch bản (giữ chất lượng)

Có bộ luật R1–R10 (mỗi kịch bản đủ Given/When/Then, một hành vi mỗi kịch bản, dùng ngôn ngữ nghiệp vụ không dùng selector/API, tên kịch bản là *kết quả nghiệp vụ*…) và bộ tuân thủ C.1–C.5 (mỗi thành phần Wireframe/AC/BR đều phải có kịch bản phủ, 0 thuật ngữ cấm…).

Trước khi sinh, nó trình **outline** các kịch bản cho bạn xác nhận (thêm/bớt) → rồi mới viết.

---

## Kết quả & vị trí trong dây chuyền

Ra các file `.feature` + cập nhật **trace `.tsv`** (bảng theo dõi mỗi kịch bản: version, ai implement, trạng thái test…).

```
PRD duyệt → [ generate-bdd ◀ bạn ở đây ] (web → app → system) → review-context(BDD) → generate-tech-docs → …
```

> **Một câu chốt:** `/generate-bdd` biến yêu cầu thành kịch bản kiểm thử nghiệp vụ; chia việc cho thợ phụ khi PRD lớn, tổng hợp System BDD từ web+app (chặn lại khi hai bên mâu thuẫn), và bám version PRD để không lệch.
