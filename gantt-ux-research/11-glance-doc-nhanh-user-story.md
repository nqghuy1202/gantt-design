# Pha (glance) — Khối "Đọc nhanh" `.posd-glance`: User Story & Điểm đau (bmad-method)

> Nguồn: `mockups/po-detail-drawer-v2.html` — khối **`.posd-glance`** (3 stat card) + **`.posd-stagestrip`**
> (4 cặp số) luôn hiển thị **trên cả 2 tab** "Ghi nhận | Lịch sử". Đây là vùng người dùng **đọc trước
> khi nhập** — không nhập liệu, chỉ truyền đạt tình trạng PO.
>
> **Phạm vi đã chốt (Giai đoạn 0):**
> - 3 số glance **ngang nhau** — không nhấn trội 1 số; tối ưu để cả 3 quét được nhanh, đồng đều thị giác.
> - Stage strip **giữ nguyên 4 số** (Công đoạn·Tổ / Đạt yêu cầu / Phế phẩm / Đã chuyển giao) — chỉ làm đẹp.
> - Phạm vi implement = **visual + sắp xếp lại** (đổi màu/spacing/thứ bậc + sắp lại thứ tự); **giữ JS logic & ID**.
> - Bright palette đã áp ([[gantt-po-drawer]]): primary xanh lá #15674C = hành động, accent xanh dương
>   #1D4ED8 = dữ liệu/tiến độ, màu semantic exception-driven, font Inter, shadow mềm harness.

## Vùng đang xét (hiện trạng)

Card glance (grid 3 cột):
1. **Tiến độ** — `%` to + progress bar (`#gProgress`, `#gProgressBar`)
2. **Còn cần SX** — số to + dòng phụ `/ KH 3.000` (`#gRemain`)
3. **Trạng thái** — text "Đúng tiến độ" + dòng phụ "NVL đủ" (`#gState`, `#gStateSub`)

Stage strip (4 cặp k/value): Công đoạn·Tổ (`Mixing · Team 2`) · Đạt yêu cầu (`#sOk`) · Phế phẩm (`#sScrap`) · Đã chuyển giao (`#sDeliver`).

## Personas (kế thừa doc 03/09)

- **P1 — Điều độ viên (primary):** liếc glance **trong <2s** để quyết định PO này có cần can thiệp không,
  rồi mới chuyển xuống tab ghi nhận. Đọc bằng mắt khi tay đang ở bàn phím.
- **P2 — Tổ trưởng:** đối chiếu nhanh số Đạt/Phế của tổ mình.
- **P3 — Quản lý xưởng:** liếc tình trạng + tỉ lệ phế, không nhập.

## User stories (MoSCoW) — tập trung khoảnh khắc đọc-nhanh

| # | Story | Persona | Ưu tiên |
|---|-------|---------|---------|
| G1 | Là điều độ viên, mở drawer tôi **quét được cả 3 số (tiến độ / còn cần SX / trạng thái) trong <2s**, không phải dừng lại đọc từng cái | P1 | Must |
| G2 | Là điều độ viên, 3 số này **quan trọng ngang nhau** nên tôi muốn chúng **cân bằng thị giác** (cùng cỡ, cùng nhịp), không cái nào "hút mắt" lệch | P1 | Must |
| G3 | Là điều độ viên, tôi muốn **chỉ thấy màu khi có vấn đề** (chậm tiến độ, thiếu NVL, phế cao) — lúc bình thường mọi số trung tính để mắt không bị nhiễu | P1 | Must |
| G4 | Là điều độ viên, mỗi số phải **tự giải nghĩa**: thấy "Còn cần SX 1.200" phải hiểu ngay là còn lại so với KH 3.000, không phải đoán | P1 | Must |
| G5 | Là điều độ viên, sau khi tôi **lưu 1 dòng ghi nhận**, các số glance phải **cập nhật ngay + báo hiệu nhẹ** (đã đổi), để tôi tin số đang sống | P1 | Must |
| G6 | Là điều độ viên/quản lý, tôi muốn **trạng thái là chỉ báo rõ ràng** (đúng/chậm + lý do) chứ không chỉ một chữ chung chung | P1 | Should |
| G7 | Là điều độ viên, stage strip 4 số phải **đọc theo cặp nhãn–giá trị mạch lạc**, phân biệt được Đạt vs Đã chuyển giao (dễ tưởng trùng) | P1 | Should |
| G8 | Là quản lý, tôi muốn **tỉ lệ phế** nổi lên khi vượt ngưỡng để bắt bất thường mà không phải tự chia | P3 | Could |

## Đi qua khối hiện tại → điểm đau (friction)

