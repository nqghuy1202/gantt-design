# Pha 1B — Nghiên cứu đặc tả: MES Control Tower (Gantt theo công đoạn sản xuất)

> Nguồn: `TÀI LIỆU ĐẶC TẢ TÍNH NĂNG SƠ ĐỒ GANT THEO CÔNG ĐOẠN SẢN XUẤT.docx` + 6 ảnh màn hình.
> Ảnh lưu tại `gantt-ux-research/docx-images/`.

## 0. Định hình lại phạm vi (QUAN TRỌNG)

Đây **không** phải Gantt quản lý dự án thông thường. Đây là **bảng điều khiển điều độ sản xuất (MES Control Tower)** trực quan hóa **Lệnh sản xuất (PO)** chạy qua các **công đoạn → tổ máy** theo trục thời gian một ca làm việc. Plug-in dhtmlxGantt chỉ là **engine render timeline**; phần lớn giá trị nằm ở lớp KPI, cảnh báo, cấu trúc cây tài nguyên và mã màu nghiệp vụ phủ lên trên.

→ Ảnh `image1.png` là **màn hình đích mong muốn**. `image2/image3` là màn APEX hiện có ("Lập lệnh sản xuất") — **nguồn dữ liệu** và là nơi cần **thêm nút "Xem tiến độ lệnh sản xuất"** để mở Control Tower này.

## 1. Bố cục màn hình đích (image1)

```
┌─ Top bar: ☰  MES Control Tower   Factory A › Workshop 1 › Line 02      🔍search  🔔 ⚙ 👤J.Smith ─┐
├─ Header KPIs: TIME HORIZON | ORDERS 142/38/10 | QTY 45.2k/48.0k | YIELD 98.5% | OEE 84.2% |
│               SA 92.4% | FCST 98.5%                                    [GANTT | HEATMAP] ───────┤
├──────────────┬──────────────────────────────────────────────────────────────────────────────────┤
│ SIDEBAR      │  Toolbar: [All Operations ▾] [All Products ▾]                 🔍 100% zoom         │
│ PRODUCTION   ├──────────────┬───────────────────────────────────────────────────────────────────┤
│ HEALTH       │ RESOURCE     │ 06:00   08:00   10:00   12:00   14:00   16:00                       │
│ (bar/stage)  │ HIERARCHY    │         ▏(current time indicator = đường đỏ dọc)                    │
│              │ 10.Phối trộn │  [PO bars với % progress, màu theo trạng thái]                      │
│ ALERT CENTER │  MX-01[0/0]🟠│                                                                     │
│ (3 loại)     │  MX-02[..]🟢 │  ▓▓PO240601 100%▓▓        ▓▓PO240602 100%▓▓                         │
│              │ ⚠20.Lọc tinh │                                                                     │
│ [APS         │  PT-01[..]🔴 │             ▓PO240605 10%▓ (đỏ - quá tải)                           │
│  Optimizer]  │  ...         │                                                                     │
└──────────────┴──────────────┴───────────────────────────────────────────────────────────────────┘
```

## 2. Tham số truy vấn (bộ lọc đầu vào)
- **Phân xưởng**: phòng ban sản xuất theo cơ cấu tổ chức, có "All".
- **Ca làm việc**: 8h hoặc 24h → quyết định TIME HORIZON của trục thời gian.
- **Đơn hàng / Dự báo**: chọn mã đơn/dự báo có lệnh sản xuất.
- **Lệnh sản xuất (PO)**: danh sách, có "All".

## 3. Header KPIs (7 chỉ số toàn ca)
| Chỉ số | Ý nghĩa | Công thức (rút gọn) |
|--------|---------|----------------------|
| **ORDERS** | Total / Running / Delayed | đếm PO theo trạng thái |
| **QTY** | Plan vs Actual | lũy kế ERP vs thực tế mặt bằng |
| **YIELD %** | tỷ lệ đạt chất lượng lần đầu | TP / (TP + phế) ×100 |
| **OEE %** | hiệu suất thiết bị (ước tính) | OK Qty / Sản lượng tiêu chuẩn |
| **SA %** | tuân thủ lịch trình | PO đúng hạn / tổng PO ×100 (đúng hạn = không trễ quá X phút so Planned End) |
| **FCST %** | % chặng đường đã đi | OK Qty lũy kế / Target Qty ×100 |

## 4. Sidebar trái

