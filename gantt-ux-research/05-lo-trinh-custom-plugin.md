# Lộ trình custom plug-in → MES Control Tower (read-only)

## Quyết định đã chốt
- **Renderer tự viết** thay dhtmlx (bỏ vendor dhtmlx khỏi luồng render).
- **Region trọn gói**: Header KPIs + Sidebar (Production Health) + Gantt ngày-24h.
- **Read-only đợt này**: hiển thị tiến độ, popover, đổi ngày, lọc. Chưa kéo-thả/ghi DB.
- **Dữ liệu**: region SQL trả JSON `row_type` (stage/machine/po) như mẫu.
- Nền giao diện: **Phương án B (light)** đã chốt qua mockup `review-ngay-24h.html`.

## Kiến trúc mới của plug-in
```
Region SQL (CLOB JSON row_type)
      │ apex.server.plugin (AJAX)            ┌─ mes-control-tower.css  (tokens + styles)
      ▼                                      │
plugin-source.sql (PL/SQL render) ──nạp────►├─ mes-control-tower.js   (renderer thay dhtmlx)
   - bỏ nạp dhtmlx.js/.css                   │     • parse JSON
   - nạp css/js mới                          │     • KPIs + Health + Gantt 24h
   - in container + onload config            │     • stacking, nhãn ngoài, popover, đổi ngày
   - giữ hàm ajax trả CLOB                    └─────────────────────────────────────────────
```

## Các pha thực hiện (có checkpoint duyệt)

### Pha A — Contract dữ liệu & cấu hình
- Chuẩn hoá tài liệu JSON contract: các trường bắt buộc/tuỳ chọn cho stage/machine/po (id, parent, row_type, start_date, duration[giờ], progress, item_name, order_number, quantity, expected_finish, po_state, material_short, actual_qty/plan_qty, status, stage_load…).
- Xác định **đơn vị duration = giờ** (đã giả định, cần bạn xác nhận lần cuối).
- Định nghĩa region attributes mới (xem Pha E).
- _Sản phẩm:_ `06-data-contract.md`.

### Pha B — Lõi renderer JS  `sources/mes-control-tower.js`
- Tích hợp APEX: đọc config (pluginId, regionId, ajaxId, page items, ngày mặc định), bind `apexrefresh`, gọi AJAX `apex.server.plugin`, xử lý loading/empty/error.
- Dựng KPIs + Production Health + lưới Gantt 24h: trục giờ, stacking (≤3) + cluster (≥4), nhãn ngoài thanh hẹp, popover hover, đường NOW, đổi ngày (‹ › Hôm nay), lọc Operations/Products.
- Lấy code từ mockup `review-ngay-24h.html` làm gốc, refactor sạch theo module.
- _Checkpoint:_ chạy thử bằng test harness (xem Pha G) trước khi gắn APEX.

### Pha C — Styles  `sources/mes-control-tower.css`
- Tách toàn bộ CSS từ mockup thành file riêng, dùng design tokens (biến CSS), prefix class để không đụng theme APEX. Responsive cơ bản.

### Pha D — PL/SQL render  `sources/plugin-source.sql`
- Bỏ `apex_css.add_file(dhtmlx…)`, `apex_javascript.add_library(dhtmlx…)`.
- Nạp `mes-control-tower.css` + `.js`.
- In container (1 div gốc) + `apex_javascript.add_onload_code` truyền config (ngày mặc định từ item/expression, page items, queryDefined…).
- Giữ nguyên hàm `dhtmlx_gantt_ajax` (đổi tên → `mes_ct_ajax`) trả CLOB JSON.

### Pha E — Region attributes (file cài SQL)
- Thay nhóm attribute dhtmlx (skin, drag*, show_links…) bằng:
  Height · Ngày mặc định (item/expr) · Bật/tắt KPIs · Bật/tắt Sidebar · Lọc mặc định · After-refresh JS… (rút gọn, đúng nhu cầu read-only).

### Pha F — Đóng gói & cài đặt
- **Cách nhanh (khuyến nghị đợt này):** trong APEX Plugin editor → dán lại render PL/SQL, upload 2 file css/js mới vào Plugin Files, xoá các file dhtmlx, cập nhật attributes.
- **Tuỳ chọn sau:** sinh lại file install `.sql` hoàn chỉnh (`create_plugin_file` + attributes) để cài 1 phát.

### Pha G — Test harness & nghiệm thu
- `sources/_dev-harness.html`: nạp `mes-control-tower.css/js` + JSON mẫu, mô phỏng `apex.server.plugin` để chạy renderer ngoài APEX → review nhanh.
- Nghiệm thu với data mẫu của bạn; checklist: KPIs đúng, stacking đúng, popover đủ, đổi ngày OK, lỗi/empty xử lý.

## Trình tự đề xuất
A → (B song song C) → G (review) → D → E → F.
Mỗi mốc B-xong, D-xong sẽ **dừng cho bạn review** trước khi đi tiếp.

## Điểm cần bạn xác nhận
1. Đơn vị `duration` = **giờ** (đúng chứ?).
2. Đóng gói đợt này theo **cách nhanh** (upload file + dán render) hay bạn muốn mình **sinh file install .sql** luôn?
3. Ngày mặc định khi mở: **hôm nay**, hay lấy từ một page item bạn chỉ định?
4. Tên nội bộ plugin giữ nguyên `COM.GITHUB.OGOBRECHT.DHTMLXGANTT` hay đổi (vd `MES.CONTROL.TOWER`)?
