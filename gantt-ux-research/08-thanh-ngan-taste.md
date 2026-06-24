# Nghiên cứu (taste): xử lý thanh PO quá ngắn

## Dữ kiện (từ người dùng)
- Lệnh siêu ngắn: **hiếm** (edge case).
- Thông tin tối thiểu cần thấy ngay trên lưới: **chỉ "có lệnh" + màu trạng thái**.
- Ưu tiên: **tương tác là chính** (chi tiết qua hover/click).
- Mật độ: **vừa, 6–15 lệnh/máy/ngày**.

## Phân tích taste
- **Nhãn ngoài bay cạnh thanh** = phản gu: thêm nhiễu thị giác, và là nguồn gốc của việc đè nhau khi mật độ vừa. Vi phạm tiêu chí "lưới sạch, tương tác là chính".
- Vì thông tin tối thiểu chỉ là *hiện diện + màu*, ta **không cần** ép chữ ra ngoài. Chữ nên **nằm trong thanh và tự co**; hết chỗ thì **ẩn**, không tràn ra ngoài.
- Lệnh ngắn hiếm → **marker** là đủ; không cần bảng kèm hay min-width gây sai tỉ lệ lịch.

## Giải pháp chốt — "In-bar auto-fit + Marker + Hover"
Bỏ hẳn floating label. Mỗi thanh tự quyết theo bề rộng thật:

| Bề rộng thật | Hiển thị trong thanh |
|---|---|
| ≥ 150px | Mã + % (dòng 1) **và** tên sản phẩm (dòng 2) |
| 70–150px | Chỉ Mã + % (ẩn dòng sản phẩm) |
| 20–70px | **Không chữ** — chỉ thanh màu trạng thái (đủ "có lệnh + màu") |
| < 20px | **Marker** ◆ — chip nhỏ cố định, màu trạng thái, luôn bấm/hover được |

- **Mọi chi tiết** (sản phẩm, đơn hàng, SL, giờ, trạng thái, NVL) → **popover khi hover** (đã có), và `title` gốc cho a11y.
- Không còn nhãn ngoài → **không còn bài toán đè nhau**; lưới sạch, đúng "tương tác là chính".
- Giữ **lịch trung thực** (không phóng to theo thời gian); chỉ marker mới có min-width để bấm được.

## Hệ quả triển khai
- Xoá logic tạo `.po-ext` + suy biến nhiều tầng; thay bằng toggle class trong thanh: `.hide-sub`, `.text-none`, `.marker`.
- Popover trở thành kênh thông tin chính (đã sẵn). Click mở panel/APEX để giai đoạn sau.
