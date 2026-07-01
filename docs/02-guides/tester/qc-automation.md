[📚 Docs](../../README.md) › [Guides](../README.md) › [Tester](README.md) › QC Automation

# Hướng Dẫn QC Automation — Spec-Driven Docs

> Đây là **một chương của [guide Tester / QA](README.md)** — QC và Tester là **cùng một role**. Chương này đi sâu vào pipeline QC tự động (`/qc-*`); phần vai trò chung, spec-manifest, `/report-bug`, `/propose-scenario`… ở [README của guide](README.md).

Pipeline QC automation **chính thức** của framework — được port từ agent của team QC (reference repo `ai-automation-qc-base`) vào ngay trong framework. Nó sinh và chạy automation **Playwright/pytest** từ BDD `.feature` chính thức, rồi ghi tín hiệu **`qc_status`** (authoritative) vào trace TSV → Living Docs.

## Mục Lục

- [Hai luồng test: dev self-check vs QC chính thức](#hai-luồng-test-dev-self-check-vs-qc-chính-thức)
- [Pipeline 6 bước](#pipeline-6-bước)
- [Stack — module qc-playwright](#stack--module-qc-playwright)
- [Trace join: qc_status đến Living Docs như thế nào](#trace-join-qc_status-đến-living-docs-như-thế-nào)
- [Entry point](#entry-point)
- [Skills theo layer](#skills-theo-layer)

## Hai Luồng Test: Dev Self-Check vs QC Chính Thức

Pipeline QC này **khác** với developer **self-check** (`/dev-gen-test`, `/dev-run-test`, `/dev-smoke-test`):

| Signal | Owner | Command | Ý nghĩa |
|--------|-------|---------|---------|
| `dev_selftest` | Dev | `/dev-run-test` | smoke check của riêng dev (KHÔNG authoritative) |
| `qc_status` | QC | `/qc-run-test` | kết quả QC automation **chính thức** |

Cả hai hiển thị **cạnh nhau** trong Living Docs; không cái nào ghi đè cái nào. `dev_selftest: pass` chỉ nghĩa "dev đã tự smoke qua"; coverage chính thức nằm ở `qc_status`.

## Pipeline 6 Bước

```
/qc-analyze → /qc-plan → /qc-design-test → /qc-review → /qc-run-test → /qc-report
 (requirement   (risk/plan   (test-case      (review     (gen+run     (report +
  breakdown)     + Q-for-dev)  .Test.md)       gate)       Python →     evidence)
                                                            qc_status)
```

| Command | Vai trò | Output |
|---------|---------|--------|
| `/qc-analyze` | bóc tách spec chính thức thành **1 file phân tích gộp** (requirement + BR/AC + data-flow) + `DOC_GAPS` | `{qc_dir}/{UC-ID}/` → `REQUIREMENT_ANALYSIS.md` + `DOC_GAPS.md` (**2 file**) |
| `/qc-plan` | risk / what-if / questions-for-dev + test plan | `TEST_PLAN.md` |
| `/qc-design-test` | thiết kế test case (Markdown `.Test.md`), tag `@trace.verifies={UC-ID}-SC{N}` | `test-cases/` |
| `/qc-review` | cổng review hai chiều: review test case (sau design) và script (sau run) | findings |
| `/qc-run-test` | sinh + chạy Python pytest-playwright; **ghi `qc_status`** per scenario | `{trace_dir}/{UC-ID}.tsv` |
| `/qc-report` | report pytest-html + Playwright trace + evidence | `reports/<feature>/report.html` |

> **Nơi lưu artifact (`qc_dir`):** mọi output của pipeline (trừ report) nằm trong thư mục QC
> **lộ ra ngoài** `{qc_dir}/{UC-ID}/` — mặc định `docs/{UC-ID}/`, cấu hình qua `paths.qc_dir`
> trong `project-context.yaml`. Đây là **working-doc riêng của QC** trong repo QC, **không ẩn**
> (KHÔNG nằm trong `.agent/`). Spec gốc (PRD/`.feature`/design-spec) KHÔNG nằm đây — chúng đến
> từ **submodule spec của PO** (`spec_source`). `/qc-analyze` chỉ ghi **đúng 2 file**:
> `REQUIREMENT_ANALYSIS.md` (phân tích gộp) + `DOC_GAPS.md`.

## Stack — Module qc-playwright

`/qc-run-test` và `/qc-report` dùng stack module `qc-playwright`
(`.agent/modules/qc-playwright/stack-profile.yaml`): **Python + pytest-playwright + Page
Object** (slim `BasePage`, 3-layer), **Playwright Trace + pytest-html** (KHÔNG Allure). Stack
này độc lập với module implementation của dev (java-spring, react, flutter, …).

Per-layer guides chi tiết nằm trong `skills/qc/<stage>/` (port từ skills của agent QC):
functional / integration / e2e / non-functional / exploratory.

### Test-ID contract — locate element không cần scan

Để QC **không** phải dò/giả lập vị trí element lúc runtime (chậm), FE tech-design có section **§2b Test Selectors**: mỗi element **có action** được gán test-id ổn định (`{uc}-{screen}-{element}-{type}`, vd `ft001-login-submit-btn`). `/generate-code` emit đúng id đó theo attribute platform (web `data-testid` · RN `testID` · Flutter `Key`/`Semantics` · iOS `accessibilityIdentifier`); `/qc-design-test` cite test-id trong step; `/qc-run-test` build Page Object locator **thẳng từ map** (ưu tiên trong locator priority `data-testid → role → …`). Element có action mà chưa có test-id trong §2b → QC **fallback** role/text (chậm hơn) và nên bổ sung vào tech-design. Xem [Trace Schema](../../05-reference/trace-schema.md) và FE tech-design (`/generate-tech-docs`).

> **Component reuse / code có sẵn:** `/generate-tech-docs` + `/generate-code` chỉ gán id cho code **mới**. Với component dùng chung (id ở usage site, component phải *forward* test-id) hoặc màn hình **brownfield** đã viết tay → chạy **`/map-testids {UC-ID}`**: reverse-document id đang có, gán id còn thiếu, patch forwarding + ghi vào `figma-components/{module}.md` (section *Test-ID Forwarding*), điền §2b. Nhờ vậy QC vẫn locate-by-id không cần scan.

## Trace Join: qc_status Đến Living Docs Như Thế Nào

1. `.feature` chính thức định nghĩa scenario là `@trace.scenario={UC-ID}-SC{N}`.
2. `/qc-design-test` ghi SC mà test case thuộc về; `/qc-run-test` tag mỗi pytest test
   `# @trace.verifies={UC-ID}-SC{N}`.
3. `/qc-run-test` ghi `qc_status` (`pass|fail|skip|not_run`) + `qc_run_at` vào trace TSV của
   service (authoritative), rồi refresh panel mirror local.
4. `/validate-traces` (hoặc `/sync`) tổng hợp các TSV thành report Living Docs tại
   `{spec_source}/.living-docs/` — dashboard hiển thị `qc_status` cạnh `dev_selftest`.

## Entry Point

Pipeline QC bắt đầu khi BDD của một UC đã duyệt — `# @trace.status: approved` (sau `/review-context (BDD)` sạch critical + người duyệt đặt). `/qc-analyze` đọc field này và **cảnh báo mềm** nếu chưa approved:

```
/qc-analyze {UC-ID}   →  … → /qc-report {UC-ID}  →  /validate-traces {UC-ID}
```

## Skills Theo Layer

Các skill chi tiết theo từng giai đoạn QC nằm trong `skills/qc/`:

| Stage skill | Vai trò |
|---|---|
| `qa-analyst` | phân tích requirement, sinh `DOC_GAPS` |
| `qa-planner` | test plan, risk, questions-for-dev |
| `qa-designer` | thiết kế test case `.Test.md` theo layer (functional / integration / e2e / non-functional / exploratory) |
| `qa-reviewer` | cổng review test case và script |
| `qa-runner` | sinh + chạy Python pytest-playwright, sinh report (Playwright Trace + pytest-html) |

Mỗi skill **tự chứa** (self-contained) và có version frontmatter để theo dõi khi nâng cấp.

## Khi QC Tìm Thấy Bug / Spec Gap — Đẩy Lên Specs

QC chạy pipeline nhưng **không tự sửa spec/code**. Khi phát hiện vấn đề, đẩy lên spec repo của PO
qua flow feedback có sẵn (`/report-bug` · `/propose-scenario` — PO/Dev nhận khi `/sync`):

| QC phát hiện ở | Loại | Hành động |
|---|---|---|
| `/qc-run-test` FAIL = **product-gap** (impl ≠ spec, lỗi thật) | bug | `/report-bug {UC-ID} {expected vs actual}` |
| `/qc-run-test` FAIL = **script-bug** (sai selector/logic script) | — | QC tự sửa script + chạy lại (**KHÔNG** file bug) |
| `/qc-analyze` `DOC_GAPS` blocker = **spec sai / mơ hồ / mâu thuẫn** (PRD·BDD) | spec defect | `/report-bug {UC-ID}` (BUG_FLOW phân loại PRD/BDD) |
| `/qc-analyze` `DOC_GAPS` = **thiếu coverage** (behavior trong AC nhưng chưa có scenario) | coverage gap | `/propose-scenario {UC-ID}` |
| `DOC_GAPS` = `ASSUMPTION` / `OPEN QUESTION` | câu hỏi | qc-plan → questions-for-dev (xác nhận với PO/Dev, không file bug) |

`/report-bug` ghi `feedback/bug-reports/BUG-xxx.md` + commit/push lên spec repo; `/propose-scenario`
ghi `feedback/bdd-proposals/`. PO/Dev thấy khi `/sync`, rồi BUG_FLOW định tuyến root cause:

```
• Code bug       → Dev /fix-bug                                  (KHÔNG đổi spec)
• BDD sai / thiếu → sửa .feature / promote proposal → /generate-bdd
• PRD mơ hồ / sai → PO sửa PRD → /refine-prd → /review-context → /generate-bdd
        ↓ (nếu spec đổi → regenerate)
  /generate-code lại  →  QC /qc-run-test lại  →  qc_status: fail → pass
```

> QC chỉ **đọc** spec (từ submodule PO) — mọi thay đổi PRD/BDD là việc của PO/Dev. QC đẩy *tín hiệu*
> (bug / proposal / gap), không tự sửa canonical spec. `/qc-report` ở cuối pipeline **in sẵn** danh
> sách lệnh `/report-bug` cho từng product-gap để QC chạy. Chi tiết flow bug: [Operations · Bug Flow](../../04-operations/bug-flow.md).

## Skill Sourcing & Upgrade-Safety

Các skill QC (`qa-analyst` / `qa-designer` / `qa-planner` / `qa-reviewer` / `qa-runner` +
`DOC_GAPS.template.md`) **không bị hardcode** — command nạp chúng từ `paths.qc_skills_dir`:

| Tình huống | `qc_skills_dir` | Hệ quả khi `/update-framework` |
|---|---|---|
| Mặc định (standalone) | `.agent/skills/qc` (bản framework bundle) | **Bị ghi đè** mỗi lần upgrade → đừng sửa trực tiếp ở đây |
| QC sở hữu skill (khuyến nghị) | repo QC / submodule, vd `qc-base/.claude/skills` | **Không bao giờ bị đụng** — upgrade chỉ rewrite `.agent/` |

> `qc_skills_dir` chỉ cần trỏ tới thư mục **chứa các folder `qa-*`**. Cả 2 layout đều hợp lệ:
> framework bundle (`.agent/skills/qc/qa-analyst/…`) và repo QC (`.claude/skills/qa-analyst/…`).

**Cách QC tách skill ra khỏi vòng đời framework:**

```bash
# 1. Đưa repo skill của QC vào project (submodule để pin version):
git submodule add <ai-automation-qc-base-repo-url> qc-base

# 2. Trỏ qc_skills_dir trong .agent/project-context.yaml:
#    paths:
#      qc_skills_dir: "qc-base/.claude/skills"

# 3. Từ giờ: QC hoàn thiện skill trong repo qc-base, bump submodule khi cần.
#    /update-framework ghi đè .agent/ nhưng KHÔNG chạm qc-base/ → skill an toàn.
```

Nhờ vậy **vòng đời skill QC** và **vòng đời framework** tách hẳn nhau: nâng cấp framework
không nuốt mất skill đang hoàn thiện, và QC update skill 1 chỗ (repo của họ) cho mọi project.

## Xem Thêm

- [Guide › Tester](README.md) — vai trò tester, spec-manifest, `/report-bug`, `/propose-scenario`
- [Concepts › Traceability](../../03-concepts/traceability.md) — trace TSV, dev_selftest vs qc_status
- [Reference › Modules](../../05-reference/modules.md) — chi tiết module `qc-playwright`
- [Reference › Commands](../../05-reference/commands.md) — danh mục đầy đủ mọi command
