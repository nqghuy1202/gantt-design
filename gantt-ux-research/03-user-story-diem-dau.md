# Pha 2 — User Story & Điểm đau (MES Control Tower)

Phương pháp bmad: lấy persona điều độ viên (chính), tổ trưởng, quản lý xưởng.

## Personas
- **P1 — Điều độ viên (primary):** nhìn màn cả ca, cần phát hiện nghẽn/trễ trong 3 giây.
- **P2 — Tổ trưởng/nhân viên:** xem việc tổ mình, cập nhật % sản lượng.
- **P3 — Quản lý xưởng/lãnh đạo:** xem tổng quan KPI, ra quyết định điều phối.

## User stories ưu tiên (MoSCoW)

| # | Story | Persona | Ưu tiên |
|---|-------|---------|---------|
| US1 | Là điều độ viên, tôi muốn thấy ngay công đoạn nào quá tải/nghẽn để can thiệp | P1 | Must |
| US2 | Là điều độ viên, tôi muốn 1 đường thời gian hiện tại để biết PO nào đang chạy đúng/trễ | P1 | Must |
| US3 | Là điều độ viên, tôi muốn cảnh báo PO trễ giao / thiếu NVL ở một chỗ | P1 | Must |
| US4 | Là tổ trưởng, tôi muốn thấy sản lượng tổ `[Actual/Plan]` và trạng thái đèn | P2 | Must |
| US5 | Là QL, tôi muốn KPI toàn ca (ORDERS/QTY/YIELD/OEE/SA/FCST) ngay đầu màn | P3 | Must |
| US6 | Là điều độ viên, tôi muốn được cảnh báo lỗi dòng chảy ngược (nhập sai liệu) | P1 | Should |
| US7 | Là người dùng, tôi muốn lọc theo công đoạn/sản phẩm và zoom thời gian | All | Should |
| US8 | Là QL, tôi muốn xem dạng Heatmap để thấy mật độ tải | P3 | Could |
| US9 | Là điều độ viên, tôi muốn click PO mở chi tiết để cập nhật/điều chỉnh | P1 | Should |

## Điểm đau hiện tại (plug-in mặc định + màn lập lệnh) → Khắc phục

| Điểm đau | Vì sao khó chịu | Cách khắc phục (đưa vào mockup) |
|----------|-----------------|----------------------------------|
| Sửa task phải double-click mở modal, nhiều bước | Chậm khi điều độ liên tục | Click bar → panel chi tiết trượt phải (không rời màn); thao tác nhanh inline |
| Không thấy bức tranh toàn ca | Phải tự ghép số liệu | Header KPIs + Sidebar Health luôn hiển thị |
| Cảnh báo rải rác / không có | Bỏ sót nghẽn, trễ | Alert Center gom 3 loại, có badge số + màu |
| Màu mặc định nhạt, khó phân biệt trạng thái | Mỏi mắt cả ca, dễ nhầm | Hệ màu trạng thái nhất quán (xanh/cam/đỏ) + đèn + viền bar |
| Không phân biệt vai trò | Ai cũng kéo được, dễ sai | Chế độ xem theo vai trò (read-only cho lãnh đạo) — đánh dấu ở mockup |
| Khó đọc nhãn tổ/PO khi dày | Mất thông tin | Template nhãn `Tên [Act/Plan]` + chip Tổ + truncation thông minh |
| Không có "đang ở đâu trong ca" | Khó ước lượng kịp/trễ | Current-time line đỏ + nhãn giờ hiện tại |

→ Các khắc phục này được hiện thực hóa trong 2 mockup ở Pha 3.
