# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Nguyễn Đôn Quốc Tuấn — 2A202602127`
Ngày: `2026-09-15`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | ~30 phút |
| Thời gian gán `clip_01` | ~90 phút |
| Số track đã vẽ trong `clip_01` | 8 |
| Số keyframe trung bình mỗi track | ~6–8 keyframe/track |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Occlusion (xe bị che)**: Ở khoảng frame 87–94, xe track 5 bị xe khác che khuất. Tôi quyết định giữ nguyên ID vì thời gian che dưới 25 frame (< 2 giây). Vẽ keyframe ngay trước và ngay sau khoảng che để CVAT nội suy đúng vị trí.
2. **Hai xe cắt nhau (frame 106–113)**: Track 6 và track 7 di chuyển qua nhau. Tôi dùng IoU và hướng di chuyển để quyết định không đổi ID. Đặt keyframe dày trong khoảng xe chồng lên nhau để bbox chính xác hơn.
3. **Xe xuất hiện ở rìa khung**: Một số xe chỉ thấy một phần khi vừa xuất hiện (frame 16–20 với track 1). Tôi bắt đầu track từ frame đầu tiên nhận ra rõ ràng là xe bốn bánh, bbox chạm đúng rìa ảnh.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Phát hiện 1 trường hợp ID nhấp nháy ở frame 87–90 (track 5 bị tắt vì xe khuất). Đã sửa bằng cách dùng Merge để nối lại thành một track.
- Lượt 2: Kiểm tra frame đầu/cuối — tất cả 8 track đều có `outside` ở đúng frame xe rời khung. Không có bbox treo.
- Lượt 3: Kiểm tra giữa các keyframe cách nhau xa nhất (frame 50–75 của track 4). Phát hiện bbox trôi nhẹ ở frame 60, đã thêm keyframe bổ sung.

Kiểm chéo với: `làm cá nhân`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `N/A`. Số lỗi bạn ấy tìm được trong bản của bạn: `N/A`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`Làm cá nhân, không có kiểm chéo thực tế. Tuy nhiên sau khi đối chiếu với gold, phát hiện cần bổ sung luật về ngưỡng bbox drift tối đa giữa hai keyframe.`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | Xem `evidence/pre-gold/clip_01/manifest.json` |
| Thời điểm khóa | 2026-09-15, trước khi nhận gold |
| Số row / frame / track trước khi mở reference | 563 bbox · 8 track · 190 frame |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.756 | 0.739 | 0.775 | 0.823 | 0.963 | 0.927 | 0.797 | 16 | 26 | 0 |
| Sau rework | 0.756 | 0.739 | 0.775 | 0.823 | 0.963 | 0.927 | 0.797 | 16 | 26 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| BBOX TRÔI | 152 | 6 | Thêm keyframe ở frame 152, kéo bbox khít hơn theo phần nhìn thấy của xe |
| BBOX TRÔI | 95–96 | 5 | Thêm keyframe ở frame 95, điều chỉnh bbox theo hướng di chuyển xe |
| BBOX TRÔI | 106–107 | 6 | Thêm keyframe để tránh drift khi xe chuyển hướng |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13 |
| weights / hai tracker | yolo26n.pt / bytetrack.yaml & botsort-reid.yaml |
| conf / IoU / imgsz / classes | 0.25 / 0.70 / 960 / [2, 5, 7] (car, bus, truck) |
| device | 0 (GPU) |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.756 | 0.739 | 0.775 | 0.823 | 0.963 | 0.927 | 0.797 | 16 | 26 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.707 | 0.653 | 0.769 | 0.825 | 0.898 | 0.785 | 0.799 | 97 | 22 | 2 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA (0.927) cao hơn IDF1 (0.963) không nhiều, cả hai đều ở mức xuất sắc. Tuy nhiên, điều quan trọng cần hiểu: MOTA chỉ đếm mỗi ID switch **một lần** trong toàn bộ quãng đời track, trong khi IDF1 phạt theo tỷ lệ số frame mà track bị gán sai ID. Nếu một xe 100 frame bị đổi ID ở frame 50, MOTA chỉ trừ 1 điểm (một lần switch), nhưng IDF1 phạt tương đương 50 frame sai. Do đó, một bài có MOTA cao nhưng IDF1 thấp tức là có ít sự kiện switch nhưng mỗi switch kéo dài lâu — dấu hiệu rõ ràng của lỗi identity nghiêm trọng. Với nhãn của tôi: 0 IDSW nên MOTA rất cao, và IDF1 cũng rất cao (0.963), nhất quán với nhau.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

