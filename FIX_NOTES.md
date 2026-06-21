# Sửa lỗi: thiếu giọng đọc + thiếu hình ảnh Amazon

## Lỗi 1: Giọng đọc bị cắt cụt (đã sửa, cần thay file)

**Nguyên nhân thật:** Engine TTS tích hợp của HyperFrames (Kokoro-82M) có
giới hạn đã biết — nó KHÔNG tự chia nhỏ đoạn văn dài, và khi đưa cả đoạn
~100 từ vào, nó chỉ đọc vài giây đầu rồi dừng lại, không báo lỗi gì (đây là
hạn chế ai dùng Kokoro cũng gặp, có ghi trong nhiều issue của cộng đồng).

**Cách sửa:** Chia 4 đoạn lớn thành 24 câu/cụm ngắn (dưới 22 từ mỗi cụm),
gọi TTS riêng cho từng cụm, rồi ghép nối lại — y hệt cách các sản phẩm TTS
khác xử lý văn bản dài với Kokoro. Animation vẫn tự canh khớp theo tổng số
giây thật, không cần tính tay.

Đã test lại: 24 audio giả tỉ lệ theo số từ → lint 0 lỗi → composition ra
đúng tổng thời lượng theo công thức gộp 4 nhóm cảnh.

**Cần thay 3 file** trong repo:
1. `.github/workflows/render.yml` — 4 bước TTS riêng lẻ gộp thành 1 bước
   lặp qua 24 file.
2. `index.html` — 4 thẻ `<audio>` thay bằng chuỗi 24 thẻ nối tiếp.
3. `scripts/build_timing.py` — đo 24 file, gộp theo nhóm cảnh.

**Và thêm 24 file mới** vào `assets/`: `narration-01.txt` đến
`narration-24.txt` (xoá 4 file `narration-1.txt`..`narration-4.txt` cũ đi).

Workflow lần này sẽ có nhiều dòng log hơn (24 lần gọi TTS) nhưng gộp
chung trong 1 bước "Generate narration audio (24 short chunks, looped)" —
mở rộng bước đó ra sẽ thấy log từng câu.

## Lỗi 2: Hình ảnh — ĐÃ TỰ ĐỘNG HOÁ

Workflow giờ tự tải ảnh bìa thật từ URL Amazon (lấy thẻ `og:image` trên
trang sản phẩm), lưu vào `assets/cover.jpg`, hiển thị bên trái Scene 1.

- URL Amazon giờ là **input của workflow** — khi bấm "Run workflow" sẽ
  thấy 1 ô để dán URL (mặc định đã điền sẵn link cuốn Simple Christian).
  Lần sau làm sản phẩm khác chỉ cần đổi URL ở đây, không cần sửa file.
- Nếu Amazon chặn request (datacenter IP của GitHub đôi khi bị chặn khác
  với máy cá nhân) → tự động dùng 1 khối màu plum thay thế, KHÔNG làm
  hỏng video, chỉ là không có ảnh thật lần đó. Bước "Report cover image
  status" trong log sẽ ghi rõ ảnh thật hay ảnh dự phòng.
- Đây là cách lấy ảnh không chính thức (đọc thẳng trang sản phẩm) — không
  phải API chính thức của Amazon. Về lâu dài, khi tài khoản Associate của
  bạn đủ điều kiện (cần có giao dịch qua link affiliate), Amazon Product
  Advertising API sẽ là cách lấy ảnh ổn định, đúng quy định hơn. Báo mình
  khi đủ điều kiện, mình chuyển sang dùng API đó.

## Về "tự động hoá hoàn toàn"

Phần **cơ học** (ảnh, giọng đọc, ghép animation, render) giờ tự động hết
qua workflow này. Phần **viết script** (research sản phẩm, viết lời thoại)
mình đề xuất vẫn giữ người (mình) review trước khi đưa vào pipeline —
để tránh script tự sinh bịa thông tin sản phẩm khi không có ai kiểm tra.
Nếu bạn vẫn muốn tự động hoá luôn cả bước viết script (paste URL → ra
video hoàn chỉnh không cần duyệt), báo mình, đó là làm được nhưng nên cân
nhắc rủi ro thông tin sai trước khi bật full-auto.

