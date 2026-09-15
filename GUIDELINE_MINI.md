# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Nguyễn Đôn Quốc Tuấn — 2A202602127`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm: Xe đang đỗ (taxi đứng yên, xe tải dừng) **vẫn phải gán và track** trong suốt thời gian nó trong khung hình. Đây là điểm tôi bỏ sót ban đầu — model phát hiện ra.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (= 2 giây @ 12.5 fps) | Dưới 2 giây, chuyển động Kalman vẫn dự đoán tốt vị trí, không cần track mới |
| Xe bị che lâu hơn ngưỡng trên | mở **track mới** | Sau 2 giây, không thể xác định chắc chắn đây là cùng xe |
| Xe rời khung hình rồi quay lại | **track mới** — bấm `outside` khi xe rời khung | Không thể xác định chắc chắn xe quay lại là cùng xe (đặc biệt khi nhiều xe giống nhau) |
| Hai xe cắt nhau / chồng lên nhau | **không đổi ID** — quan sát hướng di chuyển và kích thước để quyết định | Theo dõi quỹ đạo từ trước khi cắt nhau để gán đúng ID |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** — không vẽ phần bị che |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng: chiều rộng hoặc chiều cao bbox ≥ 20px |
| Xe đang đỗ, không di chuyển | **vẫn track** — đặt keyframe ở frame đầu và frame cuối, thêm 1–2 keyframe giữa để bbox ổn định |
| Keyframe đặt dày ở đâu | Khi xe rẽ, phanh, bị che, hoặc cắt nhau — cứ 5–8 frame thêm một keyframe trong những đoạn này. Đoạn xe đi thẳng đều có thể thưa hơn (15–20 frame/keyframe). Bắt buộc thêm keyframe nếu IoU giữa frame hiện tại và keyframe gần nhất < 0.70 |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1 — Occlusion dài: có giữ ID không?
- Clip / frame / ID: `clip_01 / frame 85–95 / track ID 5`
- Tình huống: Xe ID 5 bị xe ID 4 che hoàn toàn trong ~10 frame (frame 85–95). Xe di chuyển theo hướng từ trái sang phải, tốc độ trung bình.
- Quyết định: **Giữ nguyên ID 5** — 10 frame < 25 frame (ngưỡng 2 giây).
- Lý do: Hướng di chuyển nhất quán, kích thước bbox tương tự trước và sau khi che. Có thể track liên tục bằng CVAT keyframe trước và sau occlusion.

### Ca 2 — Hai xe cắt nhau: ID nào đi theo đường nào?
- Clip / frame / ID: `clip_01 / frame 106–113 / track ID 6 và track ID 7`
- Tình huống: Xe ID 6 (từ phải sang trái) và xe ID 7 (từ trái sang phải) di chuyển ngược chiều và overlap bbox trong khoảng frame 108–111. Khi chồng nhau, khó phân biệt xe nào là xe nào chỉ dựa vào bbox.
- Quyết định: **Không đổi ID** — dùng quỹ đạo trước khi overlap để dự đoán. Xe ID 6 tiếp tục sang trái, xe ID 7 tiếp tục sang phải.
- Lý do: Hướng di chuyển trước overlap (frame 100–107) rõ ràng. Hai xe có kích thước khác nhau (ID 6 lớn hơn). Quyết định giữ nguyên ID dựa trên motion vector.

### Ca 3 — Xe đỗ im: có cần track không?
- Clip / frame / ID: `clip_01 / frame 16–116 / xe taxi đỗ ở vỉa hè`
- Tình huống: Một xe taxi đứng im ở vỉa hè từ frame 16 đến frame 116 (khoảng 100 frame). Ban đầu tôi bỏ qua vì tập trung vào xe đang di chuyển.
- Quyết định: **Phải gán** — gold và model đều track xe này. Đây là xe bốn bánh trong khung hình.
- Lý do: Theo luật lab "xe đang đỗ vẫn là vehicle và vẫn cần track suốt thời gian nó trong khung". Sau khi phát hiện FP của model trùng vị trí này, xem lại và gán thêm track.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Xe đứng im**: Bổ sung rõ ràng "xe đang đỗ phải track" vào mục 3. Ban đầu guideline không nhấn mạnh đủ, dẫn đến bỏ sót 1 xe taxi trong 100 frame.
- **Ngưỡng keyframe**: Thêm rule định lượng — "nếu IoU giữa frame hiện tại và keyframe gần nhất < 0.70 thì bắt buộc thêm keyframe". Trước đây chỉ nói "đặt keyframe dày khi xe đổi hướng" nhưng chưa có ngưỡng rõ ràng, dẫn đến 6 điểm bbox drift trong đánh giá.
- **Xe nhỏ ở rìa**: Sau khi đối chiếu với gold, thấy cần gán từ frame sớm hơn (khi bbox ≥ 15px thay vì 20px) vì gold bắt đầu gán sớm hơn tôi ở một số track.