- **IDF1**: ByteTrack 0.875 vs ReID 0.900 — ReID tốt hơn +0.025.
- **AssA**: ByteTrack 0.776 vs ReID 0.820 — ReID tốt hơn đáng kể +0.044.
- **IDSW**: Cả hai đều có 2 ID switch.

Frame sequence đáng chú ý: Ở frame 87, ByteTrack để track gold 5 chuyển từ ID 23 → ID 32 (IDSW tại frame 94), trong khi ReID chuyển từ ID 17 → 18 (IDSW tại frame 87). Cả hai đều switch nhưng ReID phục hồi nhanh hơn nhờ appearance embedding giúp re-associate đúng sau khi xe thoát khỏi occlusion. Điều này thể hiện qua AssA cao hơn của ReID.

**Lưu ý quan trọng**: Đây là system comparison. ByteTrack và BoT-SORT là hai implementation khác nhau, có logic association và Kalman filter khác nhau. Không thể kết luận rằng ReID embedding đơn thuần là nguyên nhân dẫn đến sự khác biệt.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

- **DetA**: ByteTrack 0.649 vs ReID 0.711 — ReID cao hơn +0.062, nhưng cả hai thấp hơn nhiều so với nhãn tay (0.739). Detector bỏ sót nhiều xe bị che.
- **FP**: ByteTrack 88 vs ReID 91 — ReID có thêm bbox thừa. Cả hai cao hơn nhiều so với nhãn tay (16 FP).
- **FN**: ByteTrack 54 vs ReID 26 — ReID ít bỏ sót hơn đáng kể, gần bằng nhãn tay (26 FN).

Kết luận: Lỗi chính còn lại là **detector** — YOLO bỏ sót nhiều xe bị che (FN cao) và phát hiện nhầm vật thể tĩnh (FP cao, đặc biệt ID không khớp với gold). Association (IDSW=2) tốt hơn detector, nhưng detection miss làm tụt AssA khi xe biến khỏi detection rồi được re-associate thành track mới.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Frame 108–109: ReID model có 3 bbox thừa (chỉ model có, bạn không gán). Quan sát frame sequence cho thấy đây là các vật thể nhỏ đứng im hoặc di chuyển rất chậm ở góc trên phải của ảnh — có thể là xe máy hoặc vật thể tĩnh bị detector nhận nhầm là xe bốn bánh. Nhãn tay của tôi không gán vì nhìn rõ đây không phải xe bốn bánh (không đủ kích thước và hình dạng). ReID model lấy class [2,5,7] từ COCO nhưng detector YOLO vẫn có thể confident nhầm. Đây là điểm FP của model, không phải FN của nhãn tay.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Frame 91: ReID có 1 bbox, nhãn tay của tôi cũng có 1 bbox, nhưng disagreement. Khi xem lại frame này, model có 1 bbox thừa (ID 7, frame 16–116 kéo dài 43 frame). Điều này làm tôi xem lại và xác nhận đây là xe đứng im ở vỉa hè — có thể là taxi đang đỗ. Theo luật của lab, xe đang đỗ vẫn là `vehicle` và phải track. Tôi đã bỏ sót xe này. Đây là trường hợp model (dù nhầm ID tracking do xe không di chuyển) đã chỉ ra chỗ tôi thiếu annotation. Tuy nhiên, sau khi xem frame 16–116, tôi thấy đây thực sự là xe đứng im — model track liên tục 43 frame điều này hợp lý, còn tôi đã bỏ qua vì tập trung vào xe di động.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- Bổ sung luật rõ hơn về **xe đứng im**: phải track ngay cả khi xe không di chuyển, kể cả taxi đang đỗ. Hiện tại guideline chưa nhấn mạnh đủ.
- Thêm ngưỡng cụ thể cho **bbox drift**: nếu IoU giữa hai keyframe liên tiếp < 0.70 thì bắt buộc thêm keyframe trung gian.
- Đổi quy trình: **xem bằng mắt một lượt nhanh trước khi bắt đầu gán** để đếm tổng số xe xuất hiện trong clip, tránh bỏ sót xe đứng im hoặc xuất hiện muộn.
- Dùng `visualize_tracks.py` ngay sau khi gán xong mỗi track (thay vì đợi đến cuối) để phát hiện bbox trôi sớm hơn.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
