# Pha 4 — Xử lý chồng lấn thời gian (time-overlap) trên Gantt MES

## Bối cảnh bài toán
Hai (hoặc nhiều) PO trùng khung thời gian trên cùng một hàng. **Đây là trùng hợp lệ** — tại thời điểm đó có nhiều lệnh cùng được đưa vào sản xuất song song, KHÔNG phải xung đột tài nguyên. Vì vậy bài toán thuần là **hiển thị**: xếp các thanh sao cho đọc đủ, rõ, đẹp — không có cảnh báo/đỏ/Alert.

## Nguyên tắc
- Không tô đỏ, không pill "xung đột", không đẩy vào Alert Center.
- Mỗi PO giữ nguyên màu trạng thái riêng (ok/run/bad theo tiến độ) và thanh đáy % hiệu suất.
- Mục tiêu: đọc đủ mọi PO chồng giờ, giữ mật độ gọn, đẹp/hiện đại.

## 3 phương án khảo sát (research)

### PA1 — Sub-row stacking (tách dòng con tự động)
Khi phát hiện overlap, hàng máy tự nở cao, mỗi PO xuống 1 dòng con riêng → **cả 2 thanh đọc đủ 100%**, không che nhau.
- ✅ Rõ ràng nhất, không mất thông tin; quen thuộc (MS Project, dhtmlx).
- ❌ Tốn chiều dọc khi nhiều overlap.

### PA2 — Cascade offset (đè lệch nhẹ + z-index)
Hai thanh đè nhau, lệch dọc ~10px, hover/click đưa lên trước.
- ✅ Tiết kiệm không gian, giữ mật độ cao.
- ❌ Thanh dưới bị che một phần; khó đọc khi >2.

### PA3 — Cluster + badge "+N"
Gộp cụm overlap thành 1 chip, hiện "+N PO", click mở popover danh sách.
- ✅ Cực gọn khi rất dày.
- ❌ Ẩn chi tiết, thêm 1 cú click; không hợp khi chỉ 2 thanh.

## Khuyến nghị — Hybrid stacking (gọn + đọc đủ)

> **Mặc định PA1 (sub-row stacking) khi ≤3 PO chồng; tự chuyển PA3 (cluster +N) khi ≥4.** Không tô đỏ, không cảnh báo.

Lý do: với 2–3 lệnh song song, tách dòng con cho đọc đủ và vẫn gọn; khi quá dày (≥4) thì gộp cluster để không vỡ layout, click mở popover xem chi tiết. Đúng tiêu chí đẹp/hiện đại/chuyên nghiệp/thân thiện.

### Chi tiết tương tác
- Hàng tự nở cao theo số dòng con; mỗi PO 1 dòng riêng, giữ nguyên màu trạng thái + thanh đáy %.
- Tuỳ chọn nền nhạt mờ ở đoạn nhiều lệnh chồng (xám/xanh rất nhạt) chỉ để gom thị giác — trung tính, không mang nghĩa cảnh báo.
- ≥4 PO: chip cluster "+N", click mở popover danh sách.

→ Demo trực quan 3 phương án: `mockups/demo-overlap.html` (mở để so sánh).
