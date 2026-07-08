[📚 Docs](../../README.md) › [Guides](../README.md) › [Tester](README.md) › Workflow

# Workflow Cơ Bản

```
Nhận task: "Test FT-042 — Checkout flow"
        │
        ▼
git pull && git submodule update --remote --recursive
        │
        ▼
/generate-spec-manifest
→ spec-manifest.yaml được refresh
        │
        ▼
Lookup FT-042 trong spec-manifest.yaml
→ status: approved? ✅ (nếu draft → dừng, báo PO)
→ ghi lại paths (tất cả đều nằm trong submodule):
    prd:          {spec_source}/specs/{domain}/{prd-slug}/{TICKET-ID}-{prd-slug}.md
    bdd.system:   {spec_source}/specs/{domain}/{prd-slug}/bdd/system/FT-042-*.feature
    bdd.web:      {spec_source}/specs/{domain}/{prd-slug}/bdd/web/FT-042-*.feature
    tech_docs.be: {spec_source}/specs/{domain}/{prd-slug}/tech-docs/FT-042-*.md
    tech_docs.fe: {spec_source}/specs/{domain}/{prd-slug}/tech-docs/FT-042-*-tech-design-web.md
        │
        ▼
Đọc PRD tại path manifest.prd
→ file nằm trong SPEC submodule (shared — PO viết)
→ hiểu AC, UC, BR — đây là "ground truth"
        │
        ▼
Đọc BDD tại path manifest.bdd.be / bdd.web
→ file nằm trong SPEC submodule (PO gen từ PRD — tất cả BDD web/app/system ở spec repo)
→ mỗi Scenario = 1 test case cần cover
→ BE có BDD riêng (system/), Web có BDD riêng (web/), App có BDD riêng (app/)
        │
        ▼
Đọc Tech Docs tại path manifest.tech_docs.be / tech_docs.web
→ file nằm trong SPEC submodule (shared — Dev gen, SA sign-off)
→ BE: API endpoints, request/response schema, error codes
→ Web: screen flow, component states
        │
        ▼
Chạy QC automation pipeline (6 bước — ghi kết quả chính thức vào TSV)
        │
        ├─ /qc-analyze {UC-ID}       → 2 file: REQUIREMENT_ANALYSIS.md + DOC_GAPS.md
        ├─ /qc-plan {UC-ID}          → TEST_PLAN.md (risk + plan + questions-for-dev)
        ├─ /qc-design-test {UC-ID}   → test-cases/*.Test.md
        │     ⌙ artifact phân tích/thiết kế → {qc_dir}/{UC-ID}/ (mặc định docs/, VISIBLE)
        ├─ /qc-review {UC-ID}        → review test design trước khi chạy
        ├─ /qc-run-test {UC-ID}      → sinh + chạy pytest-playwright
        │                               → ghi qc_status (pass/fail/skip) per scenario
        │                               → vào {spec_source}/.trace/{domain}/{prd-slug}/{UC-ID}-{platform}.tsv  ← AUTHORITATIVE (spec repo)
        └─ /qc-report {UC-ID}        → report pytest-html + Playwright trace evidence
                                        → reports/<feature>/report.html  ← LOCAL, gitignored
        │
        ▼
Commit TSV vào spec repo (1 tầng — trace dồn về specs, giống feedback/)
  cd {spec_source}
  git add .trace/{domain}/{prd-slug}/{UC-ID}-{platform}.tsv
  git commit -m "qc: record qc_status for {UC-ID} — pass/fail"
  git push
        │
        ▼
/validate-traces (hoặc /sync)
→ đọc {spec_source}/.trace/ (một chỗ) → Living Docs cập nhật cột qc_status
        │
        ▼
Kết quả:
  qc_status: pass  → done, Living Docs phản ánh coverage chính thức
  qc_status: fail  → /report-bug với BDD scenario + AC bị vi phạm
                      đính kèm path evidence: reports/<feature>/report.html
                      (evidence local — share file hoặc upload riêng nếu cần)
```

---

← [Spec Manifest & Setup](spec-manifest.md) · Tiếp theo: [Đọc Spec Chain](reading-specs.md)
