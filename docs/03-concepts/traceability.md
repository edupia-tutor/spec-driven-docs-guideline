[📚 Docs](../README.md) › [Concepts](README.md) › Traceability

# Traceability & Living Docs

Mỗi artifact link tới mọi artifact khác qua `@trace.*` tags, và trạng thái coverage/drift được tổng hợp vào **trace TSV** rồi surface trong **Living Docs**. Đây là cách framework đảm bảo *mọi dòng code trace về một scenario*, và phân biệt hai tín hiệu test độc lập: **`dev_selftest`** (dev tự kiểm) vs **`qc_status`** (QC chính thức).

## Mục lục

- [Artifact chain & @trace tags](#artifact-chain--trace-tags)
- [Trace TSV — coverage & drift](#trace-tsv--coverage--drift)
- [Hai tín hiệu test: dev_selftest vs qc_status](#hai-tín-hiệu-test-dev_selftest-vs-qc_status)
- [Living Docs — canonical trong spec-module + panel mirror](#living-docs--canonical-trong-spec-module--panel-mirror)
- [/validate-traces](#validate-traces)

> Schema TSV đầy đủ (mọi cột) và full danh sách trace tag: xem [../05-reference/trace-schema.md](../05-reference/trace-schema.md). Trang này tập trung vào **khái niệm** và cách hai tín hiệu hoạt động.

---

## Artifact chain & @trace tags

Mỗi artifact liên kết về artifact trước nó bằng metadata (PRD: bảng **Metadata**; BDD / tech-doc / code / test: `@trace.*` tags):

```
product-definition.md
  └─► PRD.md (Domain, Status, Service/Module — đều trong bảng Metadata)
        └─► specs/{domain}/{prd-slug}/bdd/{web|app|system}/{UC-ID}.feature (@trace.prd_version, @trace.module)
              └─► specs/{domain}/{prd-slug}/tech-docs/{UC-ID}-tech-design*.md (@trace.bdd_version)
                    └─► src/ code — service submodule (@trace.implements)
                          └─► src/test/ (@trace.verifies, @trace.service)
   ════► {spec_source}/.trace/{domain}/{prd-slug}/{UC-ID}.tsv — TSV authoritative ở SPEC repo (coverage/drift)
```

**Các trace field quan trọng:**

| Field | Vị trí | Ý nghĩa |
|-------|--------|---------|
| `Domain` | bảng Metadata PRD | Domain của feature (auth, payment, …) — route vào đúng service submodule |
| `Status` | bảng Metadata PRD | `draft` / `approved` — dev team chỉ code khi `approved` |
| `@trace.status` | BDD `.feature` header | `draft` / `approved` — người đặt `approved` sau khi review-context BDD sạch; mirror vào `uc_status` |
| `uc_status` | trace TSV (mirror) | gương của BDD `@trace.status` — `/validate-traces` đồng bộ → dashboard `approved_ucs` |
| `@trace.service` / `@trace.module` | BDD / Tech Doc header | Service + module sẽ implement |
| `@trace.prd_version` / `@trace.bdd_version` | BDD / code / test | Version của spec mà artifact được sinh từ — base cho drift detection |
| `@trace.implements` | Code comment | Scenario mà code này implement: `={UC-ID}-SC{N}` |
| `@trace.verifies` | Test / QC test | Scenario mà test này verify: `={UC-ID}` hoặc `={UC-ID}-SC{N}` |
| `@trace.api_source: existing` | BDD header | Brownfield — API đã tồn tại, contract lấy từ PRD |

Ví dụ tags trong `.feature`, code, và test:

```gherkin
# @trace.id: FEAT-001-UC1
# @trace.service: web-admin
# @trace.module: react
# @trace.prd_version: 1.2
# @trace.bdd_version: 3
```
```java
// @trace.implements=FEAT-001-UC1-SC2   // @trace.verifies=FEAT-001-UC1
// @trace.prd_version=1.2               // @trace.service=api-backend
// @trace.bdd_version=3                 // @trace.test_type=integration
// @trace.tech_doc_revision=2
```

---

## Trace TSV — coverage & drift

`.trace/{domain}/{prd-slug}/{UC-ID}.tsv` là nguồn sự thật per-scenario về trạng thái implement/test. Được **ghi** bởi `/generate-bdd`, `/generate-code`, `/dev-gen-test`, `/qc-run-test`, và **update** bởi `/validate-traces`. Khi `spec_source` set, file `.trace/**/*.tsv` authoritative **được commit ở một chỗ** trong spec repo (`{spec_source}/.trace/`) — một nơi cho PM quản lý; mỗi scenario mang `@trace.service`. (Không có `spec_source` → `.trace` per-service.)

Status coverage per scenario:

| Status | Meaning |
|--------|---------|
| ✅ OK | Code version khớp spec version |
| ⚠️ DRIFT | Code gen từ PRD/BDD cũ hơn — re-generate |
| 🔴 GAP | Scenario có trong spec nhưng không tìm thấy code implement |
| — UNTRACKED | Scenario ghi nhận nhưng chưa code-gen |

---

## Hai tín hiệu test: dev_selftest vs qc_status

Trace TSV mang **hai cột độc lập**, cả hai **orthogonal** với cột `status` (coverage) ở trên:

| Cột | Set bởi | Giá trị | Ý nghĩa |
|-----|---------|---------|---------|
| `dev_selftest` (+ `dev_selftest_at`) | `/dev-run-test` | `pass` / `fail` / `not_run` | **Dev self-check** — dev tự chạy bộ self-check test của mình trên scenario đó. **KHÔNG** phải coverage QC chính thức. |
| `qc_status` (+ `qc_run_at`) | `/qc-run-test` | `pass` / `fail` / `skip` / `not_run` | **QC chính thức** — kết quả native QC pipeline (do QC chạy, không phải dev), keyed theo `@trace.verifies={UC-ID}-SC{N}`. |

- **`dev_selftest`** là tín hiệu để QC **thấy** rằng developer đã smoke/self-check qua scenario đó. `dev_selftest: pass` chỉ nghĩa "dev đã tự kiểm" — không phải tuyên bố coverage.
- **`qc_status`** là coverage QC chính thức từ pipeline `/qc-analyze → … → /qc-run-test → /qc-report` (xem [pipeline.md](pipeline.md#qc-automation-pipeline-phase-5b)). `/qc-run-test` dùng stack module `qc-playwright`, độc lập với dev implementation module.
- Hai tín hiệu **sit side by side** trong Living Docs để dễ phân biệt hai luồng. Tester vẫn nên verify độc lập theo BDD + PRD ở phần chưa được automation cover.

> Phân biệt rõ: `/dev-*` (dev self-check, `dev_selftest`) ≠ `/qc-*` (official QC suite, `qc_status`). Đừng coi output của `/dev-*` là coverage chính thức.

---

## Living Docs — canonical trong spec-module + panel mirror

Living Docs (VS Code panel) đọc trace TSV và hiển thị traceability health toàn dự án:

```
┌──────────────────────────────────────────────────────────────────┐
│ PRDs  Use Cases  Scenarios  Code Cov.  Test Cov.  Drift  Gap     │
│  19     86        1077        93%        89%       212    93      │
└──────────────────────────────────────────────────────────────────┘
```

- Drill down: PRD → UC → per-scenario table (Spec ver, Gen ver, Code, Tests, **Dev Self-Check**, **QC Status**, **Waiting on**, Status)
- **Waiting on** (cột `qc_owner` + `qc_blocked_by`): cho PO/PM thấy case chưa pass đang **chờ dev** (product-gap → `BUG-{id}`) hay **chờ PO** confirm/clarify (spec gap → `GAP-{id}`). Aggregates `waiting_dev` / `waiting_po` ở header dashboard.
- Status badges: ✅ OK · ⚠️ DRIFT · 🔴 GAP · — UNTRACKED · filter theo domain/PRD status/doc status · search theo UC/SC ID.
- Số **UC approved** (`approved_ucs`) trên dashboard = số UC có BDD `@trace.status: approved`, đồng bộ từ `.feature` qua `/validate-traces` (xem `uc_status` ở bảng trace field trên).

**Ba nơi chứa trace data — phân biệt authoritative vs mirror:**

| Vị trí | Vai trò | Commit? |
|--------|---------|---------|
| `{spec_source}/.trace/*.tsv` | **Authoritative** — nguồn sự thật, một chỗ cho cả team/PM (mỗi row mang `@trace.service`) | ✅ Committed trong spec repo |
| `{spec_source}/.living-docs/trace-report.json` | **Report** — dashboard tổng hợp, regenerate bởi `/sync` hoặc `/validate-traces` | ❌ Gitignored |
| `./.trace` của workspace hiện tại | **Panel mirror** — giữ Living Docs panel không trống khi mở Claude Code/VS Code ở một repo không phải spec repo | ❌ Gitignored |

**Vì sao authoritative + report nằm ở spec module?** Spec module được mount vào mọi umbrella/service workspace, nên là nơi chung luôn resolve được cho cross-team dashboard, và là **một chỗ duy nhất** để PM/PO quản lý trạng thái. Code-side commands (`/generate-code`, `/dev-run-test`, `/qc-run-test`) chạy trong service nhưng ghi trace cross-repo vào đây (giống `feedback/`). Panel mirror `./.trace` chỉ để panel không trống khi mở một repo lẻ.

Thêm vào `.gitignore` (report + mirror là read-only generated, không commit; `.trace/*.tsv` thì commit trong spec repo):
```
# spec module
.living-docs/
# workspace không phải spec repo (panel mirror)
.trace/
```

> **Prerequisite cho data chính xác:** umbrella cần `setup.spec_source` trỏ đúng spec submodule → framework ghi mọi trace TSV vào `{spec_source}/.trace`. (Chế độ không có `spec_source` mới cần `paths.trace_dir` per-service.)

---

## /validate-traces

Chạy `/validate-traces {domain}` bất cứ lúc nào (đặc biệt **sau mỗi codegen session trong umbrella mode**) để:

```
/validate-traces {domain}
→ Reads .trace/*.tsv authoritative (committed) từ MỘT chỗ: {spec_source}/.trace/
  (mỗi scenario mang @trace.service — không cần quét/merge từng service)
→ Writes trace-report.json → {spec_source}/.living-docs/ (generated, gitignored)
→ Writes panel mirror → ./.trace của workspace hiện tại (non-empty khi mở repo lẻ)
→ Living Docs panel cập nhật ngay
```

Báo cáo: scenario nào chưa có code (GAP) · code nào gen từ PRD/BDD cũ (DRIFT) · broken links / orphan BDD (không có PRD) / dead code traces · coverage % per domain.

**Khi nào chạy:** sau refactor đổi tên file/function · sau khi PO cập nhật PRD (version mới) · trước PR lớn · khi CI báo trace validation fail · sau mỗi codegen session (umbrella).

> **Nguyên tắc:** không merge code khi traces broken — fix traces trước. `/sync` cũng regenerate Living Docs như một bước phụ; `/validate-traces` là lệnh chuyên trách kiểm tra trace chain.
