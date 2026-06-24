[📚 Docs](../README.md) › [Concepts](README.md) › Pipeline

# Pipeline

Vòng đời feature đi qua các phase, mỗi transition có một **AI review gate**:
**Discovery → PRD → Design-Spec → BDD → Tech-Docs → Code → Dev self-check → QC automation → Tester feedback.** Phần sau mô tả từng phase và **step-architecture model** đằng sau mỗi command.

## Mục lục

- [Phase overview](#phase-overview)
- [Workflow chi tiết](#workflow-chi-tiết)
- [QC automation pipeline (Phase 5b)](#qc-automation-pipeline-phase-5b)
- [Command internals — step architecture](#command-internals--step-architecture)
  - [gate.md — universal entry](#gatemd--universal-entry)
  - [context-loader.md — context loading sequence](#context-loadermd--context-loading-sequence)
  - [spawn-agent.md — sub-agent orchestration](#spawn-agentmd--sub-agent-orchestration)
  - [report-footer.md — standard output format](#report-footermd--standard-output-format)

---

## Phase overview

| Phase | Who | Commands | Output |
|-------|-----|----------|--------|
| **0. Setup** *(one-time)* | Tech Lead | `/setup-ai-first`, `/sync`, `/sync-figma-*` | Config files, submodules, component catalog |
| **1. Discovery** | PO + AI | `/define-product` | `specs/product-definition/{TICKET-ID}-{slug}.md` |
| **2. PRD** | AI → SA/PO review | `/generate-prd`, `/refine-prd`, `/review-context` | `specs/{domain}/{prd-slug}/prd.md` |
| **2b. Design Spec** *(FE/App only)* | AI → Designer/PO sign-off | `/generate-design-spec` | `specs/{domain}/{prd-slug}/design-spec/{TICKET-ID}-design-spec-{platform}.md` |
| **3. BDD Spec** | AI → SA/Dev review | `/generate-bdd`, `/review-context` | `specs/{domain}/{prd-slug}/bdd/{UC-ID}.feature` |
| **4. Tech Design** *(platform-aware)* | AI → SA/Lead review | `/generate-tech-docs`, `/review-tech-docs` | BE: `specs/{domain}/{prd-slug}/tech-docs/{UC-ID}-tech-design.md` (API contract) · FE/App: `{UC-ID}-tech-design-{platform}.md` (client design — **gated** trên System BDD + BE contract) |
| **5. Code** | AI → Dev review | `/generate-code` *(FE: `--phase=ui`/`--phase=integration`)*, `/review-code` | `src/...` |
| **6. Dev Self-Check** | Dev (own code) | `/dev-gen-test`, `/dev-run-test`, `/dev-smoke-test` | `src/test/...` — dev smoke (`dev_selftest`), tách khỏi QC suite |
| **6b. QC Automation** *(official QC suite)* | QC | `/qc-analyze` → `/qc-plan` → `/qc-design-test` → `/qc-review` → `/qc-run-test` → `/qc-report` | QC test designs + run results — set `qc_status` (`qc-playwright` module) |
| **7. Trace** | Tech Lead | `/validate-traces` | Coverage matrix + drift report |
| **8. Tester Feedback** | QA → PO/Dev | `/report-bug`, `/propose-scenario` → `/sync` surfaces | `{spec}/feedback/...` → bug fix / new scenario / PRD update |
| **Cross-cutting** | Any role | `/sync`, `/learn`, `/update-framework` | Synced repo · project guardrails · upgraded tooling |

> **PRD là platform-agnostic (Option C):** Một PRD phục vụ mọi platform — chỉ mô tả business outcome, không UI/API. FE/App: PRD → Design Spec (2b) → BDD. BE: PRD → BDD (skip 2b). Thêm platform sau không cần đổi PRD.

---

## Workflow chi tiết

```
Phase 1: Discovery
  /define-product ──────────→ specs/product-definition/{slug}.md
        (7 phase Q&A: knowledge sync → feature def → user flow → clarify →
         business rules → business logic → AC → validation matrix)

Phase 2: PRD
  /generate-prd ────────────→ specs/{domain}/{prd-slug}/prd.md
  /refine-prd ──────────────→ .agent/review/{prd-slug}-findings.yaml
                  [Review Board: Accept/Modify/Reject]
                /refine-prd --resume → apply + bump version
  /review-context {prd} ────→ .agent/review/{prd-slug}-review-context-findings.yaml
       --fix (auto-fixable)  |  Review Board → --resume (apply + bump)
       Checks: banned terms (P1) · ambiguity AC/BR (P2) · conflicts (P3) · completeness (P4)
       ✅ APPROVED → Phase 3   ·   ❌ NEEDS_FIX → fix + re-run

Phase 2b/3: Design Spec & BDD
  /generate-design-spec ────→ specs/{domain}/{prd-slug}/design-spec/{TICKET-ID}-design-spec-{platform}.md
       (FE/App only — PO supplies node-level Figma link ?node-id= per screen;
        AI fetches each frame via Figma MCP. Screen with no readable link → ❌ Missing;
        sign-off + /generate-bdd blocked tới khi mọi screen có link)
              [Designer review + PO sign-off]
  /generate-bdd ────────────→ specs/{domain}/{prd-slug}/bdd/{UC-ID}.feature
       THỨ TỰ OUTSIDE-IN (khi có client): web → app → system
         System BDD được TỔNG HỢP từ web+app BDD (client-facing trước → BE/system suy ra để phục vụ)
         (project chỉ-BE, không web/app: system gen thẳng từ PRD)
       (apply platform vocabulary: web "clicks" / mobile "taps" / backend "submits a request")
  /review-context {feature} → .agent/review/{uc-id}-review-bdd-findings.yaml
       Checks: PRD coverage (mỗi AC + BR bullet → ≥1 scenario) · Gherkin R1–R10 · compliance C1–C5
       ✅ APPROVED → Phase 4

Phase 4: Tech Design  (platform-aware — đọc @trace.platform)
  BE (system):
    /generate-tech-docs {system .feature} → specs/{domain}/{prd-slug}/tech-docs/{UC-ID}-tech-design.md
         API contract: endpoints, data model, DB, caching (brownfield: reverse-document)
  FE/App (web|app) — GATED, cần CÓ trước: System BDD + BE contract (approved):
    /generate-tech-docs {web|app .feature} → specs/{domain}/{prd-slug}/tech-docs/{UC-ID}-tech-design-{platform}.md
         client design: components, state, API-integration map theo BE contract, routing,
         §2b Test Selectors (test-id ổn định cho element có action → QC locate khỏi scan)
         (thiếu System BDD hoặc BE contract → HALT, in hướng dẫn)
  /review-tech-docs ────────→ .agent/review/{uc-id}-tech-review-findings.yaml
              [Review Board] → --resume (apply + bump revision)
       ✅ APPROVED → Phase 5

Phase 5: Code
  /generate-code ───────────→ src/...  (@trace.implements tags)
       FE: --phase=ui (UI + mock adapter, tester-ready)
           mock shape: BE contract nếu có (chuẩn) → else System BDD (tạm, warn) — fixture values luôn từ System BDD
           → khi BE contract approved → /generate-tech-docs {web|app} (FE design, Phase 4)
           → --phase=integration (wire real API theo §4 của FE tech-design)
       (component enforcement: ✅ existing / ⚠️ TODO blocked / ❌ NEW confirm)
  /review-code ─────────────→ Report: Critical / Major / Minor → fix CRITICAL/MAJOR

Phase 5 (cont): Dev Self-Check  (dev verifies their OWN code — NOT the official QC suite)
  /dev-gen-test ────────────→ src/test/...  (@trace.verifies tags)
  /dev-run-test ────────────→ records dev_selftest (pass/fail/not_run) + dev_selftest_at in trace
  /dev-smoke-test ──────────→ live endpoint check (optional)
  /validate-traces {domain} → coverage matrix + drift detection

Phase 5b: QC Automation  (the OFFICIAL QC suite — see below)

Phase 6: Tester / QC Feedback  (QA/QC → PO/Dev, closes the loop)
  /report-bug {UC-ID} ──────→ {spec}/feedback/bug-reports/{BUG-ID}.md (State: Open) → push
  /propose-scenario {UC-ID} → behavior ∈ AC → {spec}/feedback/bdd-proposals/{...}.md → push
                              behavior MỚI  → {spec}/feedback/prd-change-requests/{...}.md → push
              QC nguồn: /qc-run-test FAIL & /qc-analyze DOC_GAPS cũng route vào đây
              PO/Dev: /sync → "📥 State: Open" → /fix-bug {BUG-ID} (Open→Fixed) · add BDD · update PRD
              QC: /qc-run-test re-verify pass → bug Closed (qc_owner/qc_blocked_by clear)

Cross-cutting (any role, anytime)
  /sync ────────────────────→ git pull + submodules + Living Docs + surface feedback
  /learn "AI does X, should Y" → project guardrail (loaded vào mọi command)
  /update-framework ────────→ upgrade framework tooling từ npm
```

> **Large PRD (> 3 UC hoặc > 300 dòng):** `/generate-bdd`, `/generate-code`, `/dev-gen-test` tự chuyển sang **orchestration mode** — spawn 1 sub-agent/UC (xem [spawn-agent.md](#spawn-agentmd--sub-agent-orchestration)).

> **Phase 7 — `/validate-traces` statuses:** ✅ OK (code version khớp spec) · ⚠️ DRIFT (code gen từ PRD/BDD cũ → re-generate) · 🔴 GAP (scenario có trong spec nhưng không có code) · — UNTRACKED (scenario ghi nhận nhưng chưa code-gen). Chi tiết: [traceability.md](traceability.md).

---

## QC automation pipeline (Phase 5b)

Bộ test QC **chính thức** — native pipeline, chạy như một **branch sau khi BDD được approve** (`/review-context` (BDD) APPROVED), song song / sau khi dev `/generate-code`. 6 stage tuần tự, port từ agent của team QC (QC repo nay chỉ còn reference):

```
BDD approved (specs/{domain}/{prd-slug}/bdd/*.feature)
        ▼
/qc-analyze → /qc-plan → /qc-design-test → /qc-review → /qc-run-test → /qc-report
   (đọc UC/SC, phân tích)                          chạy test → set qc_status
        ▼                                                ▼
  QC analysis + test cases                      .trace/*.tsv: qc_status
  → {qc_dir}/{UC-ID}/  (visible, mặc định docs/) (pass/fail/skip/not_run)
    REQUIREMENT_ANALYSIS.md · DOC_GAPS.md ·       + qc_owner / qc_blocked_by
    TEST_PLAN.md · test-cases/*.Test.md           (skills load từ {qc_skills_dir})

  ── Feedback loop về spec (đóng vòng) ───────────────────────────────────
  /qc-analyze DOC_GAPS blocker (spec sai/mơ hồ) ┐
  /qc-run-test FAIL = product-gap               ├─▶ /report-bug · /propose-scenario
  /qc-report tổng hợp gap → in sẵn lệnh         ┘            │  (cùng flow tester)
                                                             ▼
                                  feedback/ trong spec repo (State: Open)
                                  → commit+push → PO/Dev thấy qua /sync (xem bug-flow.md)

Mapping: mỗi QC test → scenario qua @trace.verifies={UC-ID}-SC{N}
```

- **Implementation module:** `qc-playwright` (Python + pytest-playwright + Page Object; Playwright Trace + pytest-html, KHÔNG dùng Allure). ĐỘC LẬP với dev implementation module — dùng bởi `/qc-run-test` & `/qc-report`.
- **Khác biệt với dev self-check:** dev self-check (`/dev-*`) → `dev_selftest` (smoke, dev tự kiểm); QC automation (`/qc-*`) → `qc_status` (coverage QC chính thức). Hai tín hiệu **orthogonal**, cả hai surface trong Living Docs → xem [traceability.md](traceability.md).

---

## Command internals — step architecture

Mỗi command trong `.agent/commands/` được compose từ các **step file** dùng chung trong `.agent/steps/`. Đây là nền tảng — mọi command gọi chúng theo thứ tự trước khi chạy logic riêng.

| File | Role | Called by |
|------|------|-----------|
| `gate.md` | Universal entry — model check, file resolution, context load, user checkpoint | Mọi command |
| `context-loader.md` | Multi-step context loading (stack → service routing → conventions → arch → safety → domain → UI → recap) | `gate.md` Step 2 |
| `spawn-agent.md` | Sub-agent orchestration cho PRD lớn | `generate-bdd`, `generate-code`, `dev-gen-test` |
| `capture-lesson.md` | Append/refine guardrail trong project lessons file | `learn`, `review-code`, `fix-bug`, `debug` |
| `report-footer.md` | Standard output format (status badge, artifact list, next command) | Mọi command |

### gate.md — universal entry

Mọi command bắt đầu ở đây, theo thứ tự:

| Step | What it does |
|------|-------------|
| **Step 0** | Sub-agent mode check — nếu `$ARGUMENTS` là JSON payload với `_agent_mode: true`, skip Step 1–3, dùng slim context trực tiếp |
| **Step 0-B** | Model check — prompt switch sang `claude-opus` cho generation phức tạp; `S` để skip |
| **Step 1** | Resolve target file từ `$ARGUMENTS`; nếu thiếu, liệt kê candidate và hỏi user chọn |
| **Step 2** | Execute `context-loader.md` — load toàn bộ project context vào working memory |
| **Step 3** | CHECKPOINT — hiển thị summary (target file, stack, module, domains), chờ `Y` |

```
CHECKPOINT
-----------
Target     : specs/auth/FEAT-042-login/prd.md
Project    : My App
Tech stack : TypeScript / React 18
Module     : react
Domains    : auth, profile

Proceed? (Y/N)
```

### context-loader.md — context loading sequence

Load toàn bộ project context vào working memory theo thứ tự ưu tiên nghiêm ngặt:

| Step | Priority | What loads |
|------|----------|-----------|
| 1 | PROJECT-CONFIG | `project-context.yaml` → stack, conventions, domains, services, paths |
| 1.5 | SERVICE ROUTING | *(umbrella only)* detect active domain, route `trace_dir` (+ code) tới service submodule, store `service_root`. Khi `spec_source` set: `specs_dir` (BDD) / `tech_docs_dir` / PRD / design-spec / domain-knowledge / feedback → **spec repo** (cross-team), KHÔNG per-service |
| 1.6 | SERVICE CONVENTIONS | *(umbrella only)* load `{service_root}/.agent/project-context.yaml` → override `test_command`, `build_command`, `paths.trace_dir` |
| 2 | PROJECT-CONFIG | `.agent/modules/{module}/stack-profile.yaml` → layer patterns |
| 3 | **CRITICAL** | `CLAUDE.md` → architecture layers, coding standards, naming |
| 4 | SAFETY | `.agent/rules/data-protection.md` → sensitive file patterns |
| 5 | DOMAIN | `business-dictionary.md` → canonical + banned terms |
| 6 | DOMAIN | `core-entities.md` → entity catalog |
| 6.7 | **GUARDRAILS** | `lessons_file` → mistakes tích lũy (via `/learn`) loaded như hard constraints |
| 6-B | UI COMPONENTS | `figma-components/{module}.md` → Figma name → code component + import path |
| 6-C | UI TOKENS | `figma-tokens.md` → design tokens |
| 7 | **RECAP** | `[CTX LOADED]` block — lock critical facts vào working memory |

**Anti-lost-in-middle:** critical facts (Step 3) load đầu tiên; RECAP (Step 7) restate ở cuối để fresh khi generation bắt đầu (recency effect).

```
[CTX LOADED]
Stack      : TypeScript / React 18 / PostgreSQL
Layers     : Controller → Facade → Service → Repository
Services   : 2 services: web-admin(react), api-backend(java-spring)
Svc Root   : api-backend — conventions + trace_dir loaded from service config
Dict       : loaded — 42 canonical terms, 8 banned terms
Entities   : loaded — User, Order, Product
Lessons    : loaded — 6 guardrails
Components : loaded — web-admin (react) — 23 components mapped
Tokens     : loaded — colors: 18, spacing: 12, typography: 6
Status     : FULL
```

Status values: `FULL` · `PARTIAL — missing: {list}` · `MINIMAL` (chỉ project-context.yaml).

### spawn-agent.md — sub-agent orchestration

Khi PRD vượt ngưỡng phức tạp, `generate-bdd`, `generate-code`, `dev-gen-test` tự chuyển từ single-session sang orchestration mode.

**Complexity thresholds:**

| Signal | Threshold | Action |
|--------|-----------|--------|
| UC count trong PRD | > 3 UC | Spawn 1 sub-agent / UC |
| PRD length | > 300 dòng | Spawn agents bất kể UC count |

**Orchestration flow:**

```
Main session (orchestrator)
  ├─ Step A: Build slim context JSON (stack + active_service + active_module + paths + arch summary + banned_terms)
  ├─ Step B: Scan PRD → extract UC list [{uc_id, line_start, line_end}]
  ├─ Step C: Announce plan → "Spawning N sub-agents..."
  ├─ Step D: Spawn 1 sub-agent / UC (parallel)
  │     mỗi agent nhận: { _agent_mode: true, uc_id, target_file, uc_section, context }
  │     mỗi agent đọc CHỈ UC section của mình (không phải full PRD)
  └─ Step E: Collect results → merge → { uc_id, files_created, status, errors }
```

**Tiết kiệm context window:**

| Mode | What loads per agent |
|------|---------------------|
| Single session (≤ 3 UC) | Full context + full PRD + all UCs |
| Orchestrator | Slim context + UC heading list only |
| Each sub-agent | Slim context + **1 UC section only** |

### report-footer.md — standard output format

Mọi command kết thúc với footer này:

```
---
Status : ✅ Complete
Output Artifacts:
  created  specs/auth/FEAT-042-login/bdd/FEAT-042-UC1-login.feature (3 scenarios)
  updated  .trace/auth/FEAT-042-login/FEAT-042.tsv
Next   : /review-context specs/auth/FEAT-042-login/bdd/FEAT-042-UC1-login.feature
```

**Status badges:** `✅ Complete` · `⚠️ Warnings` · `❌ Failed`. Field **Next** luôn gợi ý command tiếp theo hợp lý — không phải tra cứu.
