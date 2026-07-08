# 📚 Spec-Driven Docs — Documentation

> AI-First Spec-Driven Development for Claude Code: Discovery → PRD → Design-Spec → BDD →
> Code → Dev self-check → QC automation, with traceability and human review gates.

Tài liệu được chia theo **vai trò** và **chủ đề**. Bắt đầu ở mục phù hợp với bạn:

| Bạn là… | Bắt đầu ở |
|---------|-----------|
| Người mới / cài đặt lần đầu | [01 · Getting Started](01-getting-started/) |
| Product Owner | [Guide · Product Owner](02-guides/product-owner/README.md) |
| Developer | [Guide · Developer](02-guides/developer/README.md) |
| QC / Tester (QA) | [Guide · Tester / QA](02-guides/tester/README.md) — gồm cả pipeline QC tự động (`/qc-*`) |
| Muốn hiểu kiến trúc | [03 · Concepts](03-concepts/) |
| Vận hành / admin | [04 · Operations](04-operations/) |
| Tra cứu lệnh | [05 · Reference](05-reference/) |
| Muốn hiểu một lệnh chạy sao (dễ hiểu) | [06 · Commands (Dễ Hiểu)](06-commands/) |

---

## Mục lục

### 01 · [Getting Started](01-getting-started/)
- [Installation](01-getting-started/installation.md) — cài framework, prerequisites
- [Quick Start](01-getting-started/quickstart.md) — chạy feature đầu tiên end-to-end
- [Core Concepts](01-getting-started/core-concepts.md) — khái niệm trong 5 phút

### 02 · [Guides (theo vai trò)](02-guides/)
- [Product Owner](02-guides/product-owner/README.md) — define-product, PRD, design-spec, review
- [Developer](02-guides/developer/README.md) — BDD, tech-docs, code, dev self-check
- [Tester / QA](02-guides/tester/README.md) — spec-manifest, report-bug, propose-scenario, Living Docs, và **chương [QC Automation](02-guides/tester/qc-automation.md)** (pipeline `/qc-*`, `qc_status`)
- Checklists đầu vào (chuẩn bị đúng input để gen chuẩn ngay lần đầu): [PRD](02-guides/prd-input-checklist.md) · [BDD System/BE](02-guides/bdd-input-checklist.md) · [Tech-Docs BE / API contract](02-guides/tech-docs-input-checklist.md)

### 03 · [Concepts](03-concepts/)
- [Architecture](03-concepts/architecture.md) — layers, modules, plug-in system
- [Pipeline](03-concepts/pipeline.md) — các phase Discovery → QC
- [Traceability & Living Docs](03-concepts/traceability.md) — trace TSV, dev_selftest vs qc_status
- [Mechanisms Explained](03-concepts/mechanisms-explained.md) — cơ chế framework giải thích bằng ngôn ngữ dễ hiểu + ví von đời thường

### 04 · [Operations](04-operations/)
- [Sync & Update](04-operations/sync-and-update.md) — `/sync`, `/update-framework`, umbrella
- [Bug Flow](04-operations/bug-flow.md) — report → fix → verify
- [Publishing](04-operations/publishing.md) — build + npm publish

### 05 · [Reference](05-reference/)
- [Commands](05-reference/commands.md) — bảng đầy đủ mọi slash command
- [Trace Schema](05-reference/trace-schema.md) — cột TSV, trace tags
- [Modules](05-reference/modules.md) — stack modules (java-spring, react, …, qc-playwright)

### 06 · [Commands (Dễ Hiểu)](06-commands/)
- Giải thích luồng chạy của **cả 30 slash command** bằng ví von đời thường — mỗi trang một `explain-{command}.md`, gom theo phase pipeline. Xem [mục lục 06](06-commands/README.md).

---

*Mỗi file có breadcrumb điều hướng ở đầu. Phiên bản framework mới nhất: xem trên [npm](https://www.npmjs.com/package/@edupia-tutor/spec-driven-docs).*
