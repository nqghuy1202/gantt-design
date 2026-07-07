# Pha 14 — Đặc tả UX màn hình: Gantt nhiều-ngày (bmad-ux-designer)

> **UX Designer (Sally):** chuyển nghiên cứu [13](13-gantt-theo-ngay-nghien-cuu-hinh-thuc.md) thành đặc tả
> màn hình dựng được. Scope: chỉ vùng **lưới Gantt nhiều-ngày** (chrome Header KPI / Production Health kế
> thừa mockup cũ, không dựng lại). Mockup: [mockups/gantt-theo-ngay.html](mockups/gantt-theo-ngay.html).

## 1. Bố cục tổng thể

```
┌───────────────────────────────────────────────────────────────────────┐
│  Thanh cửa sổ ngày:  ◀  06–12/07/2026 (7 ngày)  ▶      [Hôm nay]        │
├──────────────┬────────────────────────────────────────────────────────┤
│  CỘT NHÃN     │  06 T2 │ 07 T3 │ 08 T4 │▚09 T5▚│ 10 T6 │ 11 T7 │ 12 CN │  ← header ngày (▚ = hôm nay)
│ (sticky trái) ├────────┴───────┴───────┴───────┴───────┴───────┴───────┤
│ ▸ Công đoạn   │            (lưới cột-ngày, nền vạch ngày)                │
│   • Tổ [a/p]  │   ▓▓▓▓▓░░░  ← thanh PO: track KH nhạt + fill đậm         │
│     PO WJH-11 │                                                          │
└──────────────┴────────────────────────────────────────────────────────┘
```

- **Cột nhãn trái sticky:** phân cấp công đoạn → tổ → PO (thụt lề). Tổ hiện `[actual/plan]` + đèn trạng thái.
  Công đoạn hiện badge hiệu suất (trung tính; đỏ khi quá tải theo Production Health).
- **Vùng lưới phải:** nền kẻ **vạch dọc theo ngày** (không kẻ theo giờ). Cột **hôm nay** tô nền nhạt nổi.
- **Thanh cửa sổ ngày:** điều hướng ◀ ▶ theo tuần; nút **Hôm nay** đưa anchor về SYSDATE. (Trong APEX thật:
  driven bởi page item, DA refresh region — kế thừa cơ chế `dateItem` nhưng ở mức tuần.)

## 2. Thanh PO (đơn vị trực quan chính)

- **Hình học:** trải từ cột `plan_start` → `plan_end`. Bo góc mềm, cao ~26px.
- **2 lớp:** (1) *track KH* nền nhạt suốt span; (2) *fill thực tế* đậm rộng theo `progress` (từ trái).
- **Nhãn trong-thanh (degrade, kế thừa repo):**
  - span rộng (≥3 cột): `mã PO · %tiến độ · tên SP`.
  - span vừa (2 cột): `mã PO · %`.
  - span hẹp (1 cột / <96px): chỉ `%` (căn giữa); rất hẹp → chip marker (chỉ màu trạng thái).
  - **Không nhãn nổi** ngoài thanh — chi tiết ở tooltip.
- **Badge ô-ngày:** trong span, mỗi cột-ngày có bản ghi hiện chấm/nhãn nhỏ `+ok / −scrap` (đủ chỗ mới hiện,
  còn lại dồn vào tooltip). Ngày **chưa nhập** trong span → ô nhạt "—".
- **Màu (exception-driven):**
  - đang chạy / ontrack → **fill accent xanh dương** (`--accent`), viền trung tính.
  - `progress≥1` → **xanh lá** (`--ok`) = xong.
  - `delayed` (late_days>0 & chưa xong) → **đỏ** (`--bad`) + nhãn "Chậm N ngày".
  - Bình thường **không** màu semantic.

## 3. Trạng thái màn hình

