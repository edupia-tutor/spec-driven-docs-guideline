[📚 Docs](../README.md) › [Reference](README.md) › Trace TSV Schema

# Trace TSV Schema

> Schema chuẩn của `.trace/{UC-ID}-{platform}.tsv` — file trace state per **use-case × platform**. Mỗi UC có một sổ riêng cho `system` / `web` / `app`: 1 header row + 1 data row mỗi scenario. Tách theo platform vì `sc_id` = `{UC-ID}-SC{N}` chỉ độc nhất trong (UC × platform) (mỗi platform tự đánh số SC từ 1) — gộp một sổ sẽ khiến scenario các platform đè/xoá nhau. Đây là nguồn dữ liệu cho `/validate-traces` và panel Living Docs.

---

## Mục lục

- [Canonical TSV header](#canonical-tsv-header)
- [Column reference](#column-reference)
- [Two independent signals: dev_selftest vs qc_status](#two-independent-signals-dev_selftest-vs-qc_status)
- [Status computation (validate-traces)](#status-computation-validate-traces)
- [@trace tags across artifacts](#trace-tags-across-artifacts)
- [trace-report.json](#trace-reportjson)

---

## Canonical TSV header

Tab-separated, một header row + một data row mỗi scenario. Header chính xác (từ `generate-bdd.tmpl`):

```
sc_id	sc_title	spec_ver	gen_ver	implemented_by	test_count	test_classes	dev_selftest	dev_selftest_at	qc_status	qc_run_at	qc_owner	qc_blocked_by	prd_version	bdd_version	tech_doc_revision	fe_tech_doc_revision	prd_status	uc_status	fe_phase	status	last_updated
```

22 cột, theo đúng thứ tự trên.

---

## Column reference

| # | Column | Ý nghĩa | Giá trị khởi tạo (`/generate-bdd`) | Ai ghi |
|---|--------|---------|------------------------------------|--------|
| 1 | `sc_id` | `{UC-ID}-SC{N}` — khóa chính của row | `{UC-ID}-SC{N}` | generate-bdd |
| 2 | `sc_title` | Scenario title text | từ `.feature` | generate-bdd |
| 3 | `spec_ver` | `@trace.sc_version` hiện tại của scenario | từ `.feature` | generate-bdd / validate-traces (reconcile) |
| 4 | `gen_ver` | Version tại thời điểm code được sinh | `—` | generate-code |
| 5 | `implemented_by` | `ClassName.method` cài đặt scenario | `—` | generate-code |
| 6 | `test_count` | Số test verify scenario | `—` | dev-gen-test |
| 7 | `test_classes` | Danh sách test class | `—` | dev-gen-test |
| 8 | `dev_selftest` | Kết quả dev self-check: `pass` / `fail` / `not_run` | `—` | **dev-run-test** |
| 9 | `dev_selftest_at` | Ngày chạy dev self-check (`YYYY-MM-DD`) | `—` | **dev-run-test** |
| 10 | `qc_status` | Kết quả QC chính thức: `pass` / `fail` / `skip` / `not_run` | `—` | **qc-run-test** |
| 11 | `qc_run_at` | Ngày chạy QC (`YYYY-MM-DD`) | `—` | **qc-run-test** |
| 12 | `qc_owner` | **Đang chờ ai** (PM view) khi SC chưa pass: `dev` / `po` / `—` | `—` | **qc-run-test** / **report-bug** |
| 13 | `qc_blocked_by` | Artifact liên quan: `BUG-{id}` / `GAP-{id}` / `—` | `—` | **qc-run-test** / **report-bug** |
| 14 | `prd_version` | `@trace.prd_version` từ `.feature` header | từ `.feature` | generate-bdd |
| 15 | `bdd_version` | `@trace.bdd_version` từ `.feature` header | từ `.feature` | generate-bdd |
| 16 | `tech_doc_revision` | Revision tech-doc gộp `{TICKET-ID}-tech-design.md` (§4 backend) tại thời điểm codegen | `—` | generate-code / review-tech-docs |
| 17 | `fe_tech_doc_revision` | Revision **cùng** tech-doc gộp, ghi khi FE `--phase=integration` wire adapter theo §4.5.4 (drift-detect riêng cho FE) | `—` | generate-code `--phase=integration` |
| 18 | `prd_status` | `\| **Status** \|` từ PRD metadata | từ PRD | generate-bdd |
| 19 | `uc_status` | Trạng thái duyệt BDD của UC — **gương của `@trace.status`** (`.feature`): `draft` / `approved` | `draft` (UC mới) | generate-bdd (init) · validate-traces (sync ← `@trace.status`) · review-context (reset draft khi `--fix`/`--resume`) |
| 20 | `fe_phase` | FE phase khi implement (`ui` / `integration`) | `—` | generate-code `--phase` |
| 21 | `status` | Trạng thái tổng hợp: `OK` / `DRIFT` / `GAP` / `UNTRACKED` | `UNTRACKED` | validate-traces |
| 22 | `last_updated` | Ngày update gần nhất (`YYYY-MM-DD`) | today | mọi command ghi row |

> **`qc_owner` / `qc_blocked_by` (PM "waiting-on" view):** dành cho PO kiêm PM xem **case nào đang chờ ai**. `/qc-run-test` set khi ghi `qc_status`: product-gap FAIL → `qc_owner=dev`; `skip`/`not_run` do `DOC_GAPS` 🔴 Blocker mở → `qc_owner=po` + `qc_blocked_by=GAP-{id}`; `pass` → cả hai về `—`. `/report-bug` backfill `qc_blocked_by=BUG-{id}` (+ `qc_owner` theo layer) cho SC khớp. `/validate-traces` tổng hợp `waiting_dev` / `waiting_po` cho dashboard.

> **Re-generation rules (generate-bdd):** SC đã có trong TSV & `spec_ver` không đổi → chỉ update `sc_title`, `prd_version`, `bdd_version`, `prd_status`, `uc_status`, `last_updated`. Nếu `spec_ver` đổi → thêm `spec_ver` và set `status = DRIFT` ngay. SC mới → append với `gen_ver`/`implemented_by`/`test_count`/`test_classes`/`dev_selftest`/`dev_selftest_at`/`qc_status`/`qc_run_at`/`qc_owner`/`qc_blocked_by`/`tech_doc_revision`/`fe_tech_doc_revision` = `—`. SC bị xóa khỏi `.feature` → xóa row.

> **Backward-compat:** TSV cũ thiếu cột mới ở header → `/validate-traces` đọc thành giá trị rỗng, không lỗi: `qc_owner`/`qc_blocked_by` (trước 19 cột) → `null`; `fe_tech_doc_revision` (trước 22 cột) → `0`. Lần `/generate-bdd` re-gen kế tiếp tự nâng header lên 22 cột.

---

## Two independent signals: dev_selftest vs qc_status

Trace TSV mang **hai cột tín hiệu độc lập** — không bao giờ trộn:

| | `dev_selftest` (+ `dev_selftest_at`) | `qc_status` (+ `qc_run_at`) |
|---|--------------------------------------|------------------------------|
| Ghi bởi | `/dev-run-test` | `/qc-run-test` |
| Ý nghĩa | Dev tự smoke-check code mình vừa sinh | Kết quả QC automation **chính thức** |
| Giá trị | `pass` / `fail` / `not_run` | `pass` / `fail` / `skip` / `not_run` |
| Stack | Dev implementation module | `qc-playwright` module |
| Coverage chính thức? | **KHÔNG** — chỉ là dev self-check | **CÓ** — official QC result |
| Keyed by | `@trace.verifies={UC-ID}` | `@trace.verifies={UC-ID}-SC{N}` |

`/validate-traces` chỉ **đọc** hai cột này cho report — không bao giờ ghi đè (mỗi cột do command sở hữu nó quản lý). Living Docs hiển thị cả hai cạnh nhau.

---

## Status computation (validate-traces)

`/validate-traces` tính cột `status` theo thứ tự ưu tiên (rule đầu tiên khớp thì dừng):

| Rule | Status | Điều kiện |
|------|--------|-----------|
| 1 | `UNTRACKED` | `implemented_by == —` (chưa sinh code) |
| 2 | `GAP` | `implemented_by != —` AND (`test_count == —` OR `test_count == 0`) |
| 3 | `DRIFT` | `spec_ver != gen_ver` (scenario đổi sau lần codegen gần nhất) |
| 4 | `OK` | tất cả: `spec_ver == gen_ver`, `implemented_by != —`, `test_count > 0` |

Ngoài ra `/validate-traces` còn flag **`PRD_DRIFT`** (PRD version trong code/TSV trễ hơn PRD file hiện tại), **`TECHDOC_DRIFT`** (code BE sinh từ revision cũ của tech-doc gộp — so `tech_doc_revision`), và **`FE_TECHDOC_DRIFT`** (FE integration wire theo §4.5.4 ở revision cũ của cùng doc — so `fe_tech_doc_revision`).

---

## @trace tags across artifacts

Mọi artifact link với nhau qua `@trace.*` tags.

**`.feature` files:**
```gherkin
# @trace.id: FEAT-001-UC1
# @trace.service: web-admin
# @trace.module: react
# @trace.status: draft          # draft/approved — người đặt approved sau review-context BDD sạch; mirror → uc_status
# @trace.prd_version: 1.2
# @trace.bdd_version: 3
# @trace.api_source: existing   # brownfield: API đã tồn tại, contract lấy từ PRD
```
Per-scenario trong file: `# @trace.scenario: {UC-ID}-SC{N}`, `# @trace.sc_version: 1.0`, `# @trace.business_rules: ...`.

**Code:**
```java
// @trace.implements=FEAT-001-UC1-SC2
// @trace.prd_version=1.2
// @trace.bdd_version=3
// @trace.tech_doc_revision=2
```

**Dev self-check tests:**
```java
// @trace.verifies=FEAT-001-UC1
// @trace.service=api-backend
// @trace.test_type=integration
```

**QC tests** (`qc-playwright`, drive `qc_status`):
```python
# @trace.verifies={UC-ID}-SC{N}
# @trace.source=<official .feature path>
# @trace.test_type=functional|integration|e2e|non-functional
```

---

## trace-report.json

`/validate-traces` ghi `{paths.trace_dir}/trace-report.json` — single source of truth cho web dashboards. Cấu trúc rút gọn:

- `generated_at`, `domain`
- `summary` — aggregates: `total_prds`, `approved_prds`, `total_ucs`, `approved_ucs`, `draft_ucs`, `total_scs`, `coded_scs`, `tested_scs`, `code_coverage_pct`, `test_coverage_pct`, `drift_count`, `gap_count`, `untracked_count`, `dev_selftest_passing/failing/not_run`, `qc_passing/failing/skipped/not_run`, `waiting_dev`, `waiting_po`, `tech_docs_count`
- `prds[]` → `ucs[]` → `scenarios[]` — mỗi scenario có đầy đủ các trường của TSV row (gồm `dev_selftest`, `dev_selftest_at`, `qc_status`, `qc_run_at`)
- `issues` — phân loại: `drift`, `gap`, `untracked`, `prd_version_drift`, `techdoc_drift`, `fe_techdoc_drift` (kèm `platform`), mỗi entry kèm `fix` command gợi ý

**TSV `"—"` → JSON mapping:** `implemented_by: "—"` → `null` · `test_count: "—"` → `0` · `test_classes: "—"` → `[]` · `tech_doc_revision: "—"` → `0` · `fe_tech_doc_revision: "—"` → `0` · `dev_selftest: "—"` → `"not_run"` · `dev_selftest_at: "—"` → `null` · `qc_status: "—"` → `"not_run"` · `qc_run_at: "—"` → `null`.

> **Umbrella mode:** khi `spec_source` set, `.trace/*.tsv` authoritative nằm **một chỗ** ở `{spec_source}/.trace/` (committed — PM quản lý tập trung; mỗi scenario mang `@trace.service`). `/validate-traces` (hoặc `/sync`) sinh `trace-report.json` vào `{spec_source}/.living-docs/` + mirror `./.trace/trace-report.json` ở workspace hiện tại — cả hai generated, gitignore. (Không có `spec_source` → `.trace` per-service.)

---

*Xem thêm:* [Command Reference](commands.md) (`/validate-traces`, `/qc-run-test`, `/dev-run-test`) · [03 · Concepts](../03-concepts/) (traceability system).
