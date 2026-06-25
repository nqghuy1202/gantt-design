# Pha 6 — Nguyên tắc UI/UX cho Drawer ghi nhận (research-1.0.0 + taste)

> Tổng hợp ngắn, áp dụng được. Đầu vào: doc 09 (user story + điểm đau), locked UX (CLAUDE.md).
> **Design Read (taste 0.B):** *Đây là product-UI nhập liệu cho điều độ viên desktop — ngôn ngữ
> "công cụ điều hành gọn, đáng tin", nghiêng về data-entry panel: mật độ vừa-cao, chuyển động tối thiểu,
> bàn phím là chính.* (Không phải landing page — bỏ qua phần motion/hero của taste skill.)

## Dials (điều chỉnh khỏi baseline landing 8/6/4)

| Dial | Giá trị | Lý do |
|---|---|---|
| VISUAL_DENSITY | **6** | Điều độ viên desktop cần nhiều số liệu trong tầm mắt, ít cuộn |
| MOTION_INTENSITY | **2** | Công cụ làm việc lặp lại — chuyển động chỉ để phản hồi (mở drawer, lưu OK), không trang trí |
| DESIGN_VARIANCE | **3** | Form nhập liệu cần bố cục dự đoán được, nhất quán — không "nghệ thuật" |

## A. Nguyên tắc UI (lọc từ taste skill, hợp data-entry)

1. **Màu exception-driven** (trùng locked UX): nền trung tính, **tối đa 1 accent** (xanh primary của drawer);
   xanh/cam/đỏ chỉ xuất hiện khi **chạm ngưỡng** (vượt kế hoạch, phế phẩm cao, trễ). Thường ≤3 số có màu.
2. **Form chuẩn:** label **trên** input · helper text có sẵn trong markup · **error dưới** input · `gap` đều ·
   **không dùng placeholder làm label**. (taste 4.6)
3. **Shape consistency lock:** chọn **1 thang bo góc** dùng toàn drawer (đề xuất 8px input / 10px card / 8px nút).
4. **Contrast WCAG AA** (a11y bắt buộc): chữ trên nút, placeholder, focus ring, error text đều ≥4.5:1.
   Nút primary xanh + chữ trắng đạt; ô read-only nền xám phải đủ tương phản.
5. **Trạng thái tương tác đủ vòng** (taste 4.5): loading (skeleton dạng form, không spinner tròn) ·
   empty (PO chưa có ghi nhận) · **error inline** ở form (không toast cho lỗi cần sửa) · `:active` lún nhẹ.
6. **Icon thay emoji** (taste 3.C/3.D): mockup hiện dùng emoji 🕐⏱⛔ — thay bằng glyph icon nhất quán
   (1 bộ, strokeWidth cố định) cho chuyên nghiệp; giữ emoji chỉ khi thật cần.
7. **Card chỉ khi có phân cấp thật:** số liệu nhóm bằng `border`/khoảng trắng thay vì bọc card mọi thứ
   (tránh "card trong card").

## B. Nguyên tắc UX nhập liệu nhanh (research-1.0.0 + pattern ngành)

- **Inline-in-context (không rời màn):** drawer trượt phải, Gantt vẫn thấy → điều độ viên giữ bối cảnh.
  Pattern quen thuộc: side panel của Linear / Notion / Jira.
- **Giảm gõ tối đa (defaults > input):** auto điền giờ hiện tại; nhớ giá trị hợp lý gần nhất; con trỏ tự về ô
  số sau khi lưu. Mỗi ô người dùng **không phải gõ** là một friction được gỡ.
- **Bàn phím là chính:** autofocus đúng ô · Tab order bỏ ô read-only · **Enter = Lưu** · **Esc = đóng (có guard)** ·
  ↑/↓ chỉnh số. (đáp DS8/F6/F8)
- **Realtime feedback:** lũy kế / SL còn lại / tiến độ tính lại **ngay khi gõ**, đặt cạnh ô nhập (DS5/F4/F7).
- **Validate tách 2 mức rõ ràng:**
  - **Chặn cứng** (lỗi không hợp lệ): số âm, tổng=0 → không cho Lưu.
  - **Cảnh báo + xác nhận** (bất thường nhưng có thể đúng): vượt kế hoạch, tiến độ >100%, yield âm, trùng giờ,
    giờ tương lai → cho qua khi người dùng xác nhận, tô màu ngưỡng.
- **Recognition over recall:** lý do phế phẩm = preset bấm-1-chạm (DS9), không bắt gõ lại.
- **Forgiving (chống mất công):** xác nhận khi đóng lúc chưa lưu (E7) · lưu lỗi **giữ nguyên số đã gõ** + "Thử lại"
  (E8) · phản hồi thành công rõ (dòng mới highlight nhẹ trong lịch sử).
- **Read-only theo vai trò:** quản lý/PO đã đóng → ẩn form, chỉ còn xem + lịch sử (E9/E12/DS10).

## C. Severity các điểm đau (research-1.0.0 — ưu tiên xử lý ở Giai đoạn 3)

