# Pha A — Data Contract (JSON region trả về cho MES Control Tower)

Region SQL trả **1 CLOB** là JSON object `{ "data": [ ... ], "links": [] }`. Mảng `data` gồm các hàng phân cấp theo `row_type` (stage → machine → po) qua `id`/`parent`.

## Cấu trúc chung
```jsonc
{
  "data": [ /* các node stage, machine, po */ ],
  "links": []            // chưa dùng ở chế độ read-only
}
```

## Node STAGE (công đoạn)
| Trường | Bắt buộc | Ý nghĩa |
|--------|----------|---------|
| `id` | ✓ | định danh, vd `S975` |
| `parent` | ✓ | `0` (gốc) |
| `row_type` | ✓ | `"stage"` |
| `text` | ✓ | tên công đoạn, vd `"1 - Phối trộn"` |
| `stage_load` | ✓ | % tải công đoạn → thanh Production Health (0–100) |

## Node MACHINE (tổ máy)
| Trường | Bắt buộc | Ý nghĩa |
|--------|----------|---------|
| `id` | ✓ | vd `M975_16699` |
| `parent` | ✓ | id của stage cha |
| `row_type` | ✓ | `"machine"` |
| `text` | ✓ | tên tổ/máy |
| `actual_qty` | ✓ | sản lượng thực tế → nhãn `[actual/plan]` |
| `plan_qty` | ✓ | sản lượng kế hoạch |
| `status` | ✓ | đèn trạng thái: `"g"` xanh · `"o"` cam · `"r"` đỏ |

## Node PO (lệnh sản xuất)
| Trường | Bắt buộc | Ý nghĩa |
|--------|----------|---------|
| `id` | ✓ | vd `P975_..._WJH-00011_2025` |
| `parent` | ✓ | id của machine cha |
| `row_type` | ✓ | `"po"` |
| `text` | ✓ | mã lệnh, vd `"WJH-00011/2025"` |
| `start_date` | ✓ | `"yyyy-mm-dd hh:mm:ss"` (giờ bắt đầu trong ngày) |
| `duration` | ✓ | độ dài — **đơn vị cấu hình** (mặc định GIỜ). Renderer: `end = start + duration × DURATION_UNIT_MS` |
| `progress` | ✓ | 0..1 → % + thanh đáy |
| `item_name` | ✓ | tên sản phẩm |
| `order_number` | ✓ | mã đơn hàng |
| `quantity` | ✓ | sản lượng lệnh |
| `expected_finish` |  | giờ dự kiến xong (text) |
| `po_state` | ✓ | `"ontrack"` · `"running"` · `"delayed"` → màu thanh (xanh lá / xanh dương / đỏ) |
| `material_short` |  | `true/false` → cảnh báo thiếu NVL trong popover |

## Quy ước render (read-only)
- **Phạm vi:** 1 ngày, trục 00:00–24:00. PO lọc theo ngày đang chọn (mặc định SYSDATE); phần ngoài ngày cắt bỏ (lớp an toàn).
- **Màu thanh PO:** `delayed`→đỏ; `progress≥1`→xanh lá (ok); còn lại→xanh dương (running).
- **Stacking:** PO chồng giờ trên cùng machine → tách dòng (greedy), ≥4 dòng → cluster "+N".
- **Thanh hẹp:** nhãn ngoài; **hover:** popover đầy đủ.
- **KPIs:** tính từ data trong ngày (Orders distinct, Qty plan/act, FCST=act/plan…). Trường KPI nâng cao (Yield/OEE/SA) sẽ bổ sung khi có nguồn — tạm hiển thị từ data hoặc "—".

## Cấu hình renderer (đặt trong onload PL/SQL)
| Config | Mặc định | Ghi chú |
|--------|----------|---------|
| `defaultDate` | SYSDATE | ngày mở màn |
| `durationUnit` | `"hour"` | `"minute"` \| `"hour"` — đổi đơn vị duration |
| `showKpis` | true | bật/tắt Header KPIs |
| `showSidebar` | true | bật/tắt Production Health |
| `narrowPx` | 140 | ngưỡng nhãn ngoài |
| `clusterMin` | 4 | số dòng để gộp cluster |

> Contract này tương thích ngược: nếu thiếu trường tuỳ chọn, renderer dùng giá trị an toàn (rỗng/false).