| Trạng thái | Hiển thị |
|---|---|
| **Loading** | skeleton hàng nhãn + lưới mờ, không spinner giật |
| **Empty** (không PO trong cửa sổ) | thông điệp trung tính "Không có lệnh sản xuất trong khoảng ngày này" + gợi ý đổi tuần |
| **Lỗi** (AJAX không phải JSON) | thông điệp lỗi gọn + nút thử lại (kế thừa xử lý renderer) |
| **Ngày chưa nhập** | ô "—" nhạt (KHÔNG phải 0) — quy tắc trung thực D8 |

## 4. Tooltip / popover (hover thanh PO)

Hiển thị đầy đủ (thứ tự đọc): `mã PO · tên SP · đơn hàng` → `KH bắt đầu–kết thúc (ngày)` →
`tiến độ % · đạt lũy kế / KH` → **bảng nhật ký theo ngày** (Ngày · Đạt · Phế · LK · Người) →
cảnh báo (Chậm N ngày / Thiếu NVL / Dòng chảy ngược) nếu có. Đây là nơi mọi chi tiết bị degrade dồn về.

## 5. Tương tác (kế thừa doc PO-click)

- **Hover** thanh PO → popover (mục 4).
- **Click** thanh PO → emit `data-id`, trigger event APEX `mesgantt_po_click` (mở drawer ghi nhận — doc 09/10).
- **◀ ▶ / Hôm nay** → đổi cửa sổ ngày (mock: đổi anchor + re-render; APEX: set page item + DA refresh).

## 6. Token màu (đặt trên wrapper `.mgd`, an toàn APEX)

| Token | Giá trị | Dùng cho |
|---|---|---|
| `--primary` | `#15674C` (xanh lá ERP) | hành động, header nhấn |
| `--accent` | `#1D4ED8` (xanh dương) | fill tiến độ/dữ liệu (trung tính) |
| `--ok` | xanh lá | PO xong (progress≥1) |
| `--warn` | amber | cảnh báo sắp quá tải / phế cận ngưỡng |
| `--bad` | đỏ | trễ ngày / quá tải / phế vượt ngưỡng |
| `--ink`, `--muted`, `--line`, `--surface` | trung tính | chữ / nền / vạch ngày |

> Quy tắc màu: giá trị **trung tính** mặc định; semantic (xanh-lá/amber/đỏ) **chỉ bật khi vượt ngưỡng** →
> thường ≤3 phần tử có màu cùng lúc (kế thừa quyết định khóa của repo).

## 7. Acceptance criteria

- **AD1** — Trục là **cột-ngày**; **không** có bất kỳ nhãn/vạch giờ-phút nào (D1, doc12§7).
- **AD2** — Mỗi ô-ngày trong span PO map **đúng 1 bản ghi**; ngày chưa nhập = "—" ≠ 0 (D2/D8).
- **AD3** — Track KH (nhạt) vs fill thực tế (đậm) **phân biệt rõ**; PO trễ lệch cột-ngày thấy được (D3).
- **AD4** — **Exception-driven color**: bình thường trung tính; đỏ chỉ khi trễ/quá tải/phế vượt ngưỡng (D4).
- **AD5** — Cột **Hôm nay** nổi rõ ở mức ngày (D7).
- **AD6** — Phân cấp công đoạn→tổ→PO giữ nguyên; overlap cùng ngày = tách dòng, không tô đỏ.
- **AD7** — Nhãn thanh **degrade** theo độ rộng span; chi tiết đầy đủ trong tooltip (không nhãn nổi).
- **AD8** — Mockup CSS **thuần, scoped `.mgd`**, token trên wrapper → an toàn dán vào APEX.
- **AD9** — `%` là bước nhảy theo ngày, **không** animate kiểu realtime; tôn trọng `prefers-reduced-motion`.

## 8. Bàn giao sang mockup (taste)
Dựng `mockups/gantt-theo-ngay.html`: dữ liệu mẫu theo-ngày (vài công đoạn/tổ/PO trải nhiều ngày, có ngày
chưa nhập, có 1 PO trễ), renderer JS đặt thanh theo **chỉ số cột-ngày**, áp AD1–AD9.
