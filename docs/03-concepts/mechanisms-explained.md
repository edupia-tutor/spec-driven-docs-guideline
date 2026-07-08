[📚 Docs](../README.md) › [Concepts](README.md) › Cơ Chế (Dễ Hiểu)

# Giải Thích Cơ Chế Framework — Ngôn Ngữ Dễ Hiểu

> Nơi gom các giải thích **cơ chế hoạt động** của framework bằng lời dễ hiểu + ví von đời thường. Bổ sung cho các trang concepts "chính quy" ([pipeline](pipeline.md), [traceability](traceability.md), [architecture](architecture.md)) — trang này ưu tiên *trực giác*, không phải đặc tả.

## Mục lục

- [1. PRD lớn → chia việc cho nhiều "thợ phụ" (orchestration / sub-agent)](#1-prd-lớn--chia-việc-cho-nhiều-thợ-phụ-orchestration--sub-agent)
- [2. AC / BR / Scope ở khác tầng — "4 ngăn" (altitude)](#2-ac--br--scope-ở-khác-tầng--4-ngăn-altitude)
- [3. Coding convention đặt ở đâu để update không nuốt mất](#3-coding-convention-đặt-ở-đâu-để-update-không-nuốt-mất)
- [4. Tech-doc "một bản vẽ / PRD" — gộp full-stack, lớn dần, và cách review nó](#4-tech-doc-một-bản-vẽ--prd--gộp-full-stack-lớn-dần-và-cách-review-nó)

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

**Vì sao dễ hỏng:** `/refine-prd` giống một **tổ 3 người soi bài** (DEV/SA/PO) + một người canh *"đã đủ chưa?"* (completeness-critic). Cả tổ **chỉ biết THÊM chi tiết** — mà câu hỏi mặc định *"AC đủ chi tiết chưa?"* tự nó **kéo chi tiết bản-vẽ-kỹ-thuật vào biên-bản-nghiệm-thu**. Qua nhiều vòng `--resume`, **AC dày lên bằng BR** → hai ngăn hội tụ, tài liệu trùng lặp + AC hết dùng được như checklist. In Scope tụt tầng cùng lý do (bị nhét định nghĩa cơ chế).

> **Gốc rễ:** tổ soi bài tối ưu *"đủ từng section"* nhưng **thiếu lực đối trọng giữ ranh giới vai trò** — không ai kéo "mỗi thứ về đúng ngăn", nên chi tiết cứ trôi xuống, ngăn BR hút hết.

**Cách giữ ranh giới (7 luật, 2 nhóm):**
- **Dọn (hạ nguồn):** AC chỉ ghi outcome + ref BR (cấm cơ chế) · lúc áp fix → cơ chế bỏ vào BR/BL · lính gác bắt AC lỡ chứa cơ chế · Scope chỉ là ranh giới.
- **Sửa gốc (thượng nguồn):** đổi **câu hỏi** của tổ soi bài (hỏi "AC có outcome test được chưa", chi tiết → BR/BL) · thêm **người canh "đừng lộn ngăn"** vào critic (không chỉ "đủ chưa" mà "có lộn tầng không") · **bắt AC ≈ BR** (trùng nội dung → làm mỏng AC).

> **Đừng hiểu nhầm là "cắt bớt":** chi tiết retry/timeout/nhánh lỗi **KHÔNG bị xoá** — nó **di dời** từ AC xuống BR/BL. Vì `/generate-bdd` sinh test theo *mỗi BR → ≥1 scenario*; nhánh lỗi/retry chính là nguồn đẻ test. Bỏ hẳn = mất test.

*Chi tiết áp ở đâu:* lăng kính + completeness-critic (`steps/review-fanout.md`, `commands/refine-prd.tmpl`) · P-check (`commands/review-context.tmpl`) · format AC/Scope (`templates/prd.template.md`, `commands/generate-prd.tmpl`).

---

## 3. Coding convention đặt ở đâu để update không nuốt mất

**Cơ chế:** File trong project chia làm **hai loại chủ sở hữu**. `/update-framework` (kéo bản framework mới về) **ghi đè** nhóm "của framework" và **không đụng** nhóm "của bạn":

| Của framework — **bị ghi đè** mỗi lần update | Của bạn — **được giữ** nguyên |
|---|---|
| `.agent/commands · steps · hooks · rules · templates · skills` | `CLAUDE.md` |
| `.agent/modules/{stack}/` (`module.yaml`, `stack-profile.yaml`, snippets) | `.agent/project-context.yaml` |
| `.agent/FRAMEWORK_VERSION`, `.claude/commands/` | `specs/domain-knowledge/`, `.trace/` |

**Ví von:** Framework là **bộ đồ nghề đi thuê** — mỗi lần đổi bộ mới, đồ thuê bị thay sạch, nhưng **đồ riêng của bạn** (ghi chú dán trên tường = `CLAUDE.md`, sổ tay dự án = `project-context.yaml`) thì không ai động.

**Điểm mấu chốt — đặt convention ở chỗ vừa GIỮ vừa ĐƯỢC ĐỌC:**
- **`CLAUDE.md`** là nhà chính: `/generate-code`, `/review-code`, `/generate-tech-docs`, `/fix-bug` nạp trực tiếp `§2 Architecture` (thứ tự layer + rule), `§3 Coding Standards` (naming, wrapper, forbidden), `§5 Error Handling` (exception, HTTP code). Framework sinh sẵn skeleton các §; bạn chỉ điền chi tiết stack (vd Java Spring Boot).
- **`project-context.yaml`** `conventions:` giữ lệnh `build_command` / `test_command` / `run_command` — `/generate-code` chạy đúng lệnh này khi tự-kiểm.
- **Convention dài** (cả trang) → tách file dưới `specs/domain-knowledge/` (cũng được giữ + context-loader nạp), rồi trỏ tới từ `CLAUDE.md` cho gọn.

> **Cạm bẫy hay gặp:** nhét convention riêng vào `.agent/modules/{stack}/` (vì thấy nó "nói về stack của mình"). Sai — module là **default chung của stack do framework phát hành**, nằm trong nhóm bị ghi đè; convention của team sẽ **bay sạch** ở lần update kế. Của-stack (framework) khác của-team (bạn) — để riêng.

*Chi tiết:* danh sách ghi-đè/giữ (`commands/update-framework.tmpl`) · nơi đọc convention (`commands/generate-code.tmpl` §2/§3/§5) · skeleton CLAUDE.md (`commands/setup-ai-first.tmpl`) · `conventions:` (`templates/project-context.yaml`).

---

## 4. Tech-doc "một bản vẽ / PRD" — gộp full-stack, lớn dần, và cách review nó

**Cơ chế:** Một PRD chỉ có **một** technical design document — `{TICKET-ID}-tech-design.md`, nằm trong `specs/{domain}/{prd-slug}/tech-docs/`, cạnh `bdd/` và `design-spec/`. File này **gộp full-stack**: backend (API §4.1–§4.4, data model §3, DB) *và* client (§4.5 component/state/Figma/test-id cho mỗi platform), nối với nhau bằng **sequence diagram xuyên tầng** ở §5 (client → service → API → external → DB). Một dev đọc **một** file là hiểu và code được cả tính năng.

**Ví von:** không phải nhiều bản vẽ rời (mỗi phòng một tờ) mà **một bản vẽ tổng mặt bằng** cho cả căn nhà — móng, điện, nước, nội thất trên cùng một khổ giấy, có mũi tên chỉ chúng nối nhau ra sao.

**Input là BDD tech lead trỏ, KHÔNG phải cả PRD:** `/generate-tech-docs` nhận **1..n file BDD** người dùng chỉ định (batch, cảnh báo nếu > 5 để khỏi phình context) — trỏ System BDD → vẽ §4 API; trỏ Web/App BDD → append §4.5 client. Chạy nhiều lần, **doc lớn dần** (append), không đè phần cũ. **§10 UC Coverage** là "sổ điểm danh" để lệnh biết UC nào đã vẽ, UC nào còn thiếu.

**Vì sao gộp thay vì tách BE/FE per-UC:** một tính năng vốn liền mạch (bấm nút ở web → gọi API → ghi DB); tách ra nhiều file khiến dev phải ghép hình. Gộp lại: §4.5.4 (client gọi API) map thẳng lên §4.1 (endpoint) trong **cùng** tài liệu — không lệch, không đi tìm file anh em.

**Review chạy trên doc gộp thế nào (`/review-tech-docs`):**
- **Phạm vi = cả PRD, không phải 1 UC:** đọc `@trace.ucs` (danh sách UC) để biết doc phủ những câu nào, ra **một** file findings `{TICKET-ID}-tech-review-findings.yaml` — nhưng **mỗi finding gắn `uc_id`** để biết lỗi thuộc câu nào (như phiếu chấm cho cả xấp bài, mỗi lỗi ghi rõ "câu số mấy").
- **T3 truy vết BDD 2 chiều** với **mọi** BDD của PRD (system/web/app): doc có behavior nào không có scenario? scenario nào chưa được vẽ (§5/§10)? — các file BDD này thuộc *chính PRD* nên phạm vi có giới hạn.
- **Duyệt xong → đóng dấu revision cho mọi UC:** doc bump `@trace.revision`; review ghi `tech_doc_revision = revision mới` vào TSV của **từng** UC trong `@trace.ucs` (sổ trace vẫn per-UC). Chừa cột `fe_tech_doc_revision` cho `/generate-code --phase=integration` ghi (lúc FE wire API thật theo §4.5.4) — **2 cột, cùng đọc 1 file, ghi ở 2 thời điểm**.

**Điểm mấu chốt — check trùng endpoint liên-PRD kiểu "tra sổ địa chỉ, không vào từng nhà" (T4):** vì doc rất dài, muốn bắt 2 PRD vô tình định nghĩa cùng endpoint khác shape, review **không** đọc trọn tech-doc của mọi PRD khác (nổ context). Thay vào đó: trích danh sách endpoint (§4.1) của doc đang review → `grep` các path đó trong tech-doc PRD khác (chỉ đọc dòng match) → **chỉ khi trùng path** mới nạp đúng đoạn §4 của doc kia để so. Bình thường tốn vài dòng grep; chỉ "trả tiền" khi thật sự va chạm.

> **Cạm bẫy đã tránh:** ban đầu định cho T4 nạp full mọi tech-doc cùng domain để so — mỗi doc 500–800 dòng × N PRD = context nổ. Bài học: khi cần đối chiếu chéo nhiều tài liệu lớn, **lọc bằng chỉ mục rẻ (grep path) trước, nạp full chỉ khi có hit** — đừng load tất cả rồi mới so.

> **Cạm bẫy #2 — trùng số SC khi gộp platform (số căn hộ giữa hai toà):** mã scenario `{UC}-SC{N}` **chỉ độc nhất trong (UC × platform)** — `system UC1-SC1` và `web UC1-SC1` là **hai scenario khác nhau**. Gộp cả 3 platform vào 1 doc rồi liệt kê SC phẳng → nhập nhằng (như Toà A căn 101 vs Toà B căn 101, gộp danh bạ chỉ ghi "101"). **Cách tránh — giữ lane theo platform:** §5 chia `5.A system / 5.B web / 5.C app`, mỗi SC ghi kèm platform (`web · UC1-SC1`); §10 có cột **Platform** (mỗi `(platform, SC)` một dòng); review T3 match SC trong đúng lane, không so chéo. *(Cùng gốc collision này từng nằm ở **tầng trace**: sổ TSV per-UC key thuần `sc_id` → scenario các platform **đè/xoá nhau** (gen web xoá luôn scenario system). **Đã fix bằng cách tách sổ theo platform** — `{UC-ID}-{platform}.tsv`, mỗi platform một sổ (system/web/app). Hết đè/xoá; Living Docs gắn field `platform` mỗi row để hiển thị coverage tách bạch. Chi tiết: [Trace Schema](../05-reference/trace-schema.md).)*

**Điều hướng doc gộp (mục lục + độ hạt):** vì doc là cấp PRD nhưng consumer (`generate-code`/`map-testids`/`qc`) xử **một UC**, **§10 UC Coverage là mục lục**: tra UC → scenario → section/§5-lane của nó (endpoint liên quan = §4.1 mà §5-lane của UC gọi tới). §4.5 **nhóm theo platform** (một nhóm `§4.5 — web`, một `§4.5 — app`), mỗi màn/UC là sub-block; §4.5.6 là **một bảng chung/platform** với cột "Serves SC (UC·SC)" để lọc theo UC. *Ví von: bản vẽ cả toà nhà phải có mục lục "tầng ở trang mấy"; nội thất gom theo tầng, mỗi phòng một trang.*

**Cách `/generate-code` tiêu thụ doc gộp:** code sinh cho **1 UC** nhưng mở **cùng** file gộp rồi lật đúng phần UC đó — không còn đi tìm file per-UC/per-platform:
- **Khuôn mock (`--phase=ui`):** shape request/response lấy từ **§4.1/§4.2/§4.3** (API contract) để mock giống API thật, đỡ nắn lại lúc ráp. Chưa có §4 → suy từ System BDD + cảnh báo.
- **Đấu nối thật (`--phase=integration`):** map client method → endpoint theo **§4.5.4** (sơ đồ đấu nối của platform), endpoint/shape tra §4.1–§4.3 **cùng doc**. Chưa có §4.5.4 → map thẳng từ §4.1 + warn.
- **Test-id:** emit đúng id trong **§4.5.6** (thay §2b cũ) để QC Page Object khớp ngay. (`/map-testids` backfill §4.5.6 cho component tái dùng/brownfield; `/qc-design-test` + `/qc-run-test` đọc §4.5.6 để dựng locator; `/validate-traces` so `@trace.revision` của doc gộp với 2 cột để bắt drift.)
- **Đóng dấu bản vẽ:** ghi `tech_doc_revision` (lúc gen BE) / `fe_tech_doc_revision` (lúc FE integration) = `@trace.revision` của doc → `/validate-traces` bắt drift khi code lắp theo bản cũ.

*Ví von ba mục: §4 = **kích thước ô cửa** (đóng khung mock cho khít) · §4.5.4 = **sơ đồ đấu dây** (nối API thật) · §4.5.6 = **bảng số phòng** (QC tìm element).*

**Quy tắc nhận dạng & vài edge (rút ra khi soi BDD thật):**
- **Định danh đọc từ header, không cắt tên file:** `{TICKET-ID}` ← `@trace.prd`, `{UC-ID}` ← `@trace.id` (đều slug-free). Vì TICKET-ID có gạch nối (`FEAT-01-2`) và **slug/title một UC khác nhau theo platform** (UC3: system `sua-cau-bo-qua` vs web `sua-cau-quay-lai-bo-qua`) — cắt chuỗi dễ sai. Gom UC theo `@trace.id` → system/web/app cùng id = **một** UC; title lấy canonical từ **PRD**.
- **`@trace.bdd_version` là map theo platform** (`system=1.5, web=1.9`) chứ không một số — mỗi feature giữ version riêng, gộp phẳng sẽ che mất platform nào còn cũ.
- **PRD client-only (không có system BDD):** đừng bịa BE contract — §4.1 chỉ ghi endpoint client **tiêu thụ** (external/team khác/existing), hoặc "N/A — client-only". Đối xứng với backend-only (bỏ §4.5).

> ✅ **Trạng thái migrate:** **toàn bộ command** đã theo mô hình doc gộp — `/generate-tech-docs`, `/review-tech-docs`, `/generate-code`, `/validate-traces`, `/map-testids`, `/qc-design-test`, `/qc-run-test`, `/generate-bdd`. Còn lại: một số **trang guide/reference** (getting-started, developer/tester guides, command reference, sync-and-update) vẫn mô tả layout per-UC cũ — sẽ quét đồng bộ sau (không ảnh hưởng hành vi lệnh).

*Chi tiết:* sinh doc (`commands/generate-tech-docs.tmpl` + `templates/tech-design.template.md`) · review (`commands/review-tech-docs.tmpl` — T3/T4/T7 + Resume TSV) · vị trí trong dây chuyền ([pipeline › Phase 4](pipeline.md)) · chuẩn bị input ([Checklist Input Tech-Docs](../02-guides/tech-docs-input-checklist.md)).

---

*Có thêm cơ chế nào được giải thích kiểu dễ hiểu → thêm một mục mới ở đây.*
