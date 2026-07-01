[📚 Docs](../README.md) › Guides

# Guides — Role Playbooks

Mỗi role trong framework spec-driven-docs có một guide riêng: vai trò, commands, workflow, và các tình huống thực tế. Bắt đầu từ guide khớp với role của bạn.

## Mục Lục

| Guide | Dành cho | Nội dung |
|---|---|---|
| [Product Owner / BA](product-owner/README.md) | PO, BA | Viết PRD platform-agnostic, generate Design Spec + BDD, handoff cho dev team |
| [Developer (FE / BE / App)](developer/README.md) | Dev | Đọc PRD + BDD từ spec submodule, generate tech-docs + code, dev self-check, trace system |
| [Tester / QA](tester/README.md) | QA / Tester / QC | Spec-manifest, đọc spec chain, viết test cases, `/report-bug` + `/propose-scenario`, và chương **[QC Automation](tester/qc-automation.md)** (pipeline `/qc-*` 6 bước, `qc_status`, stack `qc-playwright`) — **một role duy nhất** |
| Designer | Designer, UX | Tham gia giai đoạn Design Spec — sign-off màn hình + component trước khi PO gen BDD. Chưa có guide riêng; xem [Product Owner › Design Spec](product-owner/scenarios.md#tình-huống-3--tạo-design-spec-và-bdd-sau-khi-prd-approved) để biết điểm giao. |

## Checklists đầu vào — gen chuẩn ngay lần đầu

Chuẩn bị đúng input trước khi chạy lệnh gen, thay vì trông vào review-vá về sau:

| Checklist | Áp cho | Mục tiêu |
|---|---|---|
| [Input PRD](prd-input-checklist.md) | `/generate-prd` | PRD chuẩn ngay lần đầu, không phải `/refine-prd` nhiều vòng |
| [Input BDD (System/BE)](bdd-input-checklist.md) | `/generate-bdd` platform `system` | BDD BE chuẩn, không sinh lại |
| [Input Tech-Docs (BE)](tech-docs-input-checklist.md) | `/generate-tech-docs` trên System BDD | API contract chuẩn để FE không phải chờ sửa |

## Phân biệt nhanh hai luồng test

- **Dev self-check** — `/dev-gen-test` · `/dev-run-test` · `/dev-smoke-test`, ghi `dev_selftest`. Smoke check của riêng dev, KHÔNG phải coverage chính thức.
- **QC chính thức** — pipeline `/qc-analyze … /qc-report`, ghi `qc_status`. Bộ test authoritative, do QC chạy.

Hai tín hiệu này đứng cạnh nhau trong Living Docs và không ghi đè nhau.

## Xem thêm

- [Concepts › Traceability](../03-concepts/traceability.md) — trace chain PRD → BDD → Code
- [Reference › Commands](../05-reference/commands.md) — danh mục đầy đủ mọi command
