[📚 Docs](../README.md) › Commands (Dễ Hiểu)

# 06 · Commands — Giải Thích Dễ Hiểu

> Mỗi trang giải thích **luồng chạy của một slash command** bằng ngôn ngữ dễ hiểu + ví von đời thường. Bổ sung cho đặc tả chính thức ở [05 · Reference](../05-reference/) — mục này ưu tiên *trực giác*, giúp bạn hình dung "lệnh làm gì, theo trình tự nào, vì sao" trước khi đọc chi tiết.

Đủ **30 lệnh**, gom theo phase pipeline. Mỗi trang có một ví von + các bước + vị trí trong dây chuyền + một câu chốt.

---

## Discovery & PRD (PO/BA)

| Trang | Ví von |
|-------|--------|
| [/setup-ai-first](explain-setup-ai-first.md) | Dựng khung xưởng một-lần (thư mục + nội quy + config + từ điển) |
| [/define-product](explain-define-product.md) | Buổi phỏng vấn 8 chặng moi ý tưởng thành hồ sơ |
| [/generate-prd](explain-generate-prd.md) | Thư ký biến biên bản phỏng vấn thành hồ sơ chuẩn |
| [/refine-prd](explain-refine-prd.md) | Tổ soi bài 3 lăng kính nâng chất nội dung PRD |
| [/review-context](explain-review-context.md) | Cửa kiểm định chất lượng cuối (PRD & BDD) |

## Design & BDD

| Trang | Ví von |
|-------|--------|
| [/generate-design-spec](explain-generate-design-spec.md) | Bản thiết kế màn hình bám Figma thật, 2 tầng ngôn ngữ |
| [/generate-bdd](explain-generate-bdd.md) | Biến yêu cầu thành kịch bản kiểm thử (web/app/system) |

## Tech Design & Code (Dev)

| Trang | Ví von |
|-------|--------|
| [/generate-tech-docs](explain-generate-tech-docs.md) | Bản vẽ thi công (BE contract / FE client design) |
| [/review-tech-docs](explain-review-tech-docs.md) | Hội đồng nghiệm thu bản vẽ + cổng chữ ký liên team |
| [/generate-code](explain-generate-code.md) | Thợ thi công bám bản vẽ, làm đúng một hạng mục |
| [/review-code](explain-review-code.md) | Giám sát công trình soi code 4 mặt |
| [/map-testids](explain-map-testids.md) | Dán nhãn tên cố định lên element FE tái dùng/đã-có |

## Dev Test & Fix

| Trang | Ví von |
|-------|--------|
| [/dev-gen-test](explain-dev-gen-test.md) | Sinh bộ tự-kiểm nhanh của dev |
| [/dev-run-test](explain-dev-run-test.md) | Bấm nút chạy tự-kiểm + chẩn lỗi theo platform |
| [/dev-smoke-test](explain-dev-smoke-test.md) | Thử máy tại chỗ trên service/app đang chạy |
| [/debug](explain-debug.md) | Hỏi nhanh đồng nghiệp — phân tích lỗi, không sửa |
| [/fix-bug](explain-fix-bug.md) | Xử lý sự cố có hồ sơ (truy gốc → sửa → test hồi quy) |

## QC Automation (dây chuyền 6 trạm)

| Trang | Ví von |
|-------|--------|
| [/qc-analyze](explain-qc-analyze.md) | Trạm 1 — QC Analyst: phân rã yêu cầu + gap |
| [/qc-plan](explain-qc-plan.md) | Trạm 2 — QC Planner: rủi ro + câu hỏi cho dev |
| [/qc-design-test](explain-qc-design-test.md) | Trạm 3 — QC Designer: viết test case Markdown |
| [/qc-review](explain-qc-review.md) | Trạm 4 — cổng review hai chiều (case & script) |
| [/qc-run-test](explain-qc-run-test.md) | Trạm 5 — QC Runner: chạy Playwright, ghi `qc_status` chính thức |
| [/qc-report](explain-qc-report.md) | Trạm 6 — report + evidence, đẩy product-gap về PO/Dev |

## Feedback & Vận hành

| Trang | Ví von |
|-------|--------|
| [/report-bug](explain-report-bug.md) | Phiếu sự cố có hồ sơ spec (tester/QC) |
| [/propose-scenario](explain-propose-scenario.md) | Hòm góp ý kịch bản test mới |
| [/learn](explain-learn.md) | Dán tờ nhắc guardrail lên tường xưởng |
| [/validate-traces](explain-validate-traces.md) | Đợt kiểm kê độ phủ spec↔code↔test |
| [/generate-spec-manifest](explain-generate-spec-manifest.md) | Lập sổ mục lục spec cho agent ngoài |
| [/sync](explain-sync.md) | Nghi thức đầu ngày đồng bộ umbrella |
| [/update-framework](explain-update-framework.md) | Thay bộ đồ nghề framework lên bản npm mới |

---

*Muốn giải thích dễ hiểu cho một lệnh khác → thêm một trang `explain-{command}.md` vào thư mục này và một dòng ở bảng phù hợp.*

*Xem thêm:* [05 · Reference › Commands](../05-reference/commands.md) (đặc tả đầy đủ) · [03 · Concepts › Cơ chế (dễ hiểu)](../03-concepts/mechanisms-explained.md) (giải thích cơ chế framework).
