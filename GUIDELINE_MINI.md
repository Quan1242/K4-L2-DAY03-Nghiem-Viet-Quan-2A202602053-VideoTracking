# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Nghiem Viet Quan / 2A202602053`
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

Bổ sung của nhóm (nếu có): `...`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | identity vẫn xác định được qua chuyển động, vị trí và hình dáng |
| Xe bị che lâu hơn ngưỡng trên | giữ ID nếu vẫn xác định chắc là cùng xe; nếu không còn đủ evidence thì kết thúc track và tạo track mới khi xe xuất hiện lại | không đoán identity khi xe vắng quá lâu |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | `...` |
| Hai xe cắt nhau / chồng lên nhau | giữ hai ID riêng; cho phép bbox giao nhau nếu mỗi bbox vẫn ôm đúng một xe | overlap không có nghĩa là hai xe là một track |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; không đoán box trước đó |
| Xe đang đỗ, không di chuyển | vẫn gán track nếu còn nhìn thấy; giữ bbox ổn định và chỉ dùng Outside khi xe rời cảnh |
| Keyframe đặt dày ở đâu | đặt dày ở chỗ đổi hướng, đổi kích thước, che khuất, crossing và entry/exit; kiểm midpoint giữa các keyframe xa |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / 1–15 / ID 2`
- Tình huống: xe gần như đứng yên trong nhiều frame.
- Quyết định: vẫn giữ track và bbox, không đặt Outside khi xe còn nhìn thấy.
- Lý do: đứng yên không có nghĩa là object đã rời cảnh.

### Ca 2
- Clip / frame / ID: `clip_01 / 80–108 / ID 5`
- Tình huống: xe đổi vị trí/kích thước, interpolation có nguy cơ làm bbox trôi ở midpoint.
- Quyết định: kiểm midpoint và đặt thêm keyframe quanh đoạn đổi chuyển động.
- Lý do: hai đầu keyframe đúng chưa đủ chứng minh các frame giữa đúng.

### Ca 3
- Clip / frame / ID: `clip_01 / 149–151 / ID 4`
- Tình huống: xe ở ranh giới kết thúc track và có nguy cơ còn bbox sau khi rời cảnh.
- Quyết định: đặt Outside tại frame đầu tiên xe không còn xuất hiện; không kéo bbox vào vùng trống.
- Lý do: boundary chính xác tránh ghost track và giữ đúng thời gian tồn tại của object.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Khi báo lỗi phải ghi cả frame CVAT (0-based) và frame MOT (1-based) nếu có thể nhầm.
- Phân biệt Occluded (xe còn trong cảnh nhưng bị che) với Outside (xe không còn bbox trong cảnh).
