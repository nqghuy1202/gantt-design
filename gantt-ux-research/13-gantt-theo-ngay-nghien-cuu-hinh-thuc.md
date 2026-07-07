# Pha 13 — Nghiên cứu hình thức trực quan + Data-contract THEO NGÀY (research-1.0.0)

> Nối tiếp [12-gantt-theo-ngay-phan-tich](12-gantt-theo-ngay-phan-tich.md). Câu hỏi nghiên cứu:
> **Với dữ liệu 1 bản ghi/lệnh/công đoạn/ngày, hình thức trực quan nào truyền đạt tiến độ sản xuất
> trung thực & nhanh nhất cho điều độ viên?**

## 1. Phương pháp

- **Comparative design study** (đánh giá heuristic 3 phương án trực quan trên cùng bộ dữ liệu mẫu theo-ngày).
- **Data-form fit**: đối chiếu từng phương án với ràng buộc trung thực dữ liệu ở doc 12 (§7).
- Tiêu chí đánh giá (rút từ user stories D1–D9): *nắm tiến độ theo lịch · phân biệt KH/thực tế · phát hiện
  trễ ngày · phân biệt chưa-nhập vs 0 · giữ phân cấp công đoạn→tổ→PO · exception-driven color*.

## 2. Ba phương án được cân nhắc

### PA1 — Gantt nhiều-ngày (cột-ngày) ✅ CHỌN
Trục X = **các NGÀY**; hàng = công đoạn → tổ máy → PO. Thanh PO trải từ **ngày KH bắt đầu → ngày KH kết
thúc**. Mỗi **ô-ngày** = 1 bản ghi nhật ký ⇒ đơn vị trục khớp đơn vị dữ liệu.

- **Điểm mạnh:** giữ ẩn dụ Gantt quen thuộc (đặc tả gốc, người dùng đã quen); hành trình nhiều ngày hiện rõ;
  KH-vs-thực-tế tự nhiên (track nhạt vs fill đậm); trễ = lệch cột-ngày, đọc bằng mắt.
- **Rủi ro:** nếu span PO ngắn (1–2 ngày) thanh rất hẹp → dùng quy tắc degrade nhãn (kế thừa repo).
- **Khớp dữ liệu:** 10/10 — không cần bịa giờ.

### PA2 — Heatmap Ngày × Công đoạn
Lưới ngày (cột) × công đoạn/tổ (hàng), mỗi ô tô đậm theo `%` hoàn thành ngày.

- **Điểm mạnh:** cực gọn cho nhiều ngày; bắt "vùng nghẽn" nhanh.
- **Điểm yếu:** **mất** ẩn dụ span KH↔thực tế và quan hệ trước/sau công đoạn; khó gắn từng PO; xa đặc tả gốc.
- Kết luận: **giữ làm lớp phụ** (mini-heatmap trong tooltip hoặc chế độ "mật độ"), không làm màn hình chính.

### PA3 — Bảng-thanh (table + inline bar)
Mỗi hàng PO/công đoạn, cột số (KH, đạt, phế, %, trễ) + 1 thanh tiến độ nhỏ.

- **Điểm mạnh:** đọc số chính xác; dễ dựng bằng APEX IR/IG.
- **Điểm yếu:** **không** thấy trục thời gian/hành trình nhiều ngày; đây là *báo cáo*, không phải *điều độ trực quan*.
- Kết luận: bổ trợ (drill-down), không thay Gantt.

> **Chốt:** **PA1 (Gantt nhiều-ngày)** làm màn hình chính; PA2/PA3 là lớp bổ trợ khi cần.

## 3. Quy tắc render mới (thay §"Quy ước render" của doc 07 cho chế độ theo-ngày)

- **Phạm vi:** một **cửa sổ nhiều ngày** (mặc định 7 ngày quanh ngày chọn; cấu hình `dayWindow`).
  Trục = các cột-ngày, mỗi cột 1 ngày. **Không** trục giờ.
- **Thanh PO:** từ cột `plan_start_day` → `plan_end_day`. Track KH tô nhạt; **fill** đậm theo `progress` lũy kế.
- **Ô-ngày:** mỗi ngày trong span có thể mang badge đạt/phế (bản ghi ngày đó). Ngày **chưa có bản ghi** ⇒
  ô rỗng nhạt + "—" (khác ngày làm ra 0).
- **Màu thanh (exception-driven):** `ontrack`/đang chạy → trung tính (accent xanh dương cho fill);
  `progress≥1` → xanh lá (xong); `delayed` (trễ ngày) → đỏ. Không tô đỏ khi bình thường.
