# Pha 5 — Drawer ghi nhận sản xuất: User Story & Điểm đau (bmad-method)

> Nguồn: `mockups/po-detail-drawer.html` (drawer khi click PO) + renderer `mes-control-tower.js`
> (event `mesgantt_po_click` / `onPoClick` đã có; harness **chưa** wiring drawer).
>
> **Phạm vi đã chốt (Giai đoạn 0):** người dùng chính = **điều độ viên / desktop**; vai trò drawer =
> **cân bằng xem + nhập**; tần suất ghi nhận = **vài lần/ca**; cách nhập số = **gõ tay bàn phím**.
> Hình thức = **panel trượt phải** (theo doc 03), không rời màn Gantt.

## Personas (đào sâu từ doc 03 cho luồng ghi nhận)

- **P1 — Điều độ viên (primary):** click PO trên Gantt để vừa **đọc nhanh tình trạng** (tiến độ, còn lại,
  trễ/NVL) vừa **ghi nhận** thành phẩm/phế phẩm vài lần trong ca. Dùng chuột + **bàn phím**, mong muốn
  gõ ít nhất có thể, không rời màn điều độ.
- **P2 — Tổ trưởng:** thỉnh thoảng ghi hộ/đối chiếu số liệu tổ mình; quan tâm lịch sử ghi nhận.
- **P3 — Quản lý xưởng:** chủ yếu **xem** chi tiết + lịch sử (read-only theo vai trò), không nhập.

## User stories (MoSCoW)

| # | Story | Persona | Ưu tiên |
|---|-------|---------|---------|
| DS1 | Là điều độ viên, tôi click PO → drawer **trượt phải** mở ngay, không che lưới Gantt, để vừa nhập vừa thấy bối cảnh | P1 | Must |
| DS2 | Là điều độ viên, mở drawer tôi **thấy ngay 3 con số quyết định**: tiến độ %, SL còn cần SX, trạng thái (đúng/trễ/NVL) — không phải tự ghép | P1 | Must |
| DS3 | Là điều độ viên, khi ghi nhận tôi muốn **giờ tự điền** (mặc định = hiện tại), khỏi gõ tay mỗi lần | P1 | Must |
| DS4 | Là điều độ viên, sau khi lưu 1 dòng tôi muốn form **tự sạch + con trỏ về ô Thành phẩm**, lịch sử cập nhật ngay, để ghi dòng kế không chạm chuột | P1 | Must |
| DS5 | Là điều độ viên, tôi muốn **lũy kế + SL còn lại tự tính lại** ngay khi gõ, để biết đã đủ chưa mà không nhẩm tay | P1 | Must |
| DS6 | Là điều độ viên, tôi muốn được **chặn/cảnh báo tại chỗ** khi nhập sai (âm, vượt kế hoạch, tổng=0) trước khi lưu | P1 | Must |
| DS7 | Là điều độ viên, nếu đóng drawer khi đang gõ dở, tôi muốn được **hỏi xác nhận** để không mất dữ liệu | P1 | Must |
| DS8 | Là điều độ viên, tôi muốn **toàn bộ luồng đi bằng bàn phím**: Tab nhảy đúng thứ tự (bỏ ô read-only), Enter = Lưu & thêm, Esc = đóng | P1 | Should |
| DS9 | Là điều độ viên, lý do phế phẩm hay lặp lại nên tôi muốn **chọn nhanh từ preset** thay vì gõ lại ghi chú | P1 | Should |
| DS10 | Là quản lý, tôi muốn mở cùng drawer ở **chế độ chỉ xem** (ẩn form nhập), gọn cho lãnh đạo | P3 | Should |
| DS11 | Là điều độ viên, khi lưu lỗi (mất mạng) tôi muốn **báo rõ + giữ lại số đã gõ** để thử lại, không mất công | P1 | Should |
| DS12 | Là người dùng, tôi muốn **lịch sử ghi nhận cuộn được** và thấy ai/giờ/số, để đối chiếu cuối ca | All | Could |

## Đi qua luồng hiện tại (mockup) → điểm đau

Luồng: *click PO → đọc khối "Thông tin SX" → form "Thông tin ghi nhận" → "Lưu & thêm" → xem "Tổng quan" + "Lịch sử" → đóng.*

