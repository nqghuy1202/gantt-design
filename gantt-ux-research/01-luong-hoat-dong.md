# Pha 1 — Luồng hoạt động plug-in dhtmlxGantt cho Oracle APEX

> Nguồn phân tích: `apex-plugin-dhtmlx-gantt/` (plugin v0.11.0, thư viện DHTMLX Gantt v7.0.11 — bản GPLv2 free).

## 1. Kiến trúc tổng thể

Plug-in là **Region Type Plugin** của APEX, gồm 3 lớp:

| Lớp | File | Vai trò |
|-----|------|---------|
| **Server (PL/SQL)** | `sources/plugin-source.sql` | Render HTML container + nạp CSS/JS + sinh code cấu hình `gantt.config.*`; xử lý AJAX trả dữ liệu (CLOB) |
| **Glue (JS helper)** | `sources/plugin-dhtmlxgantt-helper.js` | Cầu nối giữa APEX và thư viện DHTMLX: nạp dữ liệu, parse XML/JSON, gắn event, đẩy event ngược về APEX |
| **Vendor (thư viện)** | `server/dhtmlxgantt/codebase/dhtmlxgantt.js` + `.css` + skins | Engine Gantt thực sự (render, kéo-thả, dependency...) |

## 2. Data flow (luồng dữ liệu)

```
[Bảng Oracle]
   │  (Region SQL trả về 1 CLOB: XML hoặc JSON)
   ▼
dhtmlx_gantt_ajax (PL/SQL)  ──DBMS_SQL──► CLOB ──HTP.prn (chunk 12KB)──►
   ▼
plugin_dhtmlxGantt.load()  ──apex.server.plugin AJAX──►
   ▼
plugin_dhtmlxGantt.parse()   → tự dò "<" = XML, "{" = JSON
   │   - util_xml2json / JSON.parse
   │   - chuẩn hóa: progress/duration → float, open → bool
   │   - holidays → gantt.setWorkTime(hours:false)
   ▼
gantt.clearAll() + gantt.parse(data)   → DHTMLX render chart
   ▼
afterRefreshCode()  +  trigger "apexafterrefresh"
```

**Định dạng dữ liệu vào:** mỗi `<task>` có `id, text, start_date, progress, duration, parent, open` + URL tùy chọn (`url_edit`, `url_create_child`); `<link>` có `source, target, type`; `<holiday>` có `date`.

## 3. Luồng tương tác (event flow) — điểm cốt lõi của plug-in

Triết lý plug-in (theo README): **không edit inline trong chart**, mà bắt event rồi mở **modal APEX chuẩn** để sửa. Các event được gắn trong `init()`:

| Event DHTMLX | Plug-in làm gì | Sự kiện APEX phát ra |
|--------------|----------------|----------------------|
| `onTaskDblClick` | Mở `task.url_edit` nếu có | `dhtmlxgantt_task_double_click` |
| `onTaskCreated` | Mở `url_create_child` / `task_create_url_no_child` | `dhtmlxgantt_task_create` |
| `onLinkDblClick` | Mở `link.url_edit` | `dhtmlxgantt_link_double_click` |
| `onBeforeLinkAdd` | Chặn add mặc định, phát event | `dhtmlxgantt_link_create` |
| `onAfterTaskDrag` | Gửi `{mode, task}` (resize/progress/move) | `dhtmlxgantt_task_drag` |

> **Quan trọng:** mọi handler đều `return false` → DHTMLX KHÔNG tự lưu. Việc ghi DB hoàn toàn do **developer tự viết Dynamic Action** bắt event + AJAX. URL phải chuẩn bị sẵn trong Region SQL bằng `apex_util.prepare_url`.

## 4. Cấu hình khả dụng (Region Attributes → gantt.config)

`attribute_01` height · `_02` before-init JS · `_03` after-init JS · `_04` skin · `_05` locale · `_06` show_grid · `_07` show_links · `_08` show_progress · `_09` show_task_cells · `_10` drag_move · `_11` drag_progress · `_12` drag_resize · `_13` drag_links · `_14` work_time · `_15` highlight weekends · `_17` extensions (tooltip...) · `_18` rtl.

Skin có sẵn: broadway, contrast_black/white, **material**, meadow, skyblue, terrace.

## 5. Đánh giá nhanh kiến trúc (làm nền cho Pha 2-3)

**Điểm mạnh**
- Tách lớp sạch; mọi sức mạnh API DHTMLX vẫn truy cập được qua before/after-init JS.
- Hỗ trợ cả XML (DB cũ) lẫn JSON.

**Điểm yếu / nợ kỹ thuật (ứng viên tối ưu)**
1. **Sửa task phải mở modal + AJAX thủ công** → nhiều bước, không có inline edit thật → chậm với người thực thi.
2. **Không có lưu lạc quan (optimistic save)** sau kéo-thả: developer phải tự viết DA cho từng `mode`; dễ quên → mất dữ liệu khi refresh.
3. **UI mặc định cũ** (skin v7.0.11, 2020): màu nhạt, thiếu chiều sâu, không token hóa, không dark mode, không responsive cho mobile.
4. **Không có phân quyền thao tác** sẵn — mọi người cùng quyền kéo-thả (xung đột với yêu cầu đa vai trò).
5. **Không có zoom switch** (ngày/tuần/tháng) hay lọc/nhóm trong UI mặc định — phải tự code.
6. **Phản hồi lỗi** chỉ là `gantt.message` đỏ kỹ thuật, không thân thiện người dùng.
7. **Vendor v7.0.11 đã cũ**; nhiều tính năng (inline editors, zoom, marker) cần config thủ công.

---

## Đề xuất phương án kỹ thuật (vì bạn cho phép sửa cả core dhtmlx)

- **Không sửa thẳng** `dhtmlxgantt.js` (khó bảo trì khi nâng cấp). Thay vào đó: **override qua before/after-init JS + lớp CSS riêng (design tokens)** đặt cạnh plug-in. Chỉ chạm core khi thực sự cần (vá bug/tính năng không config được), và ghi chú rõ patch.
- Bổ sung 1 file `gantt-ux-overrides.css` + `gantt-ux-enhance.js` nạp sau vendor.

→ **CHECKPOINT PHA 1.** Mời bạn duyệt. Khi OK, mình sang **Pha 2 — User story & điểm đau (bmad-method)**.
