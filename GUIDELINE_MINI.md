# Mini annotation guideline — Ngày 3 (tracking)

Nhóm / tên: **Lại Hoàng Duy — 2A202602271**

Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | xe máy / mô tô |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Xe đỗ thật trong cảnh vẫn được gán nếu có thể xác định là xe bốn bánh. Hình xe trên biển quảng cáo và vật thể tĩnh bị model nhận nhầm không được gán.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật áp dụng | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu bị che dưới 25 frame (2 giây ở 12.5 fps) và hướng chuyển động vẫn liên tục | Đây vẫn là cùng một vật thể trong cùng lần xuất hiện |
| Xe bị che lâu hơn 25 frame | Xem lại các frame trước/sau ở tốc độ chậm; chỉ giữ ID khi ngoại hình, quỹ đạo và vị trí cho phép xác nhận chắc chắn | Tránh nối nhầm hai xe giống nhau |
| Xe rời khung hình rồi quay lại | Tạo track mới | Không còn chuỗi quan sát liên tục để xác nhận identity |
| Hai xe cắt nhau / chồng lên nhau | Theo dõi hướng đi và đặc điểm ngoại hình của từng xe qua các frame trước–trong–sau giao cắt; không đổi ID | Vị trí tức thời có thể gây nhầm identity |

## 3. Luật bbox

| Tình huống | Luật áp dụng |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | Bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | Bbox ôm phần nhìn thấy được |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track từ frame đầu tiên có thể xác định chắc chắn là xe bốn bánh |
| Xe đang đỗ, không di chuyển | Vẫn gán nếu là xe thật trong cảnh; giữ ID xuyên suốt quãng thời gian xe còn nhìn thấy |
| Keyframe đặt dày ở đâu | Đặt dày quanh entry/exit, occlusion, crossing, đổi hướng, đổi tỷ lệ và các đoạn nội suy làm bbox trôi |

## 4. Ba ca mơ hồ đã gặp

### Ca 1

- Clip / frame / ID: `clip_01`, quanh frame 87, ID 5.
- Tình huống: tracker ReID đổi từ ID 17 sang 18 trong khi xe vẫn thuộc cùng một quỹ đạo.
- Quyết định: giữ một ID xuyên qua đoạn khó quan sát.
- Lý do: đối chiếu với gold cho thấy nhãn tay không có ID switch; model bị tách track tại đây.

### Ca 2

- Clip / frame / ID: `clip_01`, frame 103–105, ID 6.
- Tình huống: bbox xuất hiện sớm hơn thời điểm xe có thể xác định chắc chắn.
- Quyết định: kiểm tra lại điểm bắt đầu và đặt `outside`/endpoint đúng frame.
- Lý do: evaluator xác định ba bbox thừa trước khi track tham chiếu xuất hiện.

### Ca 3

- Clip / frame / ID: `clip_01`, quanh frame 109–110, ID 6.
- Tình huống: nội suy giữa các keyframe khiến bbox lệch khỏi phần xe nhìn thấy.
- Quyết định: thêm keyframe quanh đoạn xe thay đổi vị trí/hình học nhanh và kéo bbox sát phần nhìn thấy.
- Lý do: tại frame 110, IoU với gold chỉ còn khoảng 0.59.

## 5. Bổ sung sau khi chấm với gold và kiểm tra bằng model

- Kiểm tra riêng ba frame trước và sau mọi endpoint để tránh bbox bắt đầu sớm hoặc treo sau khi xe rời khung.
- Thêm keyframe ngay trước, trong và sau occlusion/crossing; kiểm tra frame giữa để phát hiện interpolation drift.
- Mọi bất đồng với model phải được xác minh bằng hình ảnh và rule; không dùng model như nhãn tham chiếu.
- Với vật thể tĩnh bị model nhận nhầm, kiểm tra ngữ cảnh để phân biệt xe thật trong cảnh với hình xe trên quảng cáo hoặc vật thể có hình dáng tương tự.
