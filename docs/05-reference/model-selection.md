# Chọn model cho từng step (Opus vs Sonnet)

> Bảng khuyến nghị model chạy cho mỗi command trong pipeline. Mục tiêu: **đặt Opus đúng chỗ đắt giá** (nơi một quyết định sai làm hỏng code/spec thật) và **để Sonnet gánh phần cơ học** (điền template, chấm luật, chạy lệnh, tổng hợp). Không phải "to hơn cho chắc" — chọn theo bản chất công việc.

## Nguyên tắc chọn

**Dùng Opus khi step có BẤT KỲ dấu hiệu nào dưới đây:**

1. **Ghi/sửa artifact khó đảo** — code thật hoặc spec có thẩm quyền (PRD, tech-doc). Sai là mất công truy vết/khôi phục.
2. **Phải hoà giải với bối cảnh có sẵn** — không clobber code UC khác, không đẻ package trùng, ghi **superset**. Cần giữ trạng thái hiện có trong đầu rồi suy giao phần.
3. **Tổng hợp / khử mơ hồ** — biến input mơ hồ thành quyết định đã chốt (discovery → PRD, PRD → tech design).
4. **Rủi ro bịa** — phải phân biệt "suy được từ nguồn" vs "khai [GAP], KHÔNG bịa". Đây là chỗ Sonnet hay tự chế cho liền mạch.
5. **Truy nguyên nhân gốc** — debug, fix-bug: đọc triệu chứng → lần ra nguyên nhân thật.

**Sonnet là đủ khi step chủ yếu là:**

- Điền template theo ánh xạ nguồn→đích rõ ràng.
- Chấm luật / checklist (trích ID vi phạm, tính status theo công thức).
- Đọc-thuần: report, tổng hợp, bookkeeping trace/TSV.
- Chạy lệnh và thuật lại output.

> **Bằng chứng thực nghiệm:** ba bug nặng nhất của project này (clobber code UC trước, package fragmentation, hardcode) đều xảy ra ở **generate-code** khi chạy model yếu — chúng là lỗi *suy luận về bối cảnh*, không phải lỗi cú pháp. Guard cơ học (Guard sau-ghi, `_seams.tsv`, Self-Review Gate) giảm rủi ro nhưng vẫn cần model đủ sức **chủ động chấp hành**, không đọc cho có.

## Bảng tra nhanh

### 🔴 Opus — bước quyết định, ghi artifact khó đảo

| Command | Vì sao Opus |
|---|---|
| `generate-code` | Ghi code thật; phải EXTEND superset (không clobber UC khác), quyết định nối hàng thật vs đẻ stub (seam), khớp package by-layer, không hardcode. Ca đắt giá nhất. |
| `generate-tech-docs` | Tech design gộp full-stack; rủi ro bịa/thiếu cao (retro đã flag). Self-Review Gate + GAP Register chỉ hiệu quả khi model đủ sức tự bác bỏ mình. |
| `fix-bug` | Sửa code thật theo nguyên nhân gốc; cùng rủi ro clobber như generate-code. |
| `debug` | Truy nguyên nhân gốc từ triệu chứng — suy luận nhiều tầng. |
| `generate-prd` | Tổng hợp discovery → PRD; khử mơ hồ, chốt scope/business rules mà hạ nguồn bám theo. |
| `refine-prd` | Phê bình 3-lens; giá trị nằm ở chất lượng chỉ trích, không phải điền chữ. |
| `review-tech-docs` | T1–T9 gồm reconciliation + gap-honesty; phải bắt được chỗ tech-doc bịa/thiếu. |
| `define-product` | Đặt nền product-definition — sai lệch lan xuống toàn bộ pipeline. Ít chạy nhưng đòn bẩy cao. |

### 🟡 Tùy độ phức tạp — mặc định Sonnet, nâng Opus khi domain khó

| Command | Mặc định | Nâng Opus khi |
|---|---|---|
| `generate-design-spec` | Sonnet | Luồng UI/UX nhiều nhánh, ràng buộc trạng thái phức tạp. |
| `generate-bdd` | Sonnet | Domain nhiều edge/nhánh lỗi khó suy từ PRD. |
| `review-context` | Sonnet | Cần đánh giá sâu độ phủ scenario, không chỉ liệt kê. |
| `review-code` | Sonnet | Review logic/nghiệp vụ (không chỉ chấm CV/BR style). |
| `propose-scenario` | Sonnet | Sinh scenario cho nghiệp vụ nhiều biến thể. |
| `qc-analyze` | Sonnet | Phân tích rủi ro test sâu, không chỉ phân loại. |
| `qc-review` | Sonnet | Đánh giá chất lượng test-case cần suy luận. |

### 🟢 Sonnet là đủ — cơ học, chấm luật, đọc-thuần

| Command | Bản chất |
|---|---|
| `dev-gen-test` | Sinh test từ BDD — ánh xạ scenario→test khá thẳng. |
| `dev-run-test` · `dev-smoke-test` · `qc-run-test` | Chạy test, thuật output. |
| `qc-plan` · `qc-design-test` · `qc-report` | Điền plan/case/report theo khuôn. |
| `map-testids` | Ánh xạ test-id. |
| `validate-traces` | Tính status theo công thức + Seam Audit theo luật rõ. |
| `generate-spec-manifest` | Tổng hợp manifest cơ học. |
| `review-code` (style) | Trích ID vi phạm CV/BR — chấm luật. |
| `sync` · `update-framework` · `setup-ai-first` | Thao tác file/scaffold theo quy trình. |
| `report-bug` · `learn` | Điền báo cáo / tra cứu. |

> **Truly trivial** (`report-bug`, `generate-spec-manifest`, `sync`): Haiku cũng chạy được nếu cần tiết kiệm — nhưng lợi ích so với Sonnet không đáng kể.

## Ngoại lệ đáng nhớ

- **generate-code trên greenfield cô lập → Sonnet ổn.** Khi UC đầu tiên của module trống: không sibling code, không seam ra ngoài, không EXTEND file có sẵn → không có gì để clobber. Hễ **module đã có code UC khác** (đúng tình huống segment-service) → về lại Opus.
- **review-* nói chung**: nếu chỉ chấm checklist cơ học thì Sonnet đủ; nếu là "cổng chặn merge" mà bỏ sót gây hậu quả thật → Opus.

## Tóm tắt một dòng

**Ghi code/spec khó đảo hoặc phải hoà giải bối cảnh → Opus. Điền khuôn, chấm luật, chạy lệnh, tổng hợp → Sonnet.**
