[📚 Docs](../README.md) › [Commands (Dễ Hiểu)](README.md) › /qc-plan

# `/qc-plan` — Giải Thích Luồng Chạy (Ngôn Ngữ Dễ Hiểu)

> **Trạm 2** của dây chuyền QC tự động. Ưu tiên trực giác — đặc tả đầy đủ xem [Reference › Commands](../05-reference/commands.md).

Trạm 2 đóng vai **QC Planner**: từ bản phân tích của Trạm 1, lập **kế hoạch test** — soi *rủi ro nằm ở đâu* và *phải hỏi dev/PO điều gì*.

---

## Cách chạy

- Đọc output của `/qc-analyze` (`REQUIREMENT_ANALYSIS.md` + `DOC_GAPS.md`).
- Lập `TEST_PLAN.md`: **ma trận rủi ro**, kịch bản **what-if**, phạm vi & chiến lược test theo từng tầng (functional / integration / e2e / non-functional), điều kiện vào/ra.
- Suy ra **questions-for-dev** từ mỗi gap còn mở/blocker — để gửi PO/Dev làm rõ trước khi thiết kế test.

Ranh giới vai: `/qc-plan` trả lời *"rủi ro ở đâu, hỏi gì?"* — **không** thiết kế test case cụ thể (đó là Trạm 3).

Giới hạn plan trong các scenario của UC để Trạm 3 thiết kế case theo từng scenario.

> **Một câu chốt:** `/qc-plan` là QC Planner — biến phân tích thành kế hoạch test (rủi ro · what-if · phạm vi từng tầng · câu hỏi cho dev), chưa viết case chi tiết.
