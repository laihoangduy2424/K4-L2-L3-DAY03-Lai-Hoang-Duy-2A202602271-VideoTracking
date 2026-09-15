# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: **Lại Hoàng Duy — 2A202602271**  
Ngày: **15/09/2026**

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT, Rectangle Track, export MOT 1.1 |
| Thời gian gán `clip_02` (warm-up) | 14:14 15/09/2026 |
| Thời gian gán `clip_01` | 15:02 15/09/2026 |
| Số track đã vẽ trong `clip_01` | 8 track, 557 bbox |
| Số keyframe trung bình mỗi track | Chưa được lưu trong MOT export/notebook |

Ba tình huống khó nhất là giữ identity qua đoạn che khuất/cắt nhau quanh frame 87, xác định đúng frame bắt đầu của ID 6 quanh frame 103–105, và ngăn bbox bị trôi do nội suy quanh frame 109–110. Tôi giữ ID khi quỹ đạo còn liên tục, rà endpoint ở các frame liền kề, và thêm keyframe tại đoạn hình học thay đổi nhanh.

## 2. Tự kiểm và kiểm chéo

- Lượt 1 — identity/timeline: rà đoạn quanh frame 87; nhãn tay giữ được identity trong khi ReID treatment tách một xe thành ID 17 và 18.
- Lượt 2 — entry/exit: phát hiện ID 6 có bbox sớm ở frame 103–105 và cần chỉnh endpoint.
- Lượt 3 — geometry/interpolation: phát hiện bbox trôi quanh frame 109–110; IoU tại frame 110 khoảng 0.59.

## 3. Pre-gold lock và chấm sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 | b1699842a60923899d18915744ce44815859d233af8263906753027b92db7e2b |
| Thời điểm khóa | 16:35 15/09/2026 |
| Số row / frame / track trước khi mở reference | 557/190/8 |

| Phiên bản | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | — | — | — | — | — | — | — | — | — | — |
| Bản cuối trong notebook | 0.826 | 0.815 | 0.838 | 0.871 | 0.975 | 0.951 | 0.859 | 6 | 22 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **ĐẠT**.

| Loại lỗi | Frame | ID | Hướng xử lý |
| --- | --- | --- | --- |
| Bbox bắt đầu sớm | 103–105 | 6 | Rà lại endpoint và bỏ ba bbox trước khi xe xuất hiện rõ |
| Bbox trôi do nội suy | 110 | 6 | Thêm keyframe quanh đoạn này và ôm sát phần xe nhìn thấy |

## 4. Kết quả model: ByteTrack control vs ReID treatment

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13 |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml` / `configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | 0.25 / 0.70 / 960 / `[2, 5, 7]` |
| device | CUDA device `0` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.826 | 0.815 | 0.838 | 0.871 | 0.975 | 0.951 | 0.859 | 6 | 22 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.782 | 0.729 | 0.840 | 0.881 | 0.907 | 0.802 | 0.871 | 95 | 14 | 1 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của tôi là 0.951, thấp hơn IDF1 0.975. Cả hai đều cao và evaluator ghi nhận 0 ID switch, nên lỗi còn lại chủ yếu thuộc coverage và geometry: 6 FP, 22 FN và một bbox có IoU thấp. Khi MOTA cao nhưng IDF1 thấp, mô hình có thể phát hiện đúng phần lớn bbox nhưng duy trì identity kém; MOTA gộp FP, FN và IDSW theo số detection của gold nên vài lần đổi ID có ảnh hưởng tương đối nhỏ, còn IDF1 đo trực tiếp tính nhất quán của identity trên toàn chuỗi.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW?**

ReID treatment đạt IDF1 0.900 và AssA 0.820, cao hơn ByteTrack lần lượt 0.025 và 0.044. Cả hai đều có 2 IDSW, nhưng vị trí lỗi khác nhau: ByteTrack đổi ID ở frame 59 (gold ID 4) và 94 (gold ID 5); ReID đổi ID ở frame 87 (gold ID 5) và 113 (gold ID 6). Điều này cho thấy treatment duy trì association tốt hơn trên tổng thể, dù chưa loại bỏ được các lần đứt identity. Đây là so sánh hai hệ thống tracker giữ cùng detector input; chênh lệch không cô lập riêng tác động nhân quả của ReID vì ByteTrack và BoT-SORT còn khác cách triển khai association.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

DetA tăng từ 0.649 lên 0.711. FN giảm mạnh từ 54 xuống 26, trong khi FP tăng nhẹ từ 88 lên 91; treatment giữ được nhiều detection thật hơn nhưng cũng duy trì thêm một số detection thừa. AssA vẫn thấp hơn chất lượng nhãn tay và cả hai model vẫn có 2 IDSW, nên lỗi còn lại gồm cả detector (FP/FN, bbox thừa và lệch) lẫn association (tách track/đổi ID).

**4. Một chỗ bạn đúng và ReID sai:**

Quanh frame 87, xe tương ứng ID 5 trong nhãn/gold vẫn là cùng một xe. ReID treatment đổi từ ID 17 sang ID 18, tạo một ID switch và tách track, trong khi nhãn tay giữ identity liên tục; đối chiếu với gold xác nhận quyết định của tôi.

**Một chỗ ReID làm bạn xem lại annotation:**

Frame 109–110 cho thấy bbox giữa nhãn và ReID lệch nhau; evaluator so với gold xác nhận bbox nhãn tại frame 110 chỉ đạt IoU khoảng 0.59. Đây là tín hiệu cần xem lại nội suy và thêm keyframe, nhưng quyết định sửa dựa trên hình ảnh cùng gold/evaluator, không dựa riêng vào model.

**5. Nếu phải gán thêm 10 clip nữa**

Tôi sẽ bổ sung quy tắc rà ba frame trước/sau mọi endpoint, đặt keyframe dày quanh occlusion/crossing/đổi hướng, và bắt buộc kiểm tra frame giữa mỗi cặp keyframe dài để phát hiện bbox trôi. Quy trình sẽ có ba lượt QC tách biệt cho identity, endpoint và geometry; mọi bất đồng với model được ghi bằng frame–ID–rule rồi xác minh bằng hình ảnh.

cấu hình                       HOTA   DetA   AssA   IDF1    FP    FN  IDSW
ReID appearance=0.7           0.763  0.711  0.820  0.900    91    26     2
ReID appearance=0.8           0.763  0.711  0.820  0.900    91    26     2
ReID appearance=0.9           0.763  0.710  0.820  0.899    91    27     2

## 6. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [x] `reports/REPORT.md`

