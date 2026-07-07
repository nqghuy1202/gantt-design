# Pha 12 — Phân tích: Gantt tiến độ sản xuất cho nhật ký GHI THEO NGÀY (bmad-analyst)

> **Analyst (Mary):** phân tích lại yêu cầu sơ đồ Gantt khi dữ liệu nhật ký sản xuất **thực tế** chỉ có
> **1 bản ghi / lệnh / công đoạn / ngày**, không có mốc giờ–phút. Mục tiêu: tách bạch **điều gì đặc tả gốc
> đòi hỏi nhưng dữ liệu không cấp được**, và **điều gì dữ liệu theo-ngày cấp được một cách trung thực**.
>
> **Quyết định đã khóa (vòng này):** độ phân giải dữ liệu **cố định theo NGÀY**; trục thời gian chuyển sang
> **NHIỀU NGÀY (lịch)**; đầu ra = nghiên cứu + 1 mockup HTML. Xem prompt điều phối ở đầu phiên.

## 1. Mâu thuẫn cốt lõi

Hai đặc tả gốc (`TÀI LIỆU ĐẶC TẢ … SƠ ĐỒ GANT THEO CÔNG ĐOẠN` và `… GHI NHẬN NHẬT KÝ`) đều **giả định
độ phân giải giờ–phút**:

- Gantt đặt thanh PO trên **trục cố định 24h**, vị trí + độ dài thanh = **giờ bắt đầu → giờ kết thúc**.
- `%` hoàn thành **chạy dần trong ngày** sau mỗi lần ghi nhận.
- **Current Time Indicator**: đường kẻ dọc đỏ theo **thời điểm thực tại đến phút**.
- Ghi nhận "theo **từng lần phát sinh** trong ngày", trường Thời gian `DD-MM-YYYY HH-MM`, lũy kế sau mỗi lần.
- Cảnh báo **dòng chảy ngược** (công đoạn sau > công đoạn trước) đánh giá **tại cùng một thời điểm**.

**Thực tế của hệ thống:** mỗi lệnh chỉ sinh **1 bản ghi/ngày** cho một công đoạn — một con số **đạt / phế
tổng cuối ngày**, không có timestamp trong ngày có ý nghĩa.

> Hệ quả: nếu bê nguyên mô hình Gantt-theo-giờ, giao diện sẽ **bịa ra độ chính xác không tồn tại** — đặt
> thanh vào 07:00–10:00 trong khi dữ liệu không biết giờ nào. Đây là lỗi trung thực dữ liệu, không phải lỗi CSS.

## 2. Nguyên tắc dẫn đường

> **Đơn vị nhỏ nhất của trục phải bằng đơn vị nhỏ nhất của dữ liệu.**
> Dữ liệu = 1 bản ghi/ngày ⇒ ô nhỏ nhất của trục = **1 ngày**. Không vẽ nhỏ hơn ngày.

Đây là mấu chốt biến "mâu thuẫn" thành "khớp hoàn hảo": khi trục là **cột-ngày**, **mỗi ô-ngày của một PO
ứng đúng 1 bản ghi nhật ký**. Không stack sub-hour, không nội suy, không giả định giờ.

## 3. Điều gì MẤT khi bỏ trục giờ (và cách xử lý)

| Tính năng đặc tả gốc | Vì sao mất với dữ liệu ngày | Xử lý trong thiết kế mới |
|---|---|---|
| Thanh đặt theo **giờ bắt đầu/kết thúc** | Không có giờ | Thanh trải theo **ngày** KH bắt đầu → KH kết thúc (cột-ngày) |
| `%` **chạy trong ngày** | Chỉ có 1 số cuối ngày | `%` cập nhật theo **ngày**; trong 1 ngày là 1 bước nhảy, không mượt |
| **Current Time Indicator** theo phút | Vô nghĩa trong 1 ô-ngày | Đổi thành **cột "Hôm nay"** (đánh dấu ở mức ngày) |
| **Chậm N phút** | Không đo được phút | Đổi thành **chậm N ngày** (KH kết thúc vs ngày hiện tại/thực xong) |
| **Dòng chảy ngược tại 1 thời điểm** | Không có timeline trong ngày | Vẫn phát hiện được, nhưng **so theo ngày** (thực ra hợp lý hơn: so lũy kế cuối ngày giữa 2 công đoạn) |
| **Sequencing nhiều PO trong 1 máy theo giờ** | Không biết PO nào chạy trước trong ngày | Không thể hiện thứ tự trong ngày; hiển thị **tổng đạt/phế theo ngày của máy**; 2 PO cùng ngày/máy = ô-ngày song song (overlap hợp lệ, quy tắc cũ giữ nguyên) |

