[📚 Docs](../../README.md) › [Guides](../README.md) › [Tester](README.md) › Spec Manifest & Setup

# Spec Manifest & Setup

## Spec-Manifest Là Gì Và Tại Sao Cần

Trong umbrella repo (có `spec_source`), **mọi spec đều nằm trong spec submodule** — gom theo feature-package: mỗi PRD có một thư mục `specs/{domain}/{prd-slug}/` chứa tất cả artifact (`{TICKET-ID}-{prd-slug}.md`, `bdd/{platform}/`, `tech-docs/`, `design-spec/`), trải trên nhiều domain:

```
free-trial-umbrella/                              ← umbrella (Claude Code mở ở đây)
├── free-trial-specs/                             ← SPEC submodule — TẤT CẢ spec ở đây
│   └── specs/
│       └── auth/FT-001-login/                    ← feature-package (1 PRD = 1 thư mục)
│           ├── {TICKET-ID}-{prd-slug}.md                            ← PRD
│           ├── design-spec/...                   ← Design Spec
│           ├── bdd/system/...                    ← BDD System (BE)
│           ├── bdd/web/...                        ← BDD Web (FE)
│           ├── bdd/app/...                        ← BDD App
│           └── tech-docs/...                     ← Tech Docs (BE contract + FE design)
├── free-trial-be/    (chỉ code)                  ← SERVICE submodule
└── free-trial-web/   (chỉ code)                  ← SERVICE submodule
```

Tester's agent không biết domain nào / file nào ứng với TICKET-ID → cần một **index file** để tra cứu. *(Chỉ khi KHÔNG có `spec_source` thì BDD mới rải theo từng service submodule.)*

`spec-manifest.yaml` là file được gen tự động, ánh xạ TICKET-ID → tất cả file liên quan:

```yaml
features:
  FT-001:
    domain: auth
    status: approved
    prd: "specs/auth/FT-001-login/FT-001-login.md"
    pdd: "specs/product-definition/auth/FT-001-def.md"
    tech_docs:
      be:  "free-trial-specs/specs/auth/FT-001-login/tech-docs/FT-001-UC1-auth-api.md"
      web: "free-trial-specs/specs/auth/FT-001-login/tech-docs/FT-001-UC1-login-web.md"
    bdd:
      be:  "free-trial-specs/specs/auth/FT-001-login/bdd/system/FT-001-UC1-login-system.feature"
      web: "free-trial-specs/specs/auth/FT-001-login/bdd/web/FT-001-UC1-login-web.feature"
```

**Quan trọng:**
- File này **không commit vào git** (gitignored) — gen local mỗi khi cần
- Luôn `git pull` trước khi gen để có submodule mới nhất
- Chỉ test các feature có `status: approved` — draft PRD chưa được approve
- Nếu nhiều người gen cùng lúc → kết quả **giống nhau** (deterministic) — không conflict
- Khi PO cập nhật PRD hoặc BDD → re-run `/generate-spec-manifest` để manifest phản ánh nội dung mới

## Setup Tester Agent

### Lần đầu tiên

```bash
# 1. Clone umbrella repo (nếu chưa có)
git clone {umbrella-repo-url} free-trial-spec
cd free-trial-spec

# 2. Init tất cả submodules
git submodule update --init --recursive

# 3. Mở Claude Code tại umbrella root
#    QUAN TRỌNG: mở tại free-trial-spec/, không phải free-trial-tests/
```

### Trước mỗi test sprint

```bash
# Cập nhật tất cả submodules về commit mới nhất
git pull
git submodule update --remote --recursive

# Gen manifest
/generate-spec-manifest
# → viết spec-manifest.yaml tại free-trial-spec/ root
```

### Cấu hình tester agent đọc manifest

Trong tester's agent config (hoặc system prompt), thêm:

```
Trước khi test bất kỳ feature nào:
1. Đọc spec-manifest.yaml tại project root
2. Lookup TICKET-ID → lấy paths của prd, bdd, tech_docs
3. Chỉ test features có status: approved
4. Đọc PRD trước để hiểu business context
5. Đọc BDD để biết scenarios cần cover
```

## Living Docs Panel (umbrella mode)

Khi làm việc với umbrella repo, VS Code Living Docs panel cần được đồng bộ trước khi xem (chạy `/sync` hoặc `/validate-traces`).

**Report canonical (Living Docs):** `{spec_source}/.living-docs/trace-report.json` + bản mirror TSV — nằm trong **spec module** (gitignored, regenerate mỗi lần `/sync` hoặc `/validate-traces`).

**Panel mirror:** `./.trace` của workspace hiện tại — giữ panel **không bị trống** kể cả khi mở Claude Code bên trong một service submodule đơn lẻ.

```bash
# Chạy sau mỗi codegen session — hoặc khi cần refresh trạng thái coverage
/sync            # hoặc /validate-traces

# → Đọc TSVs từ MỘT chỗ: {spec_source}/.trace/  (mỗi scenario mang @trace.service)
# → Ghi canonical: {spec_source}/.living-docs/trace-report.json (+ TSV mirror)
# → Ghi panel mirror: ./.trace  (workspace hiện tại — panel luôn có data)
# → Panel cập nhật ngay sau khi lệnh hoàn tất
```

Panel hiển thị trạng thái cross-service:

| Cột | Ý nghĩa |
|-----|---------|
| PRDs | Số PRD đã approve / tổng |
| UCs | Số Use Cases đã implement |
| Code Cov. | % scenarios có code |
| Test Cov. | % scenarios có test |
| DRIFT | Spec thay đổi sau khi gen code |
| GAP | Code có nhưng chưa có test |
| dev_selftest | Dev đã chạy self-check: `pass` / `fail` / `not_run` |
| qc_status | Kết quả QC chính thức: `pass` / `fail` / `skip` / `not_run` |

> **`dev_selftest`:** tín hiệu cho QC thấy dev đã tự chạy self-check. **Không phải** coverage chính thức — đó là `qc_status`.
>
> **`qc_status`:** kết quả pipeline QC chính thức (`/qc-run-test`). Dashboard hiển thị hai cột cạnh nhau để phân biệt rõ hai luồng.

> **Prerequisite:** umbrella cần `setup.spec_source` trỏ đúng spec submodule → trace TSV authoritative dồn về `{spec_source}/.trace/`. Nếu thiếu → panel trống → báo Dev team. (Chỉ chế độ không có `spec_source` mới cần `paths.trace_dir` per-service.)

---

← [Tester Guide](README.md) · Tiếp theo: [Workflow](workflow.md)