| # | Điểm đau | Vì sao khó chịu | Cách khắc phục (visual + sắp xếp) |
|---|----------|-----------------|-----------------------------------|
| FG1 | **Card "Tiến độ" có progress bar, 2 card kia không** → 3 ô lệch chiều cao/trọng lượng dù "ngang hàng" | Mắt bị kéo về card có bar → phá yêu cầu G2 (cân bằng) | Cân bằng nhịp: cho mỗi card một "dòng phụ" đồng cấp (card Còn cần SX & Trạng thái cũng có baseline phụ), hoặc hạ bar thành chi tiết tinh tế để 3 card cùng khối lượng |
| FG2 | **"Còn cần SX" và "Tiến độ" là hai mặt của cùng dữ liệu** trình bày tách rời | Đọc 2 lần cho 1 thông tin (60% ↔ còn 1.200) | Giữ cả hai (đã chốt) nhưng **buộc chúng về cùng mạch đọc**: dòng phụ thống nhất "X / KH Y" để quan hệ hiển nhiên |
| FG3 | **Trạng thái là text dài "Đúng tiến độ"** giữa 2 card toàn số | Lệch loại dữ liệu → nhịp quét gãy | Chuẩn hoá: trạng thái dùng **chip/indicator có dot màu** (đồng ngôn ngữ với header chip), ngắn gọn, dòng phụ nêu lý do |
| FG4 | **"NVL đủ" dòng phụ chữ thường** dễ bị bỏ qua khi thiếu NVL | Cảnh báo NVL chìm → bỏ lỡ rủi ro | Khi thiếu NVL → dòng phụ chuyển semantic (amber/đỏ) + dot; lúc đủ thì trung tính |
| FG5 | **Stage strip wrap 4 cặp tự do** (flex-wrap gap) → khi hẹp xuống dòng lộn xộn, nhãn 10.5px mảnh | Đọc cặp nhãn–value khó, Đạt/Đã chuyển giao dễ lẫn | Lưới cố định (2×2 hoặc 4 cột), tăng tương phản nhãn, group "Đạt → Phế → Đã chuyển" theo luồng SX |
| FG6 | **Sau lưu, số glance đổi nhưng không có tín hiệu** | Không chắc số đã cập nhật (G5) | Thêm pulse/flash nhẹ trên số vừa đổi (đồng bộ animation hàng flash lịch sử), tôn trọng `prefers-reduced-motion` |
| FG7 | **Phế phẩm chỉ tô đỏ khi vượt ngưỡng nhưng không nói ngưỡng** | Thấy đỏ mà không rõ vì sao | Tooltip `title` nêu ngưỡng (vd ">3%"), giữ exception-driven |
| FG8 | **Số to dùng `--ink`, không phân tầng nhãn/đơn vị** | "1.200" và "/ KH 3.000" gần nhau dễ đọc dính | Phân tầng cỡ/màu rõ giữa value chính (đậm, tabular) và đơn vị/denominator (mảnh, muted) |

## Exception cần thể hiện ở glance (không nhập liệu, chỉ hiển thị)

| # | Tình huống | Hành vi hiển thị mong muốn |
|---|-----------|----------------------------|
| EG1 | **Chậm tiến độ** (late > 0) | Card Trạng thái → chip amber/đỏ + dòng phụ "Chậm Nm"; đồng bộ `lateMeta` ở header |
| EG2 | **Thiếu NVL** | Dòng phụ Trạng thái → semantic + dot; không tô cả card |
| EG3 | **Tiến độ ≥ 100% / vượt KH** | Tiến độ tô `--ok`; "Còn cần SX" = 0 hiển thị "Đã đủ" thay vì số 0 trơ |
| EG4 | **Tỉ lệ phế > ngưỡng** (SCRAP_WARN 3%) | `#sScrap` → `--bad` + tooltip ngưỡng (logic đã có ở `refreshHeader`) |
| EG5 | **Số rất lớn** (vd 1.250.000) | tabular-nums + không vỡ layout; cân nhắc rút gọn ở dòng phụ nếu tràn |
| EG6 | **Dữ liệu thiếu** (chưa có ca/NVL) | Hiển thị "—" trung tính thay vì 0 hoặc trống, tránh hiểu nhầm |
| EG7 | **Vừa lưu xong** | Số đổi + flash nhẹ (FG6) |

## Acceptance criteria (cho glance sau tối ưu)

- **AG1** — 3 card glance **cân bằng thị giác**: cùng cỡ value, cùng chiều cao, cùng nhịp dòng phụ (G2/FG1).
- **AG2** — Quét hiểu cả 3 số **< 2s**, mỗi số tự giải nghĩa không cần đoán (G1/G4).
- **AG3** — **Không màu khi bình thường**; semantic (xanh-lá/amber/đỏ) chỉ bật đúng exception EG1–EG4, ≤3 số có màu cùng lúc (G3).
- **AG4** — Trạng thái dùng **chip + dot** đồng ngôn ngữ header; lý do (chậm/NVL) hiển thị ở dòng phụ (FG3/G6).
- **AG5** — Stage strip 4 số trên **lưới cố định**, phân biệt rõ Đạt vs Đã chuyển giao, nhãn đủ tương phản (G7/FG5).
- **AG6** — Sau Lưu, số glance **cập nhật + flash nhẹ**, tôn trọng `prefers-reduced-motion` (G5/FG6/EG7).
- **AG7** — Phân tầng value chính vs đơn vị/denominator rõ; `tabular-nums`; không vỡ với số lớn (FG8/EG5).
- **AG8** — **Không đổi** JS logic, ID (`gProgress/gRemain/gState/gStateSub/sOk/sScrap/sDeliver`), `refreshHeader`, hay hành vi tab/save. Token vẫn trên `.posd`.

## Hướng tối ưu đề xuất (chốt ở Pha 2)

1. **Đồng nhất 3 card** một "khuôn": `label → value (tabular) → dòng phụ/indicator` cùng cấp → cân bằng (AG1).
2. **Trạng thái thành chip có dot** (như header), dòng phụ là lý do động theo exception (AG4).
3. **Stage strip lưới 2×2 cố định**, nhóm theo luồng SX, nhãn rõ hơn (AG5).
4. **Flash nhẹ số vừa đổi** sau lưu (AG6) — chỉ CSS class do JS `refreshHeader` đã chạm các ID.
5. **Phân tầng typography** value/đơn vị, giữ accent xanh dương cho bar/tiến độ (AG7).

> Pha tiếp theo (Pha 2 — Propose): chốt layout cụ thể + token cho từng phần, chờ duyệt trước khi sửa code.