| Điểm đau / exception | Severity | Lý do |
|---|---|---|
| F2 con trỏ không về ô số sau Lưu · E7 mất dữ liệu khi đóng | **4 – Catastrophic** | Lặp vài lần/ca → mất công/mất số liệu, phải sửa trước khi ship |
| F1 định dạng giờ sai chuẩn · F3 hai nút mơ hồ · F4/F7 không realtime | **3 – Major** | Gây sai và chậm thường xuyên |
| E1/E2 chặn nhập sai · E3/E6 cảnh báo vượt/trùng · E8 giữ data khi lỗi | **3 – Major** | Tính đúng đắn dữ liệu |
| F5 ghi chú lặp · F8 thiếu phím tắt · F9 lịch sử thiếu mốc | **2 – Minor** | Tăng tốc, không chặn |
| Icon thay emoji · tinh chỉnh bo góc/màu | **1 – Cosmetic** | Hoàn thiện thẩm mỹ |

## D. Hướng layout đề xuất (để dựng ở Giai đoạn 3)

Drawer trượt phải, 3 vùng dọc rõ ràng (cân bằng xem ↔ nhập):

1. **Header** — mã PO + sản phẩm + chip trạng thái (màu theo ngưỡng) + nút đóng.
2. **Đọc nhanh (xem)** — 3 số quyết định nổi bật: **Tiến độ % · SL còn cần SX · Trạng thái**; chi tiết công đoạn/tổ
   thu gọn bên dưới.
3. **Ghi nhận (nhập)** — khối trọng tâm: giờ auto · ô **Thành phẩm** (autofocus) · **Phế phẩm** · preset lý do ·
   **1 nút "Lưu (Enter)"**; ngay cạnh hiển thị "sau khi lưu: còn lại X · tiến độ Y%".
4. **Lịch sử** — bảng cuộn, dòng mới highlight; cột lũy kế + mốc đạt/vượt theo màu ngưỡng.

Toàn bộ scope dưới 1 class wrapper, token khai báo trên wrapper (theo locked UX — chống vỡ khi dán vào APEX).

> **Quyết định đã chốt — E3 (vượt kế hoạch):** cảnh báo nhưng **vẫn cho lưu** sau khi xác nhận
> (sản xuất dư là hợp lệ), không chặn cứng. Thuộc nhóm "cảnh báo + xác nhận" ở mục B.

## E. A11y dialog + Checklist ship (gom từ ui-ux-pro-max + ui-skills)

Bổ sung các rule **triển khai cụ thể** mà mục A/B còn nói chung chung. Lấy *nguyên tắc*, không bê
class Tailwind/component React (drawer là vanilla CSS scoped trong APEX).

**Quản lý focus của drawer (ui-skills/fixing-accessibility — quan trọng nhất cho dialog):**
- **Bẫy focus** trong drawer khi mở (Tab không nhảy ra nền sau lưng).
- **Set initial focus** vào ô **Thành phẩm** khi mở (trùng AC4/DS4).
- **Trả focus** về đúng thanh PO đã click khi đóng drawer.
- **Esc đóng** drawer — nhưng đi qua guard E7 (hỏi nếu đang gõ dở).
- Mở drawer **không cuộn nền** bất ngờ.

**Announce động (ui-ux-pro-max/web):**
- Lỗi nhập dùng **`role="alert"`** (đọc ngay), không chỉ viền đỏ.
- Số tính realtime (lũy kế / còn lại / tiến độ) đặt trong vùng **`aria-live="polite"`** để báo cập nhật.

**Form & validate:**
- **`<label for>`** gắn từng ô; **không** placeholder-làm-label.
- **Validate on blur** cho hầu hết ô (không chỉ on submit); chặn cứng E1/E2 ngay khi rời ô.
- Error text **ngay cạnh** ô lỗi.

**Tactile & craft (ui-skills/baseline-ui):**
- **`AlertDialog`/hộp xác nhận** cho hành động không hoàn tác → dùng cho **E7 (đóng khi chưa lưu)**.
- **`tabular-nums`** cho mọi cột số (bảng lịch sử, lũy kế) → căn cột thẳng hàng.
- **Thang z-index cố định** (vd 10/20/30/50) cho drawer/overlay/popover, không z tùy tiện.
- Hiệu ứng **trượt drawer chỉ bằng `transform`** (không `left`/`width`), ≤200ms, `ease-out`, tôn trọng
  `prefers-reduced-motion`.
- **`:focus-visible`** ring rõ; **không** `outline:none` mà không thay thế.
- Nút icon (× đóng) có **`aria-label`**; icon trang trí `aria-hidden`.

**Checklist ship (cổng kiểm trước khi giao mockup Giai đoạn 3):**
- [ ] Không emoji làm icon (thay bằng SVG glyph 1 bộ).
- [ ] Mọi phần tử bấm được có con trỏ pointer + phản hồi hover.
- [ ] Contrast WCAG AA ≥4.5:1 (chữ, placeholder, error, focus ring, ô read-only).
- [ ] Focus thấy được + Tab đúng thứ tự (bỏ ô read-only).
- [ ] Focus trap + trả focus khi đóng; Esc có guard.
- [ ] `prefers-reduced-motion` được tôn trọng.
- [ ] Màu chỉ xuất hiện khi chạm ngưỡng (exception-driven), accent xanh nhất quán.
- [ ] Toàn bộ dưới 1 class wrapper + token trên wrapper.

> Sau khi dựng mockup, chạy `npx ui-skills get fixing-accessibility <file>` và `... baseline-ui <file>`
> làm pass audit cuối (báo vi phạm + sửa tối thiểu).

---

→ **CHECKPOINT GIAI ĐOẠN 2.** Mời bạn duyệt bộ nguyên tắc + hướng layout. Khi OK, sang
**Giai đoạn 3 — thiết kế lại mockup** `mockups/po-detail-drawer.html` (hoặc bản mới) theo các AC ở doc 09.
