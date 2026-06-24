# Nghiên cứu: Custom TẠI CHỖ trên plug-in dhtmlxGantt (không tạo plug-in mới)

## Kết luận ngắn
**Hoàn toàn khả thi, và thực ra ÍT việc hơn tạo mới.** Plug-in này có sẵn toàn bộ "bộ khung" APEX (region type, AJAX dữ liệu, cơ chế attribute, nạp file, sự kiện refresh, page items). Ta **giữ nguyên bộ khung** và chỉ **thay lớp render** (dhtmlx → renderer tự viết). Không cần dựng lại plumbing từ đầu.

## Plug-in gồm 3 phần — phần nào GIỮ, phần nào THAY

| Thành phần | Vai trò | Khi custom tại chỗ |
|-----------|---------|--------------------|
| **Render Function PL/SQL** (`plugin-source.sql` → `dhtmlx_gantt_render`) | nạp css/js, in container, truyền config | **SỬA**: bỏ nạp dhtmlx, nạp file mới, in container Control Tower |
| **AJAX Function PL/SQL** (`dhtmlx_gantt_ajax`) | chạy region SQL, trả CLOB | **GIỮ NGUYÊN** — đã trả CLOB JSON/XML, dùng lại y nguyên |
| **Helper JS** (`plugin-dhtmlxgantt-helper.js`) | parse + render + bind sự kiện APEX | **THAY RUỘT**: bỏ `gantt.*`, thay bằng renderer 24h; giữ phần AJAX/refresh/parse |
| **Vendor dhtmlx** (`server/dhtmlxgantt/…`) | thư viện gantt | **NGỪNG NẠP** (xoá hoặc để lại không dùng) |
| **CSS** | skin | **THAY** bằng `mes-control-tower.css` |
| **Attributes** (18 cái: skin, drag*, show_links…) | cấu hình | **TÁI SỬ DỤNG/ĐỔI TÊN**: bỏ cái thừa, thêm cái cần (ngày mặc định…) |
| **Sự kiện APEX** (apexrefresh, page items, before/after-init JS) | tích hợp trang | **GIỮ** — cực kỳ giá trị, không phải viết lại |

→ Nói cách khác: ta **fork tại chỗ**. Giữ ~50% (AJAX, plumbing, sự kiện, attribute framework), thay ~50% (render JS + CSS + vài dòng PL/SQL nạp file).

## So sánh 2 hướng

| Tiêu chí | **Custom tại chỗ (khuyến nghị)** | Tạo plug-in mới |
|----------|----------------------------------|------------------|
| Khối lượng việc | Ít hơn — tái dùng plumbing | Nhiều — dựng lại region type, AJAX, attribute, file mechanism |
| Rủi ro wiring APEX | Thấp (đã chạy được) | Cao hơn (dễ thiếu mắt xích) |
| Tên nội bộ | Giữ `COM.GITHUB.OGOBRECHT.DHTMLXGANTT` | Tên mới `MES.CONTROL.TOWER` |
| Region đang dùng plug-in | Bị đổi hành vi (nếu có) | Không ảnh hưởng |
| Sạch sẽ/branding | Còn dấu vết dhtmlx (tên, file) | Sạch, đặt tên theo MES |
| Nâng cấp bản gốc về sau | Xung đột (đã fork) | Độc lập |

## 3 mức "custom tại chỗ" (từ nhẹ → nặng)

1. **Chỉ qua Attributes (không sửa source)** — dùng "Before/After Init JS Code" + "After Refresh JS Code" + Custom CSS để phủ lên dhtmlx.
   → **KHÔNG hợp** với ta, vì ta *bỏ hẳn* dhtmlx, không chỉ tinh chỉnh nó. Mức này chỉ hợp khi vẫn xài gantt.

2. **Sửa Helper JS + CSS, giữ Render PL/SQL gần như cũ** — thay file js/css trong Plugin Files, render vẫn in container + onload.
   → Khả thi, gọn. Phải sửa 1 chút PL/SQL để ngừng nạp dhtmlx.

3. **Sửa cả Render PL/SQL + Helper JS + CSS + Attributes (fork tại chỗ đầy đủ)** ← **đây là hướng đúng cho ta.**
   → Vẫn là *cùng một plug-in*, chỉ thay ruột. Đúng tinh thần "custom chứ không tạo mới".

## Quy trình custom tại chỗ trong APEX (read-only)

1. Shared Components → Plug-ins → mở **dhtmlxGantt**.
2. **Render Function PL/SQL**: dán code mới (bỏ add_file/add_library dhtmlx; thêm nạp `mes-control-tower.css/js`; in container).
3. **Plug-in Files**: upload `mes-control-tower.js`, `mes-control-tower.css`; (tuỳ chọn) xoá thư mục `dhtmlxgantt/…` cho nhẹ.
4. **Attributes**: ẩn/bỏ skin & drag*; thêm "Ngày mặc định"… (hoặc giữ tạm, không bắt buộc đợt đầu).
5. Region SQL trả JSON `row_type` → chạy.

> Lưu ý GPLv2: bản gốc là GPLv2. Sửa & dùng nội bộ thoải mái; nếu phát hành lại phải giữ GPLv2 + ghi nguồn.

## Đề xuất
Đi **mức 3 — fork tại chỗ**: giữ plug-in & AJAX & sự kiện, thay render JS + CSS + vài dòng PL/SQL. Đây chính là các Pha B/C/D trong lộ trình `05-…`, chỉ khác ở chỗ **không đổi tên plug-in và tái dùng tối đa khung cũ** thay vì coi như plug-in mới.

### Cần bạn xác nhận
- Giữ tên nội bộ `COM.GITHUB.OGOBRECHT.DHTMLXGANTT` (đúng tinh thần "không tạo mới")?
- Có **xoá file dhtmlx** trong plug-in cho nhẹ không, hay để lại (an toàn, có thể quay lại)?
- Có cần giữ khả năng "bật lại chế độ gantt cũ" không, hay thay hẳn?
