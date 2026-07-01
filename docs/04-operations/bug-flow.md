[📚 Docs](../README.md) › [Operations](README.md) › Bug Flow

# Bug Flow — PO · Dev · QC/Tester

> Cách **PO, Dev, và QC/Tester phối hợp** khi phát hiện bug. Mọi bug đều được trace về spec layer để fix đúng chỗ và tránh lặp lại.

> **QC cũng là nguồn bug.** Pipeline `/qc-*` đẩy vào **cùng** flow này: `/qc-run-test` FAIL = `product-gap` → `/report-bug {UC-ID}`; `/qc-analyze` `DOC_GAPS` blocker (spec sai/mơ hồ) → `/report-bug` (Case 2/3) hoặc thiếu coverage → `/propose-scenario`. Cùng phân loại Case 1–6 bên dưới. Chi tiết: [chương QC Automation](../02-guides/tester/qc-automation.md#khi-qc-tìm-thấy-bug--spec-gap--đẩy-lên-specs).

---

## Mục lục

1. [Bug thuộc layer nào?](#1-bug-thuộc-layer-nào)
2. [Bảng quyết định nhanh](#2-bảng-quyết-định-nhanh)
3. [Case 1 — Code Bug (Code ≠ BDD)](#3-case-1--code-bug-code--bdd)
4. [Case 2 — BDD Bug (BDD ≠ PRD)](#4-case-2--bdd-bug-bdd--prd)
5. [Case 3 — PRD Ambiguity (PRD không rõ)](#5-case-3--prd-ambiguity-prd-không-rõ)
6. [Case 4 — PRD Change (yêu cầu thay đổi)](#6-case-4--prd-change-yêu-cầu-thay-đổi)
7. [Case 5 — Design Spec Bug (UI ≠ Design Spec)](#7-case-5--design-spec-bug-ui--design-spec)
8. [Case 6 — Environment / Data Bug](#8-case-6--environment--data-bug)
9. [Giao tiếp hiệu quả — các format thông báo](#9-giao-tiếp-hiệu-quả--các-format-thông-báo)
10. [Checklist đóng bug](#10-checklist-đóng-bug)

---

## 1. Bug thuộc layer nào?

Tester đọc spec chain **PRD → BDD → Code** để xác định layer của bug:

```
              ┌─────────────────────────────┐
              │     Bug được phát hiện      │
              └──────────────┬──────────────┘
              ┌──────────────▼──────────────┐
              │ Tester đọc PRD → BDD → Code │
              └──────────────┬──────────────┘
        ┌────────────────────┼────────────────────┐
┌───────▼───────┐  ┌─────────▼─────────┐  ┌────────▼────────┐
│ PRD mơ hồ /   │  │ BDD sai so PRD /  │  │ Code sai so BDD │
│ yêu cầu đổi   │  │ thiếu scenario    │  │ (phổ biến nhất) │
└───────┬───────┘  └─────────┬─────────┘  └────────┬────────┘
   → PO xử lý          → Dev xử lý           → Dev xử lý
   (Case 3, 4)          (Case 2)              (Case 1)
```

---

## 2. Bảng quyết định nhanh

| PRD | BDD | Code | Chẩn đoán | Ai fix | Re-test |
|---|---|---|---|---|---|
| ✅ rõ ràng | ✅ đúng | ❌ sai | **Code bug** | Dev | Tester |
| ✅ rõ ràng | ❌ sai | ❌ sai | **BDD bug** | Dev | Tester |
| ❌ mơ hồ | bất kỳ | bất kỳ | **PRD ambiguity** | PO → Dev | Tester |
| ✅ rõ ràng | ✅ đúng | ✅ đúng | **Env / data bug** | DevOps / Dev | Tester |
| Yêu cầu thay đổi | cũ | cũ | **PRD change** | PO → Dev | Tester |
| ✅ rõ ràng | ✅ đúng | ❌ UI sai | **Design Spec bug** | PO/Designer → Dev | Tester |

> **Sửa PRD/BDD/Design Spec → phải duyệt lại:** mọi case mà PO sửa PRD (Case 3/4), Dev sửa BDD (Case 2), hoặc PO/Designer sửa Design Spec (Case 5) → artifact đó **tự về `draft`** (PRD/Design `Status` ở Metadata; BDD `@trace.status`). Phải `/review-context` lại + người **đặt `approved`** trước khi downstream (tech-docs / code / QC) tiêu thụ. Proposal scenario: PO/Dev đặt `Status: accepted` → `/generate-bdd` tự nạp rồi lưu trữ.

---

## 3. Case 1 — Code Bug (Code ≠ BDD)

> Phổ biến nhất. Spec đúng, code implement sai.

```
Tester                    Dev
──────                    ────────────────────────────────────────────
Gửi bug report
(kèm spec context:
 PRD path, BDD path,
 AC bị vi phạm,
 BDD scenario fail)
                          Nhận bug report; đọc BDD scenario được chỉ định
                          /fix-bug "{BUG-ID}: {mô tả}"
                              → agent tìm divergence code vs BDD → propose fix
                          Review + apply fix
                          /dev-run-test → dev self-check pass ✅
                          (dev_selftest — smoke check, KHÔNG phải official QC;
                           QC pipeline /qc-run-test set qc_status ở flow riêng)
                          /validate-traces → no broken
                          Tạo PR + notify tester:
                          "Fixed — root cause: {X}; Re-test: {BDD scenario}"
Nhận thông báo; re-test scenario
→ PASS: đóng bug
→ FAIL: gửi lại với actual behavior mới
```

**Ví dụ:**

```
Bug: FT-001 — tài khoản khoá sau 6 lần sai, không phải 5
PRD AC3 : "5 lần sai → khoá"           ✅
BDD SC3 : "When 5 failures, Then 423"   ✅
Code    : if failCount > 5: lock()      ❌  ← > thay vì >=
Fix: đổi > thành >= → chạy BDD test → pass
→ notify: "Case 1 fixed, re-test FT-001-UC2-SC3"
```

> **Nếu root cause là lỗi AI hay lặp khi gen code** (vd dùng `>` thay vì `>=` cho boundary, hay gọi repository thẳng từ controller): `/fix-bug` sẽ hỏi *"Record as a project lesson? (Y/N)"* → chọn **Y**. Lesson được nạp vào mọi lần gen sau. Hoặc chủ động `/learn "AI hay X, đúng phải Y"`.

---

## 4. Case 2 — BDD Bug (BDD ≠ PRD)

> Dev gen BDD sai từ đầu, hoặc PRD đã update nhưng BDD chưa.

```
Tester                    Dev                       PO/BA
──────                    ────────────              ──────
Gửi bug report
(BDD scenario mâu
 thuẫn với PRD AC)
                          Đọc PRD → BDD: xác nhận BDD sai
                          Nếu PRD rõ ràng:
                            → update BDD scenario + code nếu cần
                            → /dev-run-test → dev self-check pass
                            → /validate-traces → PR + notify
                          Nếu PRD chưa đủ rõ → báo PO
                                                    Clarify PRD
                                                    (không đổi version
                                                     nếu chỉ làm rõ)
                          Update BDD + code → PR + notify
Re-test
```

**Ví dụ:**

```
Bug: FT-001 — BDD ghi "3 lần sai → khoá", PRD ghi "5 lần"
PRD AC3 : "5 lần sai → khoá"           ✅
BDD SC3 : "When 3 failures, Then 423"   ❌  ← dev gen BDD sai
Code    : if failCount >= 3: lock()     ❌  ← code theo BDD sai
Fix: 1) BDD 3→5  2) code >=3 → >=5  3) update test data
→ notify: "BDD + code fixed, re-test với 5 lần"
```

---

## 5. Case 3 — PRD Ambiguity (PRD không rõ)

> Không ai sai — PRD viết thiếu, dev và tester hiểu khác nhau.

```
Tester                    Dev                       PO/BA
──────                    ────────────              ──────
Gửi bug report
(AC mơ hồ: dev hiểu
 X, tester expect Y)
                          Đọc PRD → xác nhận AC chưa đủ rõ
                          KHÔNG tự fix code
                          Báo PO: "AC3 mơ hồ: dev assume 5 lần,
                          tester expect 3. Đúng là bao nhiêu?"
                                                    Quyết định: 5 lần
                                                    Update PRD AC3
                                                    Bump version nếu
                                                    thay đổi behavior
                                                    Notify dev + tester
                          Nhận PRD mới; update BDD/code nếu cần
                          /validate-traces → PR + notify
Re-test theo PRD mới
```

**Ví dụ:**

```
PRD AC3: "Sai password nhiều lần → khoá tài khoản"  ← "nhiều lần" = bao nhiêu?
Dev assume 5 lần; Tester expect 3 lần (thông lệ ngân hàng)
→ Không phải bug code/BDD — là PRD thiếu thông tin
→ PO quyết định 5 lần: "Sai password 5 lần liên tiếp → khoá 30 phút"
→ BDD đã đúng → chỉ close bug, không fix code
```

---

## 6. Case 4 — PRD Change (yêu cầu thay đổi)

> Không phải bug — business requirement thay đổi sau khi đã implement.

```
Tester                    Dev                       PO/BA
──────                    ────────────              ──────
Gửi bug report
(behavior đổi theo
 yêu cầu business mới)
                          So sánh PRD version: code theo v1.0,
                          tester expect v1.1 → báo PO xác nhận
                                                    Xác nhận: PRD change,
                                                    không phải bug
                                                    Update PRD v1.1
                                                    Notify dev + tester
                          Nhận PRD v1.1
                          /review-context → xem diff
                          Update BDD + code + tests
                          /validate-traces → PR + notify:
                          "Implemented per PRD v1.1; re-test behavior mới"
Re-test theo PRD v1.1
```

**Lưu ý quan trọng:** Bug report phải được **re-classify** thành "PRD Change Request" trước khi xử lý — không đưa vào bug backlog.

---

## 7. Case 5 — Design Spec Bug (UI ≠ Design Spec)

> Chỉ áp dụng cho FE/App. Code không khớp Design Spec.

```
Tester                    Dev (FE/App)              PO/BA + Designer
──────                    ────────────              ──────────────────
Gửi bug report
(screen X không khớp
 Design Spec section Y)
                          Đọc Design Spec → xác nhận spec nói gì
                          Nếu code sai Design Spec:
                            → fix code theo spec → PR + notify
                          Nếu Design Spec sai/lỗi thời:
                            → báo PO/Designer
                                                    Review + update Design Spec
                                                    Notify dev
                          Nhận Design Spec mới → fix code → PR + notify
Re-test UI
```

---

## 8. Case 6 — Environment / Data Bug

> Spec đúng hết, code đúng hết — lỗi ở infra hoặc test data.

```
Dấu hiệu nhận biết:
- Bug chỉ xảy ra trên staging, không reproduce local
- Bug xảy ra với 1 user cụ thể, không với user khác
- Bug xảy ra sau deploy, không trước
- /dev-run-test (dev_selftest — dev self-check) pass nhưng manual / QC test fail

Xử lý:
Tester → cung cấp: environment, user ID, timestamp, request ID
Dev    → check logs, config, database state
DevOps → check infra, env vars, deploy artifacts
```

---

## 9. Giao tiếp hiệu quả — các format thông báo

### Git flow của feedback loop (2 repo, 1 vòng)

Mọi feedback đi qua **git**, không qua chat: file được ghi vào spec repo, commit+push, rồi PO/Dev kéo về bằng `/sync`. Đây là đường đi đầy đủ — từ lúc QC/Tester phát hiện tới lúc bug `Closed`:

```
  QC / Tester                     SPEC REPO (PO sở hữu)              SERVICE SUBMODULE (Dev sở hữu)
  ─────────────                   ─────────────────────              ──────────────────────────────
  /report-bug      read-only
  /propose-scenario  specs/code
  (QC: /qc-run-test FAIL,
   /qc-analyze DOC_GAPS)
        │ ghi file ──────────────▶ feedback/bug-reports/*.md          (KHÔNG sửa specs/code trực tiếp)
                                   feedback/bdd-proposals/*.md
                                   feedback/prd-change-requests/*.md
                                     State: Open
                                       │ git commit + push  (tầng spec repo)
                                       ▼
                                  origin(spec) ──── /sync (PO/Dev pull) ────┐
                                                                            ▼
                                       ┌──────────────── phân loại (Case 1–6) ───────────────┐
                                       ▼                                                      ▼
                          PRD/BDD gap → PO sửa PRD + bump                  Code bug → Dev /fix-bug {BUG-ID}
                          → commit+push spec repo                          → set State: Fixed
                          → /generate-bdd lại                              → commit+push service (tầng 1)
                                       │                                   + umbrella pointer (tầng 2)
                                       └───────────────────┬──────────────────┘
                                                           ▼
                          QC /qc-run-test re-verify ──── pass ──▶ State: Closed
                                                                  (qc_owner / qc_blocked_by tự clear)
```

- **2 tầng push** chỉ áp dụng phía service submodule (xem [Sync & Update §4.4](sync-and-update.md#44--commit-2-tầng-thay-đổi-trong-service-submodule)); feedback file nằm trong **spec repo** nên chỉ cần push 1 tầng ở đó.
- `/sync` chỉ surface bug `State: Open`; `Fixed`/`Closed` để riêng (xem [§10](#10-checklist-đóng-bug)).

### Bug report (Tester → Dev)

Tester chạy `/report-bug {UC-ID} {mô tả}` → tự sinh report theo format dưới (gồm phân loại layer + phát hiện coverage gap), commit+push vào **spec repo** `feedback/bug-reports/{BUG-ID}.md`. PO/Dev thấy khi chạy `/sync` (dòng `📥 New tester feedback`):

```
[BUG-{ID}] {Feature} — {mô tả ngắn}

Spec:  {PRD path} v{x.x} | AC{N}: "{AC text}"
BDD:   {BDD path} → "{Scenario title}"   (hoặc: ⚠️ no scenario covers this)
Layer: Code / BDD / PRD / Env  ← Tester đề xuất, Dev xác nhận

Expected: {theo spec}
Actual:   {thực tế}
Repro:    {steps}
Env:      staging / {date deploy}
```

> **Nếu Layer = "coverage gap"** (behavior đúng nhưng chưa scenario nào cover): tester chạy `/propose-scenario {UC-ID}` → draft scenario vào `bdd-proposals/` cho PO/Dev duyệt. Xem [Case 2](#4-case-2--bdd-bug-bdd--prd).

### Fix xong (Dev → Tester)

```
[BUG-{ID}] Fixed ✅

Root cause: Case {1..6} — {mô tả ngắn}
Changed:
  - {file/component}: {what changed}
  - BDD: {updated / unchanged}
  - PRD: {unchanged / clarified by PO}

Deploy: staging @ {time} — {commit/PR link}
Re-test: {BDD scenario ID hoặc AC number}
```

### Cần PO clarify (Dev → PO)

```
[PRD-CLARIFY] {Feature} — AC{N} mơ hồ

Tình huống:
  Dev implement: {X}
  Tester expect: {Y}
  Triggered by: BUG-{ID}

PRD AC hiện tại: "{AC text}"

Câu hỏi: {câu hỏi cụ thể — 1 câu}
Cần trả lời trước: {date} để unblock tester
```

---

## 10. Checklist đóng bug

Trước khi đánh "Resolved":

**Dev:**
- [ ] Root cause xác định rõ (Case 1/2/3/4/5/6)
- [ ] Fix đúng layer — không patch code khi lỗi ở BDD hoặc PRD
- [ ] `/validate-traces` → no broken traces
- [ ] `/dev-run-test` → dev self-check pass (`dev_selftest` — smoke check, KHÔNG thay official QC `qc_status` do `/qc-run-test` set)
- [ ] Nếu root cause là lỗi AI gen hay lặp → đã `/learn` (hoặc accept prompt khi `/fix-bug`)
- [ ] Notify tester với đầy đủ thông tin re-test

**Tester / QC:**
- [ ] Re-test đúng scenario được chỉ định (QC: `/qc-run-test {UC-ID}` lại → `qc_status` flip `fail → pass`)
- [ ] Kiểm tra regression: các AC khác của cùng PRD không bị ảnh hưởng
- [ ] Confirm PASS trước khi close

**Vòng đời bug report (đừng để `feedback/` phình mãi):**
- [ ] `State` của `feedback/bug-reports/{BUG-ID}.md`: `🟢 Open` → `🟡 Fixed` (set bởi `/fix-bug {BUG-ID}`) → `🟢 Closed` (sau khi `/qc-run-test` re-verify pass).
- [ ] Khi `Closed`: `/qc-run-test` clear `qc_owner`/`qc_blocked_by` của SC (tự động khi `qc_status=pass`); tuỳ chọn move file sang `feedback/bug-reports/archive/`.
- [ ] `/sync` chỉ surface bug `State: Open` là "đang chờ" — Fixed/Closed không làm nhiễu PO/PM.

**PO** *(nếu PRD được cập nhật)*:
- [ ] PRD version mới đã được bump
- [ ] Changelog có entry cho thay đổi
- [ ] Notify dev và tester về version mới

---

*Xem thêm:* [Sync & Update](sync-and-update.md) · [Guide · Tester](../02-guides/tester/README.md) · [chương QC Automation](../02-guides/tester/qc-automation.md).