## 4. Điều gì CÒN (dữ liệu theo-ngày cấp được trung thực)

- **Phân cấp công đoạn → tổ máy → PO** — giữ nguyên như đặc tả.
- **Theo từng ngày** cho mỗi PO: SL đạt, SL phế, KH ngày, `%` lũy kế đến ngày đó, Yield ngày.
- **Hành trình nhiều ngày của 1 PO** qua các công đoạn: công đoạn bắt đầu ngày nào, tiến triển, xong ngày nào.
- **Kế hoạch vs Thực tế theo ngày**: KH span (nhạt) vs tiến độ thực (đậm) → lệch vị trí = trễ.
- **Production Health** (tải công đoạn theo ngày) & **KPIs lũy kế** (Orders, Qty plan/actual, Yield, FCST%)
  — tính từ tổng theo ngày, không cần giờ.

## 5. Personas & khoảnh khắc sử dụng (kế thừa doc 03/09)

- **P1 — Điều độ viên (primary):** mở màn hình đầu ca, quét **tuần này**: PO nào trễ ngày, công đoạn nào
  đang nghẽn, tổ nào chưa nhập số hôm qua. Quyết định điều phối theo **ngày**, không theo phút.
- **P2 — Tổ trưởng:** đối chiếu đạt/phế **theo ngày** của tổ mình so KH ngày.
- **P3 — Quản lý xưởng:** nhìn xu hướng nhiều ngày, phát hiện PO tụt tiến độ tích lũy.

## 6. User stories (MoSCoW)

| # | Story | Persona | Ưu tiên |
|---|-------|---------|---------|
| D1 | Là điều độ viên, tôi thấy **trục là các NGÀY** để nắm tiến độ theo lịch, không bị con số giờ-phút giả | P1 | Must |
| D2 | Là điều độ viên, mỗi **ô-ngày của PO** cho tôi đạt/phế **đúng 1 bản ghi nhật ký ngày đó** | P1 | Must |
| D3 | Là điều độ viên, tôi phân biệt ngay **KH (span nhạt)** với **thực tế (fill đậm)** để thấy PO đang trễ ngày | P1 | Must |
| D4 | Là điều độ viên, PO **trễ ngày** mới tô đỏ; bình thường trung tính (exception-driven) | P1 | Must |
| D5 | Là tổ trưởng, tôi thấy **tổng đạt/phế theo ngày** ở dòng tổ máy, không cần thứ tự giờ | P2 | Should |
| D6 | Là quản lý, tôi liếc **xu hướng nhiều ngày** của 1 PO qua các công đoạn | P3 | Should |
| D7 | Là điều độ viên, **cột "Hôm nay"** rõ ràng ở mức ngày để định vị | P1 | Should |
| D8 | Là điều độ viên, **ngày chưa có bản ghi** phải khác ngày có số 0 (chưa nhập ≠ làm ra 0) | P1 | Must |
| D9 | Là quản lý, hệ thống **cảnh báo dòng chảy ngược theo ngày** (công đoạn sau vượt công đoạn trước) | P3 | Could |

## 7. Ràng buộc trung thực dữ liệu (đưa vào acceptance)

- **KHÔNG** hiển thị bất kỳ trục/nhãn giờ-phút nào (giờ, HH:MM, thanh sub-day).
- **Chưa nhập** ⇒ hiển thị trung tính "—", **không** hiển thị 0 (phân biệt D8).
- `%` là **bước nhảy theo ngày**, không animate liên tục kiểu realtime.
- Overlap 2 PO cùng máy cùng ngày = **song song hợp lệ**, không tô đỏ (giữ quy tắc repo).

## 8. Bàn giao sang Pha 13 (research)

Câu hỏi mở cho nghiên cứu: **hình thức trực quan nào tối ưu cho dữ liệu theo-ngày** — Gantt-nhiều-ngày
(đã chọn hướng) so với các biến thể (heatmap ngày×công đoạn, bảng-thanh). Chốt data-contract mới theo-ngày,
loại bỏ trường phụ thuộc giờ (`start_date` giờ, `duration` giờ), thay bằng ngày.
