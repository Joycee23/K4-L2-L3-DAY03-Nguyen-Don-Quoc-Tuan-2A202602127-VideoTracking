# Peer review — Day 3

Chép file này thành `reports/review_partner.md`. Reviewer chỉ ghi finding; tác giả
tự sửa bài của mình và điền closure.

| Trường | Giá trị |
| --- | --- |
| Author | Nguyễn Đôn Quốc Tuấn — 2A202602127 |
| Reviewer | Nguyễn Đôn Quốc Tuấn (tự review) |
| Pair ID | solo-2A202602127 |
| CVAT version | app.cvat.ai (cloud) |
| Thời điểm review | 2026-09-15, sau khi xuất MOT 1.1 |

## Danh sách finding

Mỗi finding bắt buộc có `frame + ID + lỗi + sửa thế nào`. Nếu chưa thống nhất,
dùng `needs-review`; không ép tác giả sửa theo cảm tính.

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure: fixed / not-a-defect / needs-review |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | 151 | 152 | 6 | BBOX TRÔI | IoU chỉ còn 0.51 so với gold. Xe đang chuyển hướng nhẹ giữa hai keyframe xa nhau. Rule: thêm keyframe khi IoU < 0.60 | Thêm keyframe tại frame 152, kéo bbox khít theo phần nhìn thấy | fixed |
| 2 | 94 | 95 | 5 | BBOX TRÔI | IoU 0.51, xe đang tăng tốc. Interpolation bị drift | Thêm keyframe tại frame 95, điều chỉnh theo vị trí thực tế của xe | fixed |
| 3 | 105 | 106 | 6 | BBOX TRÔI | IoU 0.51, xe cắt nhau với track 7. Interpolation không chính xác | Thêm keyframe để xử lý đoạn xe cắt nhau (frame 104–113) | fixed |
| 4 | 106 | 107 | 6 | BBOX TRÔI | IoU 0.52, tiếp theo finding #3 | Đã xử lý cùng với finding #3 | fixed |
| 5 | 133 | 134 | 5 | BBOX TRÔI | IoU 0.55, xe đang phanh nhẹ | Thêm keyframe tại frame 134 | fixed |
| 6 | 146 | 147 | 4 | BBOX TRÔI | IoU 0.55, xe đang rẽ | Thêm keyframe tại frame 147 | fixed |

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | 8 track, tất cả là xe bốn bánh, không có xe máy hay người |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | Kiểm tra toàn bộ 190 frame, 0 ID switch |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS | Track 5 bị che ~10 frame (frame 85–95), giữ nguyên ID |
| Entry/exit đúng; không box treo sau khi xe rời khung | PASS | Tất cả 8 track có `outside` đúng frame cuối |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | FINDING | frame 95–96 track 5: bbox hơi rộng hơn phần thực sự nhìn thấy → đã sửa |
| Frame giữa hai keyframe không bị interpolation drift | FINDING | Xem findings 1–6 ở trên, đã thêm keyframe bổ sung |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | Đã kiểm: `cut -d, -f2 annotations/clip_01/gt.txt | sort -un | wc -l` trả về 8 |
| Mọi finding có cách sửa và closure do tác giả điền | PASS | Xem bảng findings ở trên |

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | ĐÃ SỬA | Lượt 1: phát hiện bbox drift ở 6 frame, đã sửa bằng cách thêm keyframe |
| 2 — endpoint/scope | PASS | Lượt 2: tất cả track có frame đầu đúng lúc xe rõ ràng, `outside` ở frame xe rời khung |
| 3 — geometry/interpolation | ĐÃ SỬA | Lượt 3: kiểm tra giữa keyframe dài nhất, phát hiện và sửa 6 điểm drift |

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: `Bbox drift ở frame 152 track 6 (IoU 0.51). Rule: khi xe thay đổi hướng hoặc tốc độ, khoảng cách giữa hai keyframe không được để IoU < 0.60 — phải thêm keyframe trung gian.`
2. Một finding tác giả đóng là `not-a-defect`, kèm lý do (nếu có): `Không có finding nào đóng là not-a-defect — tất cả 6 finding đều là bbox drift thực sự cần sửa.`
3. Một rule cần Lab Coach làm rõ (nếu có): `Ngưỡng IoU tối thiểu giữa các frame liên tiếp (không phải chỉ hai keyframe) là bao nhiêu thì được coi là "bbox ổn định"? Guideline hiện tại chỉ nói "thêm keyframe khi xe đổi hướng" nhưng chưa định lượng ngưỡng IoU cụ thể.`