| # | Điểm đau (nhập lặp / thao tác thừa) | Vì sao khó chịu | Cách khắc phục (đưa vào mockup Giai đoạn 3) |
|---|-------------------------------------|-----------------|---------------------------------------------|
| F1 | Ô **Thời gian định dạng `08-24-2024 14:30`** (MM-DD-YYYY) | Lẫn với chuẩn `DD-MM-YYYY` của cả dự án → dễ đọc/gõ sai; khi bỏ tick "dùng giờ hiện tại" phải gõ chuỗi dài | Mặc định auto giờ hiện tại; khi sửa dùng `DD-MM-YYYY HH24:MI` + time-picker, **không gõ chuỗi tự do** |
| F2 | **Sau "Lưu & thêm" không rõ con trỏ về đâu** | Ghi vài lần/ca mà mỗi lần phải click lại ô Thành phẩm = thao tác thừa | Auto-clear số + **autofocus ô Thành phẩm**, giữ giờ tự cập nhật |
| F3 | **Hai nút "Lưu & thêm" và "⊕ Thêm dòng"** cạnh nhau, nghĩa chồng lấn | Không rõ khác gì → do dự, bấm nhầm | Gộp còn **1 hành động chính "Lưu (Enter)"**; "thêm dòng" chỉ là kết quả của Lưu |
| F4 | **Lũy kế / SL còn lại / tiến độ % là số tĩnh** | Đang gõ phải tự nhẩm "ghi nữa thì còn bao nhiêu" | **Tính lại realtime** khi gõ: hiển thị "sau khi lưu: còn lại X, tiến độ Y%" |
| F5 | **Ghi chú là ô trống mỗi lần** | Lý do phế phẩm hay lặp → gõ lại tốn công | Preset lý do phế phẩm bấm-1-chạm (DS9) + cho nhập tự do khi cần |
| F6 | **Ô Thời gian read-only vẫn nằm trong Tab order** | Tab bằng bàn phím dừng ở ô vô nghĩa | Bỏ ô read-only khỏi tab; Tab: Thành phẩm → Phế phẩm → Lý do → Lưu |
| F7 | **Khối "Tổng quan" tách rời khối nhập** | Đang nhập phải liếc lên-xuống để biết tác động | Đặt số "còn lại / tiến độ" **ngay cạnh ô nhập** hoặc cập nhật nổi bật khi gõ |
| F8 | **Không có phím tắt** | Điều độ viên desktop thao tác nhanh bị buộc dùng chuột | Enter=Lưu, Esc=đóng (có guard F7-exception), ↑↓ trên ô số |
| F9 | **Lịch sử không thấy "còn lại sau mỗi lần"** rõ ràng | Khó kiểm tra mạch ghi nhận | Cột lũy kế TP đã có; thêm trạng thái mốc (đạt/vượt) màu theo ngưỡng |

## Exception cần xử lý (chưa có trong mockup)

| # | Tình huống | Hành vi mong muốn |
|---|-----------|-------------------|
| E1 | Nhập **số âm** (TP hoặc PP) | Chặn nhập / báo đỏ tại ô, không cho Lưu |
| E2 | **Tổng TP và PP = 0** rồi bấm Lưu | Chặn no-op, nhắc "chưa nhập số lượng" |
| E3 | **TP lũy kế vượt SL kế hoạch** (vd >3.000) | ✅ **CHỐT: cảnh báo nhưng vẫn cho lưu.** Hiện "vượt kế hoạch +N — xác nhận?" màu cảnh báo; người dùng xác nhận thì lưu (sản xuất dư là hợp lệ). KHÔNG chặn cứng |
| E4 | **Ghi làm tiến độ vượt 100%** | Cùng nhóm E3; hiển thị rõ, cho phép nếu xác nhận |
| E5 | **PP > SL tiếp nhận** (yield âm vô lý) | Cảnh báo logic, yêu cầu xác nhận |
| E6 | **Trùng mốc giờ** với bản ghi đã có | Cảnh báo "đã có ghi nhận lúc HH:MM" — cho ghi đè/cộng dồn/hủy |
| E7 | **Đóng drawer khi đang gõ dở chưa lưu** | Hỏi xác nhận "Bỏ thay đổi chưa lưu?" (chặn Esc/click-ngoài/nút ×) |
| E8 | **Lưu lỗi (mất mạng / server)** | Báo thân thiện, **giữ lại số đã gõ**, nút "Thử lại"; không xóa form |
| E9 | **PO đã hoàn thành/đóng** | Mở ở chế độ xem, ẩn/khóa form nhập, ghi rõ "đã hoàn thành" |
| E10 | Số có **dấu phân tách nghìn** (`3,000`) hoặc khoảng trắng khi gõ | Chuẩn hóa parse về số nguyên; không vỡ tính toán |
| E11 | **Mốc giờ ở tương lai** (gõ sai giờ) | Cảnh báo "giờ ghi nhận ở tương lai" |
| E12 | **Quản lý (P3) mở** drawer | Chế độ read-only: ẩn form + nút Lưu, chỉ còn xem + lịch sử (DS10) |

## Acceptance criteria (chốt cho thiết kế Giai đoạn 3)

- [ ] AC1 — Mở drawer trong ≤1 thao tác từ Gantt; panel **trượt phải**, lưới Gantt vẫn thấy.
- [ ] AC2 — Mở ra thấy ngay **tiến độ %, SL còn cần SX, trạng thái** (DS2) không cần cuộn.
- [ ] AC3 — Giờ ghi nhận **mặc định auto hiện tại**; định dạng hiển thị `DD-MM-YYYY HH24:MI` (F1).
- [ ] AC4 — Sau Lưu: form sạch số, **autofocus ô Thành phẩm**, lịch sử + lũy kế cập nhật ngay (DS4/F2).
- [ ] AC5 — Lũy kế, SL còn lại, tiến độ **cập nhật realtime khi gõ** (DS5/F4/F7).
- [ ] AC6 — **Toàn luồng bàn phím**: Tab bỏ ô read-only, Enter=Lưu, Esc=đóng-có-guard (DS8/F6/F8).
- [ ] AC7 — Validate tại chỗ E1/E2 (chặn), E3/E4/E5/E6/E11 (cảnh báo + xác nhận).
- [ ] AC8 — Guard đóng khi chưa lưu (E7); lưu lỗi giữ dữ liệu + thử lại (E8).
- [ ] AC9 — Chế độ **chỉ xem** cho P3 / PO đã đóng (E9/E12/DS10).
- [ ] AC10 — Chỉ làm mockup HTML/CSS standalone trong `mockups/`; tôn trọng locked UX (màu theo ngưỡng,
  exception-driven; chữ/khối scope class; token trên wrapper).

---

→ **CHECKPOINT GIAI ĐOẠN 1.** Mời bạn duyệt user stories + bảng điểm đau/exception. Khi OK, sang
**Giai đoạn 2 — Nghiên cứu UI/UX (research-1.0.0 + taste)** cho form nhập liệu nhanh trên desktop.
