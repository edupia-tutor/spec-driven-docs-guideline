[📚 Docs](../README.md) › [Operations](README.md) › Sync & Update

# Sync & Update — Vận hành hằng ngày

> Cách đồng bộ nội dung dự án (`/sync`), nâng cấp framework (`/update-framework`), làm việc ở umbrella mode với git submodule, đồng bộ Living Docs, và workflow hằng ngày theo từng role.

---

## Mục lục

1. [Hai loại "update" — đừng nhầm](#1-hai-loại-update--đừng-nhầm)
2. [`/sync` — đồng bộ nội dung dự án](#2-sync--đồng-bộ-nội-dung-dự-án)
3. [`/update-framework` — nâng cấp framework](#3-update-framework--nâng-cấp-framework)
3.5. [`--migrate-specs` — chuyển specs cũ sang feature-package layout](#35---migrate-specs--chuyển-specs-cũ-sang-feature-package-layout-một-lần)
4. [Umbrella mode & git submodule](#4-umbrella-mode--git-submodule)
5. [Living Docs sync](#5-living-docs-sync)
6. [Workflow hằng ngày theo role](#6-workflow-hằng-ngày-theo-role)
7. [Project Lessons — không để AI lặp lỗi](#7-project-lessons--không-để-ai-lặp-lỗi)
8. [Câu hỏi thường gặp](#8-câu-hỏi-thường-gặp)

> Pipeline QC native (`/qc-*`) có trang riêng — xem [chương QC Automation](../02-guides/tester/qc-automation.md). Ở đây chỉ điểm qua chỗ `/sync` chạm vào QC.

---

## 1. Hai loại "update" — đừng nhầm

Có 2 thứ cần update, **nguồn khác nhau, tần suất khác nhau**:

| | `/sync` | `/update-framework` |
|---|---|---|
| **Update cái gì** | Nội dung dự án: code/specs trong submodule + Living Docs | Bản thân framework: `.agent/commands/`, steps/, modules/ |
| **Nguồn** | Git remote của các submodule | npm registry |
| **Tần suất** | Mỗi sáng / trước khi work | Khi có version framework mới (hiếm) |
| **Đụng `project-context.yaml` / `CLAUDE.md`?** | Không | Không (chỉ refresh file framework) |

```bash
# Update nội dung dự án — chạy thường xuyên
/sync

# Nâng cấp framework — chạy khi có bản mới
/update-framework
```

---

## 2. `/sync` — đồng bộ nội dung dự án

`/sync` gộp `git pull` + cập nhật submodule + refresh Living Docs vào **1 lệnh duy nhất**, chạy từ umbrella root:

- Pull specs (spec submodule) + services (service submodules).
- Refresh Living Docs panel (regenerate trace report — xem [§5](#5-living-docs-sync)).
- Liệt kê **📥 New tester feedback** — bug report / scenario proposal mới từ tester trong vùng `feedback/` của spec repo. Chỉ surface bug có `State: Open` là đang chờ; `State: Fixed` (chờ QC re-verify) / `Closed` liệt kê riêng để không làm nhiễu view của PO/PM. Vòng đời đầy đủ: [bug-flow.md §10](bug-flow.md).

```bash
/sync
# hoặc override branch của spec submodule một lần:
/sync develop
```

**Branch nào được kéo?** `/sync` đọc `submodule.<spec>.branch` trong `.gitmodules`; nếu không khai báo → lấy **default branch của remote** (`origin/HEAD`, thường `main`). PO làm trên branch khác (vd `develop`) → pin một lần cho cả team:

```bash
git config -f .gitmodules submodule.my-project-specs.branch develop
git add .gitmodules && git commit -m "chore: pin spec submodule branch = develop"
```

Service submodules không bị ảnh hưởng — luôn checkout đúng SHA mà umbrella ghi, không phụ thuộc branch.

---

## 3. `/update-framework` — nâng cấp framework

```bash
/update-framework
# → đọc .agent/FRAMEWORK_VERSION, so với npm
# → chạy npx @latest --init
# → review `git diff .agent/` rồi commit
```

> **Umbrella mode:** `/update-framework` chỉ cần chạy ở **umbrella root**. Service submodule chỉ chứa `.agent/project-context.yaml` (config), không có command files → không cần nâng cấp riêng.

> ⚠️ **`--init` ghi đè TOÀN BỘ `.agent/`** (gồm cả `.agent/skills/qc/`) — KHÔNG đụng `.agent/project-context.yaml` (config của bạn) và mọi thứ **ngoài** `.agent/`.
> **Hệ quả với skill QC:** nếu QC sửa skill **trực tiếp trong `.agent/skills/qc/`** thì sẽ **mất sau upgrade**. Để skill QC tồn tại độc lập, trỏ `paths.qc_skills_dir` (trong `.agent/project-context.yaml`) tới **repo QC riêng / submodule** (vd `qc-base/.claude/skills`) — `qc_skills_dir` nằm ngoài `.agent/` nên upgrade không bao giờ chạm. Chi tiết: [chương QC Automation](../02-guides/tester/qc-automation.md#skill-sourcing--upgrade-safety).

---

## 3.5. `--migrate-specs` — chuyển specs cũ sang feature-package layout (một lần)

Từ phiên bản dùng **feature-package layout**, mọi artifact của một PRD nằm chung một thư mục
`specs/{domain}/{prd-slug}/` thay vì tách theo loại (`specs/prd/`, `specs/bdd/`, …). Dự án **đã có
specs theo layout cũ** chạy lệnh sau **một lần** để di chuyển:

```bash
# DRY-RUN — chỉ in kế hoạch, không đổi gì (chạy trước để review)
npx @edupia-tutor/spec-driven-docs --migrate-specs

# Thực thi
npx @edupia-tutor/spec-driven-docs --migrate-specs --apply
# hoặc trực tiếp: node scripts/migrate-specs.js --apply
```

Lệnh sẽ:
- Di chuyển `specs/prd/{domain}/{slug}.md` → `specs/{domain}/{slug}/prd.md`, và `specs/bdd|tech-docs|design-spec/{domain}/…` → `specs/{domain}/{prd-slug}/{bdd|tech-docs|design-spec}/…`; flat `.trace/{UC-ID}.tsv` → `.trace/{domain}/{prd-slug}/{UC-ID}.tsv`.
- Suy ra `{prd-slug}` cho mỗi file BDD/tech-doc/design-spec/trace bằng cách đọc header `@trace.source` (PRD nguồn) hoặc tag `@trace.uc` (map qua index `.feature`).
- Dùng `git mv` cho file đã track (giữ history), `fs.rename` cho phần còn lại.
- Rewrite các tham chiếu nội bộ (`@trace.source`, đường dẫn `specs/…`) trong những file vừa move.
- File **không resolve được** prd-slug sẽ **để nguyên** và liệt kê trong report — không bao giờ move vào chỗ đoán.

> ⚠️ **Phạm vi:** lệnh chỉ đụng `specs/` và `.trace/`. Tham chiếu `@trace.source=specs/bdd/…` trong **code** (file `src/`) không được sửa tự động — sau khi migrate, grep codebase:
> ```bash
> grep -rn "specs/\(prd\|bdd\|tech-docs\|design-spec\)/" src/
> ```
> rồi cập nhật thủ công, và chạy `/validate-traces` để xác nhận coverage vẫn resolve trước khi commit.

---

## 4. Umbrella mode & git submodule

Claude Code luôn mở ở **umbrella repo** (không mở trong service submodule). Umbrella thấy được cả spec submodule (read-only) lẫn service submodule (code).

### Ai commit vào repo nào? (git flow theo role)

3 repo độc lập (mỗi cái có origin riêng), umbrella chỉ giữ **pointer**. Mỗi role ghi vào repo khác nhau → số "tầng push" khác nhau:

```
                UMBRELLA REPO  (Claude Code mở ở đây · .gitmodules + docs/)
                      │  giữ pointer (SHA) tới từng submodule
        ┌─────────────┴──────────────┐
        ▼                             ▼
 SPEC SUBMODULE                       SERVICE SUBMODULE(s)
 (PO sở hữu · origin riêng)           (Dev sở hữu · origin riêng)
 specs/{domain}/{prd-slug}/          src/   (chỉ code + tooling)
   prd.md · bdd/ (web/app/system)
   design-spec/ · tech-docs/
 feedback/ (bug · proposal · prd-change)
 .trace/{domain}/{prd-slug}/*.tsv  (qc_status + dev_selftest — write area của dev/QC)
```

> **Tất cả spec + `.trace/` ở SPEC repo; service submodule chỉ chứa code.** PRD/BDD/design-spec/tech-docs read-only với dev/QC; `feedback/` và `.trace/` là vùng dev/QC được ghi.

| Role | Tạo gì | Repo đích | Push mấy tầng |
|------|--------|-----------|---------------|
| **PO** | PRD · BDD (web/app/system) · design-spec · sửa PRD theo change-request | SPEC repo | **1 tầng** (spec repo) |
| **Dev (FE/BE)** | tech-docs · code · dev test (`dev_selftest`) | tech-docs → SPEC repo · `dev_selftest` → SPEC `.trace/` · code → SERVICE | code: **2 tầng** (service → umbrella pointer) · tech-docs + trace: 1 tầng ở spec repo (cross-repo write) |
| **QC / Tester (QA)** | QC pipeline (analysis/test-cases · `qc_status`) · bug report · scenario proposal · prd-change-request | artifacts → `{qc_dir}/{UC-ID}/` (umbrella `docs/`) · `qc_status` → SPEC `.trace/` · bug/proposal/change-request → SPEC `feedback/` | artifacts: 1 tầng (umbrella) · trace + feedback: **1 tầng** (spec repo) — read-only trên specs/code |

> **QC = Tester (một vai trò QA):** cùng một người chạy pipeline `/qc-*` (sinh `qc_status` chính thức) **và** `/report-bug` · `/propose-scenario`. Phân biệt thật sự là **Dev self-test (`dev_selftest`) vs QC/Tester authoritative (`qc_status`)** — không phải QC vs Tester.

Thứ tự bàn giao (mỗi mũi tên = `git push` rồi role kế tiếp `/sync`):

```
PO ─push spec─▶ Dev /sync ─code 2 tầng─▶ QC/Tester /sync ─qc_status + bug 1 tầng (spec)─▶ PO/Dev /sync
 BDD approved      tech-docs + src         test + report                 fix code / sửa PRD → /generate-bdd lại
                                                                         (vòng feedback: xem Bug Flow §9)
```

> **`qc_status` ở spec repo nhưng QC analysis docs ở umbrella root:** `qc_status` ghi vào `.trace/` (đã dồn về `{spec_source}/.trace`), còn `qc_dir` (analysis/test-cases, mặc định `docs/`) **không** auto-route — nằm ở umbrella root. Muốn artifacts QC nằm chỗ khác → set `paths.qc_dir` trong `.agent/project-context.yaml`, hoặc trỏ ra repo QC riêng.

> **Git flow `git` cụ thể cho từng role** (ví dụ end-to-end một feature, kèm `cd`/`add`/`commit`/`push` thực tế): [§4.7](#47--git-flow-cụ-thể-theo-role-ví-dụ-feature-ft-042-checkout-domain-payment).

### 4.1 — Clone lần đầu

```bash
git clone --recurse-submodules https://gitlab.company.com/my-project-web.git

# Hoặc nếu đã clone mà submodule trống:
git submodule update --init --recursive
```

### 4.2 — Kiểm tra trạng thái submodule

```bash
git submodule status
```

| Ký hiệu | Ý nghĩa | Cách xử lý |
|---|---|---|
| `abc1234 my-project-specs` | Bình thường | — |
| `+abc1234 …` | Umbrella có thay đổi chưa commit | `git add <sub> && git commit` |
| `-abc1234 …` | Chưa init | `git submodule update --init` |
| `Uabc1234 …` | Conflict | Giải quyết conflict rồi commit |

### 4.3 — Cập nhật spec submodule (lấy PRD mới từ PO)

```bash
# Cách 1: kéo version mới nhất của branch (thường dùng nhất)
git submodule update --remote my-project-specs

# Cách 2: kéo version mà umbrella đang trỏ (sau git pull)
git submodule update my-project-specs

# Commit lại pointer để cả team cùng version
git add my-project-specs
git commit -m "chore: update spec submodule to latest"
git push
```

> **Khác biệt:** `--remote` lấy commit mới nhất từ remote branch (khi PO vừa push); không có `--remote` → sync về đúng commit umbrella đang trỏ. `/sync` xử lý việc này tự động.

### 4.4 — Commit 2 tầng (thay đổi trong service submodule)

```bash
# Tầng 1: commit trong service submodule (CHỈ code — BDD, tech-docs, .trace đều ở spec repo)
cd user-service
git add src/
git commit -m "feat(FEAT-01): implementation"
git push origin main
cd ..

# Tầng 2: update pointer ở umbrella
git add user-service
git commit -m "chore: update user-service submodule to FEAT-01"
git push
```

> ⚠️ **PHẢI push cả 2 tầng.** Push umbrella mà quên push submodule → người khác pull về bị lỗi "commit không tồn tại".
>
> **Tech-docs (API contract):** khi `setup.spec_source` được set, tech-design **luôn** nằm ở `{spec_source}/specs/tech-docs` (spec repo chung) — commit + push riêng trong spec submodule, FE/App đọc qua `/sync`. Per-service tech-docs chỉ dùng khi **không** có `spec_source`.
>
> **`.trace/` cũng ở spec repo:** khi `spec_source` set, `/generate-code` · `/dev-run-test` · `/qc-run-test` chạy trong service nhưng ghi trace vào `{spec_source}/.trace` → commit + push **trong spec submodule** (1 tầng, cross-repo, giống `feedback/`). Service submodule **không** còn `.trace/`.

### 4.5 — Lỡ tay sửa **spec** trong spec submodule (PRD/BDD/design-spec/tech-docs read-only)

PRD/BDD/design-spec/tech-docs trong spec submodule là **read-only** cho dev/tester (chỉ PO sửa). `feedback/` và `.trace/` thì dev/QC được ghi. Nếu lỡ sửa spec → bỏ thay đổi:

```bash
cd my-project-specs
git checkout .      # hoặc: git restore .
```

Muốn cập nhật PRD → báo PO sửa trong spec repo của họ rồi push.

### 4.6 — Troubleshooting

| Lỗi | Nguyên nhân | Cách xử lý |
|---|---|---|
| Thư mục submodule trống | Chưa init | `git submodule update --init` |
| `fatal: not a git repository` | Sai thư mục | `cd` về đúng chỗ |
| `object not found` sau pull | Push umbrella nhưng quên push submodule | Người commit submodule chạy `cd <sub> && git push` |
| Submodule ở `detached HEAD` | Bình thường khi dùng submodule | Không cần lo nếu không sửa file trong đó |
| Conflict trong submodule pointer | 2 người cùng update submodule | `git checkout --theirs <sub> && git add <sub>` |

### 4.7 — Git flow cụ thể theo role (ví dụ feature **FT-042 Checkout**, domain `payment`)

Topology giả định cho mọi ví dụ dưới đây:

| Repo | Vai trò | Ai sở hữu |
|------|---------|-----------|
| `my-project/` | **umbrella** — Claude Code mở ở đây, giữ pointer | team |
| `my-project-specs/` | **spec** submodule — PRD/BDD/design-spec/tech-docs + `.trace/` + `feedback/` | PO |
| `payment-be/` | **service** submodule — chỉ code BE | BE dev |
| `my-project-web/` | **service** submodule — chỉ code FE | FE dev |

> **Quy tắc 1 dòng:** chỉ **code** mới push **2 tầng** (service → umbrella pointer). Mọi thứ khác — PRD/BDD/tech-docs/`.trace/`/`feedback/` ở spec repo, và QC artifacts ở umbrella — đều **1 tầng**.

**① PO — 1 tầng (spec repo)**
```bash
cd my-project-specs && git pull
# /generate-prd · /generate-design-spec · /generate-bdd (web/app/system) · set @trace.status: approved
git add specs/payment/checkout/prd.md \
        specs/payment/checkout/design-spec/FT-042-checkout-design.md \
        specs/payment/checkout/bdd/system/FT-042-UC1-checkout-system.feature \
        specs/payment/checkout/bdd/web/FT-042-UC1-checkout-web.feature \
        specs/payment/checkout/bdd/app/FT-042-UC1-checkout-app.feature
git commit -m "feat(payment): FT-042 checkout — PRD + design-spec + BDD (web/app/system)"
git push origin main          # → báo Dev team chạy /sync
```

**② BE Dev — code 2 tầng · tech-docs + trace 1 tầng (cross-repo)**
```bash
cd my-project && /sync        # umbrella root: pull spec + services + Living Docs

# (a) tech-docs (API contract) → SPEC repo  [1 tầng]
#     /generate-tech-docs payment/system/FT-042-UC1 ghi vào my-project-specs/specs/payment/checkout/tech-docs/
cd my-project-specs
git add specs/payment/checkout/tech-docs/FT-042-UC1-tech-design.md
git commit -m "docs(payment): FT-042 UC1 BE API contract"
git push origin main
cd ..

# (b) code → SERVICE repo  [Tầng 1]
#     /generate-code payment/system/FT-042-UC1 ghi vào payment-be/src/
cd payment-be
git checkout -b feat/FT-042-checkout
git add src/
git commit -m "feat(payment): FT-042 UC1 checkout endpoint + service"
git push origin feat/FT-042-checkout      # → mở MR trong payment-be
cd ..

# (c) dev_selftest → SPEC repo .trace  [1 tầng, cross-repo]
#     /dev-run-test ghi my-project-specs/.trace/payment/checkout/FT-042.tsv
cd my-project-specs
git add .trace/payment/checkout/FT-042.tsv
git commit -m "chore(trace): FT-042 dev_selftest=pass"
git push origin main
cd ..

# (d) sau khi MR payment-be merge → bump umbrella pointer  [Tầng 2]
cd payment-be && git checkout main && git pull && cd ..
git add payment-be
git commit -m "chore: bump payment-be → FT-042 checkout"
git push origin main
```

**③ FE/Web Dev — 2 phase; push giống BE (code 2 tầng · tech-docs + trace 1 tầng)**
```bash
cd my-project && /sync

# Phase 1 — UI + mock (chưa cần BE deploy)
#   /generate-code payment/web/FT-042-UC1 --phase=ui  → code trong my-project-web
cd my-project-web
git checkout -b feat/FT-042-checkout-ui
git add src/
git commit -m "feat(payment): FT-042 checkout UI + mock adapter"
git push origin feat/FT-042-checkout-ui
cd ..

# Phase 2 — khi BE System BDD + contract approved:
#   (a) FE tech-design (§2b test-ids + §4 API map) → SPEC repo  [1 tầng]
cd my-project-specs
git add specs/payment/checkout/tech-docs/FT-042-UC1-tech-design-web.md
git commit -m "docs(payment): FT-042 UC1 FE web tech-design (§2b test-ids + §4 API)"
git push origin main
cd ..
#   (b) /generate-code payment/web/FT-042-UC1 --phase=integration → wire API thật  [Tầng 1]
cd my-project-web
git add src/
git commit -m "feat(payment): FT-042 wire real checkout API"
git push origin feat/FT-042-checkout-ui
cd ..
# (c) dev_selftest → spec .trace  +  (d) bump umbrella pointer sau merge — giống BE bước (c)(d)
```

**④ QC / Tester (một vai trò QA) — artifacts 1 tầng (umbrella) · qc_status + bug/proposal 1 tầng (spec repo)**

> QC và Tester là **cùng một người**: vừa chạy pipeline `/qc-*` (sinh `qc_status` chính thức) vừa `/report-bug` · `/propose-scenario`. Read-only trên specs + code.

```bash
cd my-project && /sync         # đọc specs read-only
# /qc-analyze · /qc-plan · /qc-design-test · /qc-review · /qc-run-test · /qc-report

# (a) artifacts phân tích/test-case → {qc_dir}/{UC-ID}/ (qc_dir mặc định = docs/) → commit UMBRELLA  [1 tầng]
git add docs/FT-042/           # REQUIREMENT_ANALYSIS.md · TEST_PLAN.md · test-cases/*.Test.md
git commit -m "qc(payment): FT-042 analysis + test design"
git push origin main

# (b) qc_status → SPEC .trace  +  bug/proposal/change-request → SPEC feedback/  [1 tầng, cross-repo]
cd my-project-specs
git add .trace/payment/checkout/FT-042.tsv
git add feedback/bugs/FT-042-UC1-SC3-bug.md              # /report-bug khi qc_status=fail
#   hoặc: feedback/bdd-proposals/FT-042-new-scenario.md  # /propose-scenario (edge case ngoài BDD)
#   hoặc: feedback/prd-change-requests/FT-042-change.md   # đề xuất sửa PRD
git commit -m "qc(payment): FT-042 qc_status=fail (SC3) + bug report"
git push origin main           # → PO/Dev /sync thấy '📥 New tester feedback' → /fix-bug · promote
cd ..
```

> **Tóm tắt số tầng push:** chỉ **code = 2 tầng** (service → umbrella pointer); PRD/BDD/tech-docs/`.trace`/`feedback` (spec repo) = **1 tầng**; QC artifacts `qc_dir` (umbrella) = **1 tầng**. Lỡ tay sửa spec read-only → §4.5.

---

## 5. Living Docs sync

Living Docs panel hiển thị trạng thái trace của toàn bộ feature. Refresh nó bằng `/validate-traces` (hoặc tự động qua `/sync`):

```bash
/validate-traces
```

- Đọc TSVs từ **một nơi duy nhất** `{spec_source}/.trace/*.tsv` (authoritative — commit trong spec repo; mỗi scenario mang `@trace.service` để biết service nào). KHÔNG còn quét/merge từng service.
- Cập nhật cột `dev_selftest` (pass/fail/not_run) + `dev_selftest_at`.
- Ghi `trace-report.json` → `{spec_source}/.living-docs/` (generated, gitignored).
- Mirror thêm về `./.trace` của workspace hiện tại → panel không rỗng kể cả khi đứng trong 1 service submodule đơn lẻ.

> **Report nằm ở đâu:** authoritative `.trace/*.tsv` ở **spec module** `{spec_source}/.trace/` (committed — một chỗ cho PM). Report `trace-report.json` ở `{spec_source}/.living-docs/` + panel mirror `./.trace` đều **generated, gitignored** — regenerate bởi `/sync` hoặc `/validate-traces`.
>
> **.gitignore:** thêm `.trace/` trong repo hiện tại **và** `.living-docs/` trong spec module — đều là mirror read-only, không commit.

### `dev_selftest` vs `qc_status` — đừng nhầm

| Cột | Set bởi | Ý nghĩa |
|---|---|---|
| `dev_selftest` (+ `dev_selftest_at`) | `/dev-run-test` | Dev tự kiểm tra code mình viết (self-check / smoke). **KHÔNG** phải coverage QC chính thức. |
| `qc_status` (+ `qc_run_at`) | `/qc-run-test` (keyed theo `@trace.verifies={UC-ID}-SC{N}`) | Kết quả QC chính thức của framework. |

Sau khi chạy `/qc-run-test` rồi `/validate-traces` (hoặc `/sync`), `qc_status` hiện lên Living Docs cạnh `dev_selftest`.

---

## 6. Workflow hằng ngày theo role

### PO — Viết tài liệu cho tính năng mới

```bash
cd my-project-spec
git pull
# Mở Claude Code tại thư mục này:
/define-product                          # khởi tạo product definition
/generate-prd {product-definition-file}
/refine-prd {prd-file}                   # review 4 lens: QA/DEV/SA/PO
/review-context {prd-file}               # chất lượng + P0 check
/review-context --fix {prd-file}         # auto-fix vấn đề nhỏ
# → update @trace.status: approved khi PRD sẵn sàng
/generate-design-spec {prd-file}         # FE/App
/generate-bdd {prd-file}                 # outside-in: web → app → system (System BDD tổng hợp từ web+app)

git add specs/
git commit -m "feat({domain}): add {TICKET-ID} PRD, design spec, and BDD"
git push
```

### FE/Web Dev — Nhận task mới

```bash
cd my-project-web
/sync                                    # pull + submodule + Living Docs
git add my-project-specs && git commit -m "chore: sync specs" && git push

/review-context my-project-specs/specs/{domain}/{prd-slug}/prd.md
# → P0: @trace.domain khớp services config? @trace.status: approved?
# Đọc Web BDD do PO generate (KHÔNG tự generate BDD):
#   my-project-specs/specs/{domain}/{prd-slug}/bdd/web/{TICKET-ID}-UC*.feature

# ── BE chưa deploy API → làm UI trước ──
/generate-code {domain}/web/{TICKET-ID}-UC1 --phase=ui  # UI + mock (shape: BE contract nếu có, else System BDD + warn)
/dev-gen-test  {domain}/{TICKET-ID}-UC1
/review-code   {files-changed}
/dev-run-test                                           # dev self-check (dev_selftest)

# ── Khi BE đã có System BDD + tech-docs (API contract) approved ──
/generate-tech-docs {domain}/web/{TICKET-ID}-UC1        # FE client design — GATED: cần System BDD + BE contract
/review-tech-docs   {domain}/{TICKET-ID}-UC1-web        # SA/Lead review (bump revision)
# → sau khi sign-off gate approved:
/generate-code {domain}/web/{TICKET-ID}-UC1 --phase=integration   # wire API thật theo §4 FE tech-design

# ── API đã có sẵn ──
/generate-code {domain}/{TICKET-ID}-UC1                  # không cần --phase
/dev-gen-test  {domain}/{TICKET-ID}-UC1
/review-code   {files-changed}
/dev-run-test

# Commit 2 tầng (code) + 1 tầng (tech-docs/trace) — git flow cụ thể: §4.7 ②③
```

### BE Dev — Nhận task mới

```bash
cd my-project-be
/sync
# → cũng liệt kê "📥 New tester feedback" → /fix-bug · promote proposal · báo PO

/review-context my-project-specs/specs/{domain}/{prd-slug}/prd.md
# Đọc System BDD: my-project-specs/specs/{domain}/{prd-slug}/bdd/system/{TICKET-ID}-UC*.feature

/generate-tech-docs {domain}/{TICKET-ID}-UC1
/review-tech-docs   {domain}/{TICKET-ID}-UC1
/generate-code      {domain}/{TICKET-ID}-UC1
/dev-gen-test       {domain}/{TICKET-ID}-UC1
/review-code        {files-changed}      # AI lặp lỗi → accept "Record as lesson?"
/dev-run-test

# Commit 2 tầng vào service submodule theo domain + tech-docs/trace 1 tầng — git flow cụ thể: §4.7 ②
```

### QC / Tester — Chạy native QC pipeline (tóm tắt)

QC và Tester là **một vai trò QA** (xem [Tester guide](../02-guides/tester/README.md)) — cùng người chạy pipeline `/qc-*` **và** `/report-bug` · `/propose-scenario`. Điều kiện: BDD đã `approved`. Module `qc-playwright` (Python + pytest-playwright + Page Object).

```bash
cd my-project-web
/sync
/qc-analyze · /qc-plan · /qc-design-test · /qc-review · /qc-run-test · /qc-report
/validate-traces            # refresh dashboard → qc_status hiện trên Living Docs
```

> `/qc-run-test` ghi `qc_status` + `qc_run_at` vào trace TSV. Chi tiết từng layer: [chương QC Automation](../02-guides/tester/qc-automation.md). Git flow cụ thể (artifacts umbrella + qc_status/bug/proposal → spec): §4.7 ④ (QC/Tester là một role).

---

## 7. Project Lessons — không để AI lặp lỗi

Khi AI lặp đi lặp lại một lỗi, ghi lại thành **lesson** thay vì sửa thủ công mỗi lần. context-loader nạp lesson vào đầu **mọi** lệnh như ràng buộc cứng.

```bash
# Cách 1 — chủ động:
/learn "AI hay gọi repository thẳng từ controller, phải đi qua service layer"

# Cách 2 — tự động: /review-code, /fix-bug, /debug hỏi "Record as a project lesson? (Y/N)" → Y
```

- **Lưu ở:** `paths.lessons_file` (mặc định `specs/domain-knowledge/lessons-learned.md`; umbrella: `.agent/project-lessons.md` mỗi service).
- **Commit file lessons** để cả team cùng được bảo vệ — đây là **bộ nhớ dự án**, không phải fine-tune model.
- Kiểm tra `[CTX LOADED]` có dòng `Lessons: loaded — N guardrails`.

---

## 8. Câu hỏi thường gặp

**Q: Có cần cài framework vào từng service submodule không?**
A: Không. Chỉ cài ở umbrella repo. Service submodule là git repo thường chứa code + specs (+ `.agent/project-context.yaml` config).

**Q: Claude Code nên mở ở đâu?**
A: Luôn ở **umbrella repo** — có config và thấy được cả spec submodule lẫn service submodule.

**Q: Generate BDD ra file ở đâu?**
A: Khi có `spec_source` (umbrella) → **spec repo**: `{spec_source}/specs/{domain}/{prd-slug}/bdd/…` (tất cả BDD web/app/system đều ở đây, cross-team). Trace `.tsv` **cũng** ở spec repo (`{spec_source}/.trace`) — một chỗ cho PM. Service submodule chỉ chứa code. Chỉ khi KHÔNG có `spec_source` thì BDD + trace mới per-service.

**Q: Quên push service submodule, chỉ push umbrella, người khác báo lỗi?**
A: `cd <service> && git push origin main` — xong, người khác pull lại hết lỗi.

**Q: PO sửa PRD, có cần generate lại BDD?**
A: Chạy `/review-context {feature-file}` (B1 check) xem có coverage gap không. Có → `/review-context --resume`. Không cần viết lại toàn bộ BDD.

**Q: Mở nhiều Claude Code cho nhiều umbrella cùng lúc được không?**
A: Được. Mỗi umbrella là session riêng, hoàn toàn độc lập.

**Q: Đang ở `detached HEAD` trong submodule, có sao không?**
A: Bình thường — submodule trỏ tới một commit cụ thể. Chỉ lo nếu định sửa file (phải checkout branch trước). Chỉ đọc thì không sao.

---

*Xem thêm:* [Bug Flow](bug-flow.md) · [Publishing](publishing.md) · [02 · Guides](../02-guides/) · [03 · Concepts — Traceability](../03-concepts/traceability.md).
