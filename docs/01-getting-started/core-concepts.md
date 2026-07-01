[📚 Docs](../README.md) › [Getting Started](README.md) › Core Concepts

# Core Concepts

Các khái niệm cốt lõi trong 5 phút. Đào sâu hơn ở [../03-concepts](../03-concepts).

## Mục lục

- [Spec là source of truth](#spec-là-source-of-truth)
- [Pipeline phases](#pipeline-phases)
- [Traceability](#traceability)
- [dev_selftest vs qc_status](#dev_selftest-vs-qc_status)
- [Umbrella & spec module](#umbrella--spec-module)

## Spec là source of truth

> **Write the spec first. Generate the code from the spec. Trace everything.**

- Con người định nghĩa *WHAT* — acceptance criteria, business rules, platform requirements.
- AI sinh ra *HOW* — BDD scenarios, tech design, code, tests — thích ứng theo platform.
- Mỗi artifact được review (AI tìm findings) rồi **người duyệt** (đặt `Status` / `@trace.status: approved`) trước khi sang phase kế tiếp; sửa nội dung sau khi duyệt → tự về `draft`. Lệnh tiêu thụ **cảnh báo mềm** nếu nguồn chưa approved.
- Mỗi dòng code truy ngược về một scenario trong file `.feature`.

**PRD platform-agnostic (Option C):** một PRD phục vụ mọi platform — chỉ mô tả nghiệp vụ, không có chi tiết UI hay API. FE/App đọc PRD → Design Spec → BDD; BE đọc PRD trực tiếp → BDD. Thêm platform mới sau không cần sửa PRD.

## Pipeline phases

```
Discovery → PRD → BDD Spec → Tech Design → Code → Dev Self-Check
                                                       (+ QC suite chính thức song song)
```

| Phase | Ai | Lệnh chính | Output |
|-------|-----|-----------|--------|
| Discovery | PO + AI | `/define-product` | `specs/product-definition/{slug}.md` |
| PRD | AI → SA/PO | `/generate-prd`, `/refine-prd`, `/review-context` | `specs/{domain}/{prd-slug}/{TICKET-ID}-{prd-slug}.md` |
| Design Spec *(FE/App)* | AI → Designer/PO | `/generate-design-spec` | `specs/{domain}/{prd-slug}/design-spec/...` |
| BDD Spec | AI → SA/Dev | `/generate-bdd`, `/review-context` | `specs/{domain}/{prd-slug}/bdd/{UC-ID}.feature` |
| Tech Design | AI → SA/Lead | `/generate-tech-docs`, `/review-tech-docs` | `tech-docs/{domain}/{UC-ID}-tech-design.md` |
| Code | AI → Dev | `/generate-code`, `/review-code` | `src/...` |
| Dev Self-Check | Dev | `/dev-gen-test`, `/dev-run-test`, `/dev-smoke-test` | `src/test/...` |
| QC Automation | QC | `/qc-analyze → /qc-plan → /qc-design-test → /qc-review → /qc-run-test → /qc-report` | QC designs + run results |
| Trace audit | Tech Lead | `/validate-traces {domain}` | Coverage + drift report |

Chi tiết từng phase + happy-path lệnh: [quickstart.md](quickstart.md).

## Traceability

Mỗi artifact link tới artifact khác qua `@trace.*` tags:

```
product-definition.md
  └─► PRD.md (Domain, Status, Service, Module — bảng Metadata)
        └─► specs/{domain}/{prd-slug}/bdd/{web|app|system}/{UC-ID}.feature (@trace.prd_version)
              └─► specs/{domain}/{prd-slug}/tech-docs/{UC-ID}-tech-design*.md (@trace.bdd_version)
                    └─► src/ code — service submodule (@trace.implements)
                          └─► src/test/ (@trace.verifies)
   ════► {spec_source}/.trace/{domain}/{prd-slug}/{UC-ID}.tsv — authoritative ở SPEC repo (drift tracking)
```

`/validate-traces {domain}` đọc các `.trace/{domain}/{prd-slug}/{UC-ID}.tsv` và báo cáo:

| Status | Nghĩa |
|--------|-------|
| ✅ OK | Code version khớp spec version |
| ⚠️ DRIFT | Code sinh từ PRD/BDD version cũ — cần re-generate |
| 🔴 GAP | Scenario có trong spec nhưng chưa có code implement |
| — UNTRACKED | Scenario ghi nhận nhưng chưa code-gen |

Chi tiết tags + chain: [../03-concepts/traceability.md](../03-concepts/traceability.md).

## dev_selftest vs qc_status

Hai signal **độc lập** trong trace TSV — đừng nhầm lẫn:

| Signal | Set bởi | Ý nghĩa |
|--------|---------|---------|
| `dev_selftest` (+ `dev_selftest_at`) | `/dev-run-test` | Dev tự verify code của mình (smoke / self-check). **KHÔNG phải** official coverage. |
| `qc_status` (+ `qc_run_at`) | `/qc-run-test` | Bộ QC suite chính thức (native `/qc-*` pipeline), keyed theo `@trace.verifies={UC-ID}-SC{N}`. |

QC owns `qc_status`; dev owns `dev_selftest`. Living Docs panel hiển thị cả hai cạnh nhau. `/qc-run-test` & `/qc-report` dùng stack `qc-playwright` (độc lập với dev module — xem [installation.md](installation.md)).

## Umbrella & spec module

Cho project có nhiều service repo riêng (microservices, multi-platform): pattern khuyến nghị là **umbrella repo** — repo tổng chứa các service repo dưới dạng git submodule.

```
my-project-umbrella/          ← mở Claude Code ở đây
├── .agent/project-context.yaml   ← routing config (domain → service)
├── my-project-specs/         ← submodule: spec module của PO (PRD, design-spec, tech-docs)
├── user-service/             ← submodule: microservice
└── web-app/                  ← submodule: FE app
```

- **Spec module** (`{spec_source}/`): chứa PRD, Design Spec, **ALL BDD (web/app/system)**, tech-docs (BE API contract + FE tech-design), domain-knowledge, feedback — shared để mọi umbrella đọc cùng nguồn. Report canonical của Living Docs cũng sinh tại `{spec_source}/.living-docs/`.
- **Routing:** context-loader đọc row `Domain` (bảng Metadata) từ PRD → tra trong `services` config → route **code** vào đúng service submodule; **BDD + tech-docs + specs + `.trace/` + feedback** đều ở spec module (cross-team, một chỗ cho PM). PRD phải có row `Domain` khớp một key trong `services`.

Setup umbrella đầy đủ, file ownership, two-layer commit, multi-platform → [Operations › Sync & Update §4 Umbrella mode](../04-operations/sync-and-update.md#4-umbrella-mode--git-submodule).

---

Sẵn sàng chạy? → [quickstart.md](quickstart.md).
