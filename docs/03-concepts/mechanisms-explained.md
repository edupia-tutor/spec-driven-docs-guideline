[📚 Docs](../README.md) › [Concepts](README.md) › Cơ Chế (Dễ Hiểu)

# Giải Thích Cơ Chế Framework — Ngôn Ngữ Dễ Hiểu

> Nơi gom các giải thích **cơ chế hoạt động** của framework bằng lời dễ hiểu + ví von đời thường. Bổ sung cho các trang concepts "chính quy" ([pipeline](pipeline.md), [traceability](traceability.md), [architecture](architecture.md)) — trang này ưu tiên *trực giác*, không phải đặc tả.

## Mục lục

- [1. PRD lớn → chia việc cho nhiều "thợ phụ" (orchestration / sub-agent)](#1-prd-lớn--chia-việc-cho-nhiều-thợ-phụ-orchestration--sub-agent)
- [2. AC / BR / Scope ở khác tầng — "4 ngăn" (altitude)](#2-ac--br--scope-ở-khác-tầng--4-ngăn-altitude)

---

## 1. PRD lớn → chia việc cho nhiều "thợ phụ" (orchestration / sub-agent)

**Cơ chế:** Khi PRD **nhỏ**, một mình AI làm hết từ đầu tới cuối. Khi PRD **lớn** (> 3 Use Case hoặc > 300 dòng), AI chuyển sang **chế độ điều phối**: session chính thành "sếp" nhẹ, **spawn mỗi Use Case một sub-agent** ("thợ phụ") có context window riêng để làm song song cho nhanh và đỡ tốn bộ nhớ.

**Ví von:** Việc nhỏ thì một người ôm trọn. Việc lớn thì **sếp chia mỗi thợ một Use Case**.

**Điểm mấu chốt — sếp làm phần chung MỘT LẦN, rồi "photo" đưa thợ:**
- **Sếp làm trước, một lần** (ở session chính, đọc cả PRD): kiểm tra **cổng duyệt** (PRD approved?), **nạp Design Spec** (đã duyệt + còn mới + soi nhanh) → rút ra phần màn hình + AC-UI cần phủ (gọi là `design_coverage`), và chốt **platform** (web/app/system).
- **Khi giao việc**, sếp **kèm sẵn cho mỗi thợ**: platform + `design_coverage` đã rút. Thợ **không** phải tự đi đọc lại design-spec, **không** phải hỏi lại "đã duyệt chưa".
- Mỗi thợ chỉ đọc đúng **1 Use Case** trong PRD + dùng đồ sếp đưa → sinh BDD cho UC đó.

**Vì sao thiết kế vậy:**
- **Rẻ:** thợ khỏi đọc lại cả design-spec (đỡ token), khỏi bắt người dùng bấm Y/N nhiều lần (cổng duyệt hỏi 1 lần ở sếp).
- **Không sót:** phần phủ design (Screen States loading/lỗi/trống + AC-UI) áp **cho cả PRD lớn**, không chỉ PRD nhỏ.

> **Cạm bẫy đã từng có (và cách tránh):** nếu sếp quên "photo" `design_coverage`/platform khi giao việc → thợ làm theo trí nhớ thiếu → BDD của PRD lớn **rớt mất phần design** (dù PRD nhỏ vẫn đúng). Bài học: state nào orchestrator đã phân giải mà bước sau cần → **phải nhét vào payload giao cho sub-agent**, đừng để rơi.

*Chi tiết kỹ thuật:* [pipeline › spawn-agent](pipeline.md#spawn-agentmd--sub-agent-orchestration) · [architecture › sub-agent orchestration](architecture.md#sub-agent-orchestration).

---

## 2. AC / BR / Scope ở khác tầng — "4 ngăn" (altitude)

**Cơ chế:** Nội dung PRD có **4 "ngăn"**, mỗi loại thông tin ở đúng một ngăn:
- **AC** (Acceptance Criteria) = *biên bản nghiệm thu* → "đạt khi nào" (kết quả nhìn/đo được) + trỏ số hiệu BR.
- **BR / BL** (Business Rule / Logic) = *bản vẽ kỹ thuật* → "chạy thế nào" (thử lại mấy lần, timeout, cờ ai giữ, nhánh lỗi).
- **Scope (In/Out)** = *ranh giới lô đất* → "làm gì / không làm gì" (một dòng, đọc lướt hiểu ngay).
- **BUSINESS DEFINITION / business-dictionary** = *giải nghĩa từ*.

**Vì sao dễ hỏng:** `/refine-prd` giống một **tổ 4 người soi bài** (QA/DEV/SA/PO) + một người canh *"đã đủ chưa?"* (completeness-critic). Cả tổ **chỉ biết THÊM chi tiết** — mà câu hỏi mặc định *"AC đủ chi tiết chưa?"* tự nó **kéo chi tiết bản-vẽ-kỹ-thuật vào biên-bản-nghiệm-thu**. Qua nhiều vòng `--resume`, **AC dày lên bằng BR** → hai ngăn hội tụ, tài liệu trùng lặp + AC hết dùng được như checklist. In Scope tụt tầng cùng lý do (bị nhét định nghĩa cơ chế).

> **Gốc rễ:** tổ soi bài tối ưu *"đủ từng section"* nhưng **thiếu lực đối trọng giữ ranh giới vai trò** — không ai kéo "mỗi thứ về đúng ngăn", nên chi tiết cứ trôi xuống, ngăn BR hút hết.

**Cách giữ ranh giới (7 luật, 2 nhóm):**
- **Dọn (hạ nguồn):** AC chỉ ghi outcome + ref BR (cấm cơ chế) · lúc áp fix → cơ chế bỏ vào BR/BL · lính gác bắt AC lỡ chứa cơ chế · Scope chỉ là ranh giới.
- **Sửa gốc (thượng nguồn):** đổi **câu hỏi** của tổ soi bài (hỏi "AC có outcome test được chưa", chi tiết → BR/BL) · thêm **người canh "đừng lộn ngăn"** vào critic (không chỉ "đủ chưa" mà "có lộn tầng không") · **bắt AC ≈ BR** (trùng nội dung → làm mỏng AC).

> **Đừng hiểu nhầm là "cắt bớt":** chi tiết retry/timeout/nhánh lỗi **KHÔNG bị xoá** — nó **di dời** từ AC xuống BR/BL. Vì `/generate-bdd` sinh test theo *mỗi BR → ≥1 scenario*; nhánh lỗi/retry chính là nguồn đẻ test. Bỏ hẳn = mất test.

*Chi tiết áp ở đâu:* lăng kính + completeness-critic (`steps/review-fanout.md`, `commands/refine-prd.tmpl`) · P-check (`commands/review-context.tmpl`) · format AC/Scope (`templates/prd.template.md`, `commands/generate-prd.tmpl`).

---

*Có thêm cơ chế nào được giải thích kiểu dễ hiểu → thêm một mục mới ở đây.*