**A. PRODUCTION HEALTH** — mỗi công đoạn một thanh % hiệu suất = Σ Actual Qty / Σ Target Qty các PO trong ngày. **Mã màu:**
- 🔵 Xanh dương <80% — bình thường.
- 🟠 Cam 80–90% — sắp quá tải.
- 🔴 Đỏ >90–≥100% — quá tải / nghẽn cổ chai.

**B. ALERT CENTER** (3 loại cảnh báo tự động):
1. **Capacity Overload** — Target Qty PO trong ngày > Standard Rate tổ × thời gian mở ca → bắn cảnh báo + thanh công đoạn đỏ 100%.
2. **Orders Delayed** — Current Time + thời gian chạy dự kiến còn lại (= Target Qty ÷ Standard Rate) > Delivery Due Date → liệt kê PO trễ.
3. **Material Shortage** — đối chiếu BOM PO sắp chạy với tồn kho ERP; Tồn < Nhu cầu → cảnh báo thủ kho.

**C. Nút APS Optimizer** (tối ưu xếp lịch — giai đoạn sau).

## 5. Lưới Gantt — Resource Hierarchy (cốt lõi nghiệp vụ)

**Cây tài nguyên 2 cấp:** Công đoạn (10. Phối trộn, 20. Lọc tinh, 30. Thanh trùng...) → Tổ máy.

- **Nhãn tổ máy động:** `Tên máy [Actual/Plan]` ví dụ `PT-01 [2678/3000]` + chip "Tổ 1".
- **Hộp nhãn hiệu suất** cạnh tên công đoạn: <100% = nghẽn, ≥100% = thông suốt.
- **Đèn trạng thái (status dots):** 🟢 vận hành bình thường · 🟠 trống việc/chờ BTP · 🔴 trễ nặng/sự cố.

**Thanh PO (PO Bars):**
- Dải màu progress chạy trái→phải theo % sản lượng thực tế.
- **Current Time Indicator**: đường kẻ dọc đỏ xuyên lưới = thời gian thực hiện tại.
- **Cảnh báo dòng chảy ngược (reverse-flow):** nếu công đoạn sau (Thanh trùng 85%) có % cao hơn công đoạn trước (Lọc tinh 10%) cùng thời điểm → cảnh báo lỗi quên nhập liệu / transfer lag sai. (icon ⚠ trên bar).
- Bar đỏ = PO/công đoạn quá tải hoặc trễ; xanh lá = đúng tiến độ; xanh dương = đang chạy.

## 6. Khoảng cách giữa plug-in mặc định và màn hình đích

| Yêu cầu đích | dhtmlx mặc định | Cần làm |
|--------------|-----------------|---------|
| Cây tài nguyên Công đoạn→Tổ→PO | grid task phân cấp parent | map dữ liệu MES vào cấu trúc, render layer riêng theo resource |
| Nhãn `[Actual/Plan]` + chip Tổ + đèn | không có | custom grid column template |
| Current time indicator đỏ | có (`gantt.addMarker`) | bật + style |
| Mã màu theo hiệu suất/trạng thái | template class | viết `gantt.templates.task_class` theo nghiệp vụ |
| Header KPIs + Sidebar Health + Alert Center | KHÔNG có | build UI APEX/HTML bao quanh region Gantt |
| Cảnh báo reverse-flow | KHÔNG có | logic JS so sánh % giữa công đoạn |
| Heatmap toggle | KHÔNG có | view thứ 2 (giai đoạn sau) |
| Lọc Operations/Products, zoom | thủ công | toolbar custom |

## 7. Hệ quả cho kế hoạch (sẽ chi tiết ở Pha 2-3)
- Plug-in chỉ là 1 phần; phần lớn là **lớp dashboard MES bao quanh** + **template render tùy biến**. Sẽ tận dụng before/after-init JS + CSS tokens, và bổ sung 1 layer JS điều phối (KPIs, alerts, color logic).
- Mockup HTML ở Pha 3 sẽ dựng lại đúng `image1` với design tokens hiện đại (đẹp/chuyên nghiệp) nhưng giữ mật độ thông tin cao đặc thù điều độ.
- Cần xác nhận: dữ liệu KPI/alert lấy realtime tới đâu, hay tính sẵn trong SQL region.

---

→ **CHECKPOINT.** Mời bạn duyệt phần hiểu đặc tả này. Mình có vài câu hỏi thu hẹp ở tin nhắn kèm theo trước khi sang Pha 2 (user story).
