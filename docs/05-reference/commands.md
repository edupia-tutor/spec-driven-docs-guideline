[📚 Docs](../README.md) › [Reference](README.md) › Command Reference

# Command Reference

> Mọi slash command của framework, gom theo phase. Mỗi command kèm **Cách dùng (ví dụ cụ thể)**, **Output**, và **When to use**. Phần cuối mô tả **Command Internals** — kiến trúc step file dùng chung bởi mọi command.
>
> Ví dụ dùng chung một feature xuyên suốt: domain `auth`, ticket `FEAT-01`, use case `FEAT-01-UC1` (đăng nhập). Thay bằng file/UC-ID của bạn.

---

## Mục lục

- [Setup & Maintenance](#setup--maintenance)
- [Phase 1 — Discovery](#phase-1--discovery)
- [Phase 2 — PRD](#phase-2--prd)
- [Phase 2b — Design Spec (FE/App)](#phase-2b--design-spec-feapp)
- [Phase 3 — BDD Spec](#phase-3--bdd-spec)
- [Phase 4 — Tech Design](#phase-4--tech-design)
- [Phase 5 — Code](#phase-5--code)
- [Phase 6 — Dev Self-Check](#phase-6--dev-self-check)
- [Phase 6b — QC Automation (official QC suite)](#phase-6b--qc-automation-official-qc-suite)
- [Phase 7 — Traceability Audit](#phase-7--traceability-audit)
- [Phase 8 — Tester / QC Feedback](#phase-8--tester--qc-feedback)
- [Debugging & Cross-cutting](#debugging--cross-cutting)
- [Command Internals — Step Architecture](#command-internals--step-architecture)

> **Lưu ý số phase:** README dùng cả "Phase 5b / Phase 6b" (sơ đồ workflow) và "Phase 6b" (bảng tổng quan) cho QC Automation. Ở đây dùng nhãn **Phase 6b** cho QC suite để khớp bảng "Phase overview". Dù số khác nhau, ý nghĩa không đổi: QC suite là pipeline `/qc-*` riêng biệt với dev self-check `/dev-*`.

---

## Setup & Maintenance

| Command | Cách dùng (ví dụ) | Output | When to use |
|---------|-------------------|--------|-------------|
| `/setup-ai-first` | `/setup-ai-first` | Project structure + config files | First-time project setup |
| `/sync` | `/sync` | Git pull + submodule update + Living Docs sync | **Daily driver** — đồng bộ *nội dung* project (code/specs trong submodule) |
| `/update-framework` | `/update-framework` | Refreshed `.agent/` command files from npm | **Occasionally** — nâng cấp *bản thân framework* khi có version mới |
| `/sync-figma-components {module}` | `/sync-figma-components react` | Updated `figma-components/{module}.md` | After Figma design system changes |
| `/sync-figma-tokens {url?}` | `/sync-figma-tokens` *(hoặc kèm URL Figma)* | Updated `figma-tokens.md` | After design token changes |

> **`/sync` vs `/update-framework` là hai thứ khác nhau:**
> - `/sync` cập nhật **nội dung project** — pull code/specs mới từ git remote + refresh Living Docs. Nguồn = git remotes của bạn. Chạy thường xuyên.
> - `/update-framework` cập nhật **framework tooling** — refresh `.agent/commands/`, `steps/`, `modules/` lên version mới nhất. Nguồn = npm registry. Chạy hiếm. Không bao giờ đụng `project-context.yaml`, `CLAUDE.md`, domain-knowledge, hay `.trace/`.

---

## Phase 1 — Discovery

| Command | Cách dùng (ví dụ) | Output | When to use |
|---------|-------------------|--------|-------------|
| `/define-product` | `/define-product` | `specs/product-definition/{TICKET-ID}-{slug}.md` | Starting any new feature. AI dẫn PO qua 7 phase Q&A có cấu trúc — không cần spec viết sẵn. |

---

## Phase 2 — PRD

| Command | Cách dùng (ví dụ) | Output | When to use |
|---------|-------------------|--------|-------------|
| `/generate-prd` | `/generate-prd specs/product-definition/FEAT-01-login.md` | `specs/{domain}/{prd-slug}/{TICKET-ID}-{prd-slug}.md` | After define-product. Thêm `API Source: existing` + section "Existing API Contract" cho feature brownfield. |
| `/refine-prd` | `/refine-prd specs/auth/login/{TICKET-ID}-login.md` | `.agent/review/{prd-slug}-findings.yaml` | After generate-prd — fan-out 3 review lens (DEV/SA/PO) + completeness-critic loop, mở Review Board. |
| `/refine-prd --resume` | `/refine-prd --resume specs/auth/login/{TICKET-ID}-login.md` | Applied findings + bumped version | Sau khi review trong Review Board (nút ⚡ Apply tự chạy lệnh này). |
| `/review-context` (PRD) | `/review-context specs/auth/login/{TICKET-ID}-login.md` | `.agent/review/{prd-slug}-review-context-findings.yaml` | Quality gate trước Phase 3. Checks: routing P0 (umbrella), banned terms (P1), ambiguity (P2), conflicts (P3), completeness (P4), custom (P5). 0 critical → PO đặt `Status: approved`. `--fix`/`--resume` reset Status→draft khi sửa PRD. |
| `/review-context --fix` | `/review-context --fix specs/auth/login/{TICKET-ID}-login.md` | Applies all auto-fixable findings immediately | Dev quick-fix, không cần Review Board. |
| `/review-context --resume` | `/review-context --resume specs/auth/login/{TICKET-ID}-login.md` | Applies accepted findings + bump version | PO/SA review — human quyết từng finding. |

> **Change Log gọn (rolling-window):** PRD chỉ giữ **5 version gần nhất** trong `# Change Log` (bảng phẳng 1 dòng/version); lịch sử cũ hơn được `/refine-prd` & `/review-context` tự dồn sang `changelog/{TICKET-ID}-{prd-slug}.changelog.md` (thư mục con của feature-package). Nhờ vậy PRD **không phình** theo các vòng refine, Dev/QC khỏi nạp lịch sử thừa vào context. `/generate-bdd` đọc 5 row gần để bắt drift; nếu BDD cũ hơn cửa sổ này → khuyến nghị gen lại toàn bộ (F).

---

## Phase 2b — Design Spec (FE/App)

| Command | Cách dùng (ví dụ) | Output | When to use |
|---------|-------------------|--------|-------------|
| `/generate-design-spec` | `/generate-design-spec specs/auth/login/{TICKET-ID}-login.md` | `specs/{domain}/{prd-slug}/design-spec/{TICKET-ID}-design-spec-{platform}-{slug}.md` | FE/App: sau khi PRD approved (guard mềm), trước BDD. Ghi `Built from PRD: vX` (phát hiện lỗi thời); re-run có Version Check theo PRD. PO cấp **node-level Figma frame link** (`?node-id=`) cho mỗi screen; AI fetch từng frame qua Figma MCP. Screen thiếu link → ❌ Missing, Status giữ `draft`. **Self-Review Gate** tự rà trước khi ghi; `/generate-bdd` chỉ **cảnh báo mềm** nếu design-spec chưa approved. |

> BE teams bỏ qua Phase 2b — đọc PRD trực tiếp rồi `/generate-bdd`.

---

## Phase 3 — BDD Spec

| Command | Cách dùng (ví dụ) | Output | When to use |
|---------|-------------------|--------|-------------|
| `/generate-bdd` | `/generate-bdd specs/auth/login/{TICKET-ID}-login.md` | `specs/{domain}/{prd-slug}/bdd/{UC-ID}.feature` (multi-service: `{prd-slug}/bdd/{service}/{UC-ID}.feature`) | After PRD approved (+ Design Spec sign-off cho FE/App). Đọc Service/Module từ PRD metadata, áp platform vocabulary, hiện SC outline chờ confirm. **Thứ tự outside-in: web → app → system**; `system` **tổng hợp từ web+app BDD** (web&app lệch contract → CHECKPOINT chọn union/platform-hint/separate-endpoints, ghi `@system.resolution:`; chỉ-BE → từ PRD). PRD lớn (>3 UC hoặc >300 dòng) → orchestration mode (1 sub-agent / UC). FE/App: nuốt AC-UI + Screen States từ design-spec (gate approved+fresh+sanity). Proposal tester: chỉ nạp `Status: accepted` rồi lưu trữ. |
| `/review-context` (BDD) | `/review-context specs/auth/login/bdd/system/FEAT-01-UC1-login.feature` | `.agent/review/{uc-id}-review-bdd-findings.yaml` | Quality gate trước Phase 4. Checks: PRD coverage (mỗi AC + BR → ≥1 SC), Gherkin R1–R10, compliance C1–C5. 0 critical → đặt `@trace.status: approved`. |
| `/review-context --fix` | `/review-context --fix specs/auth/login/bdd/web/FEAT-01-UC1-login.feature` | Auto-fix terminology, metadata, coverage matrix + reset `@trace.status` & `uc_status` → draft | Dev quick-fix. |
| `/review-context --resume` | `/review-context --resume specs/auth/login/bdd/system/FEAT-01-UC1-login.feature` | Applies accepted findings + bump bdd_version + reset `@trace.status` & `uc_status` → draft | SA review. |

---

## Phase 4 — Tech Design

| Command | Cách dùng (ví dụ) | Output | When to use |
|---------|-------------------|--------|-------------|
| `/generate-tech-docs` | `/generate-tech-docs specs/auth/login/bdd/system/FEAT-01-UC1-login.feature` | **1 doc full-stack/PRD**: `specs/{domain}/{prd-slug}/tech-docs/{TICKET-ID}-tech-design.md` | After BDD approved. **Input = 1..n file BDD tech lead trỏ** (batch, cảnh báo >5), gộp vào doc chung của PRD (append qua nhiều lần chạy). Doc chứa API contract (§4.1–§4.4) + client design (§4.5 component/state/API-map/**§4.5.6 Test Selectors** per platform) + §5 sequence xuyên tầng. Brownfield (`@trace.api_source: existing`) → reverse-document phần API. |
| `/map-testids {UC-ID}` | `/map-testids FEAT-01-UC1` | Cập nhật §4.5.6 Test Selectors trong tech-doc gộp + patch forwarding/usage-site + ghi `figma-components/{module}.md` | FE/App, cho component **reuse** / code **có sẵn** (brownfield): reverse-document id đang có, gán id còn thiếu, đảm bảo component dùng chung forward được test-id. Bổ trợ cho `/generate-tech-docs` (chỉ gán id cho code mới). |
| `/review-tech-docs` | `/review-tech-docs specs/auth/login/tech-docs/FEAT-01-tech-design.md` | `.agent/review/{TICKET-ID}-tech-review-findings.yaml` | After generate-tech-docs — mở Review Board. Review cả doc/PRD (findings gom theo UC). |
| `/review-tech-docs --resume` | `/review-tech-docs --resume specs/auth/login/tech-docs/FEAT-01-tech-design.md` | Applies accepted findings + bump revision | Sau khi review trong Review Board. |

---

## Phase 5 — Code

| Command | Cách dùng (ví dụ) | Output | When to use |
|---------|-------------------|--------|-------------|
| `/generate-code` | `/generate-code specs/auth/login/bdd/system/FEAT-01-UC1-login.feature` | `src/...` (tags `@trace.implements`) | After tech-design approved. Guard mềm: BDD `@trace.status` approved; FE/App design-spec approved+fresh+sanity. Default = full impl/BE. |
| `/generate-code --phase=ui` | `/generate-code --phase=ui specs/auth/login/bdd/web/FEAT-01-UC1-login.feature` | UI + mock API adapter | FE: BE chưa sẵn sàng — sinh UI + mock adapter. Mock **shape**: BE contract nếu có (chuẩn) → else System BDD + warn; fixture values luôn từ System BDD. Tester test FE ngay được. |
| `/generate-code --phase=integration` | `/generate-code --phase=integration specs/auth/login/bdd/web/FEAT-01-UC1-login.feature` | Wires real API | FE: sau khi T7 sign-off gate approved — thay mock adapter bằng real API calls. |
| `/review-code` | `/review-code FEAT-01-UC1` | Review report: Critical / Major / Minor | After generate-code. Check: mỗi scenario có endpoint, trace tags, layer rules, error handling khớp CLAUDE.md §5. Fix CRITICAL + MAJOR trước khi merge. |

---

## Phase 6 — Dev Self-Check

> Đây là **developer self-check / smoke** — dev tự kiểm tra code mình vừa sinh ra. **Không phải** official QC suite (đó là Phase 6b `/qc-*`). Signal = `dev_selftest`, tách biệt với `qc_status`.

| Command | Cách dùng (ví dụ) | Output | When to use |
|---------|-------------------|--------|-------------|
| `/dev-gen-test` | `/dev-gen-test specs/auth/login/bdd/system/FEAT-01-UC1-login.feature` | `src/test/...` (tags `@trace.verifies`) | After generate-code. Tự chọn test framework đúng platform. |
| `/dev-run-test` | `/dev-run-test FEAT-01-UC1` | Self-check report — ghi `dev_selftest` (pass/fail/not_run) + `dev_selftest_at` vào trace TSV | After dev-gen-test. |
| `/dev-smoke-test` | `/dev-smoke-test FEAT-01-UC1` | Live endpoint check | Optional — sau deploy. |

---

## Phase 6b — QC Automation (official QC suite)

> Pipeline `/qc-*` **là** official QC suite. Chạy **theo thứ tự** sau khi BDD `@trace.status: approved` (review-context BDD sạch + người duyệt; `/qc-analyze` cảnh báo mềm nếu chưa). Tách biệt hoàn toàn với dev self-check: QC sở hữu `qc_status`, dev sở hữu `dev_selftest`. `/qc-run-test` & `/qc-report` dùng module `qc-playwright`, độc lập với dev implementation module. Xem [../02-guides/tester/qc-automation.md](../02-guides/tester/qc-automation.md).

| Command | Cách dùng (ví dụ) | Output | When to use |
|---------|-------------------|--------|-------------|
| `/qc-analyze` | `/qc-analyze FEAT-01-UC1` | **2 file** trong `{qc_dir}/{UC-ID}/` (mặc định `docs/`): `REQUIREMENT_ANALYSIS.md` (gộp requirement + BR + data-flow + AC) + `DOC_GAPS.md` | After BDD `@trace.status: approved` (guard mềm) — đầu pipeline QC. |
| `/qc-plan` | `/qc-plan FEAT-01-UC1` | `{qc_dir}/{UC-ID}/TEST_PLAN.md` (scope, layers, priorities, questions-for-dev) | After `/qc-analyze`. |
| `/qc-design-test` | `/qc-design-test FEAT-01-UC1` | `{qc_dir}/{UC-ID}/test-cases/*.Test.md` mapped to `{UC-ID}-SC{N}` | After `/qc-plan`. |
| `/qc-review` | `/qc-review FEAT-01-UC1` | Verdict APPROVED / NEEDS_FIX + findings (**inline — không sinh file**) | After `/qc-design-test` (test-case) hoặc `/qc-run-test` (script). |
| `/qc-run-test` | `/qc-run-test FEAT-01-UC1` | Script Python (`pages/`, `tests/`) + ghi `qc_status` (pass/fail/skip/not_run) + `qc_run_at` vào trace TSV, keyed by `@trace.verifies={UC-ID}-SC{N}` | After `/qc-review`. |
| `/qc-report` | `/qc-report FEAT-01-UC1` | QC report `reports/<feature>/report.html` (Playwright Trace + pytest-html) | After `/qc-run-test`. |

---

## Phase 7 — Traceability Audit

| Command | Cách dùng (ví dụ) | Output | When to use |
|---------|-------------------|--------|-------------|
| `/validate-traces {domain}` | `/validate-traces auth` *(hoặc `/validate-traces FEAT-01-UC1`)* | Coverage matrix + drift report + `trace-report.json` | Anytime — verify traceability. Đọc mọi `.trace/{domain}/{prd-slug}/{UC-ID}-{platform}.tsv` rồi tính status OK / DRIFT / GAP / UNTRACKED. |

---

## Phase 8 — Tester / QC Feedback

> Cả hai đều **read-only trên canonical specs/code**. Chúng chỉ ghi vào `feedback/` của shared spec repo và **commit + push** → PO/Dev thấy qua `/sync`. Tester/QC không bao giờ sửa `.feature` trực tiếp. QC cũng dùng hai lệnh này: `/qc-run-test` FAIL = product-gap và `/qc-analyze` DOC_GAPS blocker đều route sang `/report-bug` hoặc `/propose-scenario` (xem [../02-guides/tester/qc-automation.md](../02-guides/tester/qc-automation.md)).

| Command | Cách dùng (ví dụ) | Output | When to use |
|---------|-------------------|--------|-------------|
| `/report-bug {UC-ID} {desc}` | `/report-bug FEAT-01-UC1 "nhập sai mật khẩu vẫn đăng nhập được"` | `{spec_repo}/feedback/bug-reports/{BUG-ID}.md` + spec context + layer classification | Tester hoặc QC tìm thấy bug (gồm QC product-gap). Phân loại layer (Code/BDD/PRD/Design/Env) để route. |
| `/propose-scenario {UC-ID} {desc}` | `/propose-scenario FEAT-01-UC1 "khoá tài khoản sau 5 lần sai mật khẩu"` | `{spec_repo}/feedback/bdd-proposals/{UC-ID}-*.md` draft Gherkin (mang `Status: proposed`) | Tester hoặc QC tìm thấy edge case BDD chưa cover. Map vào PRD AC sẵn có, hoặc emit PRD change request nếu hành vi thực sự mới. Vòng đời: `proposed` → PO/Dev đặt `accepted` → `/generate-bdd` nạp rồi đặt `incorporated` + lưu trữ. |

---

## Debugging & Cross-cutting

| Command | Cách dùng (ví dụ) | Output | When to use |
|---------|-------------------|--------|-------------|
| `/fix-bug {BUG-ID\|ticket\|desc}` | `/fix-bug BUG-012` *(hoặc ticket / mô tả lỗi)* | Code fix trên branch + regression test; nếu nhận `{BUG-ID}` → set report State `🟢 Open` → `🟡 Fixed` (QC re-verify pass mới → `🟢 Closed`) | Khi tìm thấy bug/ticket, hoặc route một filed bug report. |
| `/debug` | `/debug "NullPointerException ở LoginService"` | Debug session | Deep debugging. |
| `/learn {text}` | `/learn "AI hay dùng var, phải dùng const"` | Appends guardrail vào project lessons file | Khi AI lặp lại lỗi bạn muốn chặn. |
| `/generate-spec-manifest` | `/generate-spec-manifest` | `spec-manifest.yaml` | Sinh manifest làm entry point cho external/tester agents. |

> **Project Lessons:** lessons lưu ở `paths.lessons_file` (default `specs/domain-knowledge/lessons-learned.md`; per-service `.agent/project-lessons.md` trong umbrella mode). Context-loader **Step 6.7** load mọi lesson đầu **mỗi** command như hard constraint. Đây là *project memory*, không phải fine-tuning model.

---

## Command Internals — Step Architecture

Mọi command trong `.agent/commands/` được compose từ **step file** dùng chung trong `.agent/steps/`. Các step này là nền móng — mọi command gọi chúng tuần tự trước khi chạy logic riêng.

### Step files at a glance

| File | Role | Called by |
|------|------|-----------|
| `gate.md` | Universal entry — model check, file resolution, context load, user checkpoint | Every command |
| `context-loader.md` | Multi-step context loading (stack → service routing → service conventions → arch → safety → domain → UI → recap) | `gate.md` Step 2 |
| `spawn-agent.md` | Sub-agent orchestration cho PRD lớn | `generate-bdd`, `generate-code`, `dev-gen-test` |
| `capture-lesson.md` | Append/refine guardrail vào project lessons file | `learn`, `review-code`, `fix-bug`, `debug` |
| `report-footer.md` | Standard output format (status badge, artifact list, next command) | Every command |

### gate.md — Universal Entry Procedure

| Step | What it does |
|------|-------------|
| **Step 0** | Sub-agent mode check — nếu `$ARGUMENTS` là JSON payload có `_agent_mode: true`, skip Steps 1–3 và dùng slim context trực tiếp |
| **Step 0-B** | Model check — prompt chuyển sang `claude-opus`; `S` để skip |
| **Step 1** | Resolve target file từ `$ARGUMENTS`; nếu thiếu, list candidates và hỏi user |
| **Step 2** | Execute `context-loader.md` — load mọi project context vào working memory |
| **Step 3** | CHECKPOINT — hiện summary (target file, stack, module, domains), chờ `Y` |

### context-loader.md — Context Loading Sequence

Load context theo thứ tự ưu tiên nghiêm ngặt (anti-lost-in-middle: facts critical load trước, RECAP restate cuối):

| Step | Priority | What loads |
|------|----------|-----------|
| 1 | PROJECT-CONFIG | `project-context.yaml` → stack, conventions, domains, services, paths |
| 1.5 | SERVICE ROUTING | *(umbrella only)* detect active domain, route `trace_dir` (+ code) → service submodule, store `service_root`. Khi `spec_source` set: `specs_dir` (BDD) / `tech_docs_dir` / PRD / design-spec / domain-knowledge / feedback → **spec repo** (cross-team), KHÔNG per-service |
| 1.6 | SERVICE CONVENTIONS | *(umbrella only)* load `{service_root}/.agent/project-context.yaml` → override `test_command`, `build_command`, `paths.trace_dir` |
| 2 | PROJECT-CONFIG | `.agent/modules/{module}/stack-profile.yaml` → framework-specific layer patterns |
| 3 | **CRITICAL** | `CLAUDE.md` → architecture layers, coding standards, naming |
| 4 | SAFETY | `.agent/rules/data-protection.md` → sensitive file patterns |
| 5 | DOMAIN | `business-dictionary.md` → canonical + banned terms |
| 6 | DOMAIN | `core-entities.md` → entity catalog |
| 6.7 | **GUARDRAILS** | `lessons_file` → accumulated lessons (via `/learn`) as hard constraints |
| 6-B | UI COMPONENTS | `figma-components/{module}.md` → Figma → code component + import path |
| 6-C | UI TOKENS | `figma-tokens.md` → design tokens |
| 7 | **RECAP** | `[CTX LOADED]` block printed để khóa critical facts vào working memory |

Status values của RECAP: `FULL` · `PARTIAL — missing: {list}` · `MINIMAL` (chỉ project-context.yaml).

### spawn-agent.md — Sub-Agent Orchestration

Khi PRD vượt ngưỡng complexity, `generate-bdd` / `generate-code` / `dev-gen-test` tự chuyển từ single-session sang orchestration mode.

| Signal | Threshold | Action |
|--------|-----------|--------|
| UC count in PRD | > 3 UCs | Spawn 1 sub-agent per UC |
| PRD length | > 300 lines | Spawn agents regardless of UC count |

Orchestrator: Step A build slim context JSON → Step B scan PRD → UC list → Step C announce plan → Step D spawn 1 sub-agent/UC (parallel, mỗi agent chỉ đọc UC section của mình) → Step E collect + merge results.

### report-footer.md — Standard Output Format

Mọi command kết thúc bằng footer: `Status` badge (`✅ Complete` · `⚠️ Warnings` · `❌ Failed`), danh sách **Output Artifacts** (created/updated), và field **Next** gợi ý command kế tiếp.

---

*Xem thêm:* [Trace TSV Schema](trace-schema.md) · [Stack Modules](modules.md) · [02 · Guides](../02-guides/) (theo vai trò).