- **Cột "Hôm nay":** highlight cột ngày hiện tại (thay Current Time Indicator theo phút).
- **Stacking:** ≥2 PO cùng tổ **trùng ngày** → tách dòng (song song hợp lệ); ≥4 → cluster "+N".
- **Trễ:** `late_days = ngày hiện tại − plan_end_day` (nếu >0 và progress<1) → PO đỏ + nhãn "Chậm N ngày".
- **Dòng chảy ngược (theo ngày):** cùng PO, nếu `%` công đoạn sau > công đoạn trước tại cùng ngày → cảnh báo.

## 4. Data-contract THEO NGÀY (thay bản 24h ở doc 07)

Region SQL vẫn trả **1 CLOB** `{ "data":[...], "links":[] }`, phân cấp qua `row_type` + `id`/`parent`.
Khác biệt: **bỏ trường phụ thuộc giờ**, thêm trường theo-ngày.

### Node STAGE (công đoạn) — giữ như cũ
| Trường | BB | Ý nghĩa |
|--------|----|---------|
| `id`, `parent`(=0), `row_type`="stage", `text` | ✓ | như doc 07 |
| `stage_load` | ✓ | % tải công đoạn (ngày) → Production Health |

### Node MACHINE (tổ máy)
| Trường | BB | Ý nghĩa |
|--------|----|---------|
| `id`, `parent`, `row_type`="machine", `text` | ✓ | như doc 07 |
| `actual_qty`, `plan_qty` | ✓ | lũy kế trong cửa sổ ngày → nhãn `[actual/plan]` |
| `status` | ✓ | đèn `"g"`/`"o"`/`"r"` |

### Node PO (lệnh sản xuất) — **ĐỔI trục sang ngày**
| Trường | BB | Ý nghĩa |
|--------|----|---------|
| `id`, `parent`, `row_type`="po", `text` | ✓ | như doc 07 |
| `plan_start` | ✓ | **ngày** KH bắt đầu `"yyyy-mm-dd"` (❌ bỏ giờ) |
| `plan_end` | ✓ | **ngày** KH kết thúc `"yyyy-mm-dd"` |
| `progress` | ✓ | 0..1 lũy kế → fill + % |
| `po_state` | ✓ | `"ontrack"`/`"running"`/`"delayed"` |
| `item_name`, `order_number`, `quantity` | ✓ | như cũ |
| `daily` | ✓ | **mảng bản ghi theo ngày** (xem dưới) — cốt lõi mô hình mới |
| `late_days` |  | số ngày trễ (server tính sẵn hoặc renderer suy từ `plan_end`) |
| `material_short` |  | `1`/`0` (JSON number, không phải chuỗi) |

#### `daily[]` — 1 phần tử = 1 bản ghi nhật ký/ngày
```jsonc
"daily": [
  { "d": "2026-07-06", "ok": 520, "scrap": 12, "cum": 520, "by": "Nguyễn Văn A" },
  { "d": "2026-07-07", "ok": 610, "scrap":  8, "cum": 1130, "by": "Nguyễn Văn A" }
  // ngày KHÔNG có phần tử = chưa nhật ký ⇒ ô "—" (D8)
]
```
| Trường | BB | Ý nghĩa |
|--------|----|---------|
| `d` | ✓ | ngày bản ghi `"yyyy-mm-dd"` (đúng 1 bản ghi/ngày) |
| `ok` | ✓ | SL đạt (thành phẩm) ngày đó |
| `scrap` | ✓ | SL phế ngày đó |
| `cum` |  | lũy kế đạt đến hết ngày (nếu thiếu, renderer tự cộng dồn) |
| `by` |  | người ghi nhận |

## 5. Cấu hình renderer (chế độ theo-ngày)
| Config | Mặc định | Ghi chú |
|--------|----------|---------|
| `renderMode` | `"day"` | phân biệt với `"hour"` (24h cũ) |
| `dayWindow` | 7 | số cột-ngày hiển thị |
| `anchorDate` | SYSDATE | ngày trung tâm cửa sổ |
| `narrowPx` | 96 | ngưỡng degrade nhãn thanh hẹp (span ngắn) |
| `clusterMin` | 4 | số dòng để gộp cluster |
| `scrapWarn` | 0.03 | ngưỡng tỉ lệ phế bật màu |

> Tương thích ngược: thiếu trường tuỳ chọn → renderer dùng giá trị an toàn. `daily` rỗng ⇒ PO hiện track KH
> nhưng mọi ô-ngày là "—" (chưa nhập), progress=0.

## 6. Bàn giao sang Pha 14 (UX spec)
Chốt: layout màn hình (grid cột-ngày + cột nhãn cố định), thành phần, trạng thái (empty/loading/lỗi),
quy tắc degrade nhãn, tooltip đạt/phế theo ngày, và mapping token màu exception-driven.
