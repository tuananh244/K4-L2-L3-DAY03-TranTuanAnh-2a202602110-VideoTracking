# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Trần Tuấn Anh`
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

Bổ sung của nhóm (nếu có): `không`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | 25 frame đủ ngắn để tin rằng đó vẫn là cùng một xe đang di chuyển liên tục qua vùng bị che, tránh tạo ID mới do nhiễu che khuất tạm thời. |
| Xe bị che lâu hơn ngưỡng trên | Nếu xe xuất hiện lại ở vị trí/hướng phù hợp với chuyển động (vị trí,kích thước bbox) ngoại suy từ trước lúc bị che thì vẫn giữ ID cũ, nhưng ghi lại thành "ca mơ hồ" ở mục 4;nếu không rõ ràng (vị trí lệch nhiều, kích thước đổi đột ngột) thì tạo track mới | Che khuất càng lâu, rủi ro nhầm hai xe khác nhau càng cao, nên cần thêm bằng chứng về vị trí/tốc độ trước khi giữ ID thay vì áp dụng máy móc một con số ngưỡng. |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Không đảm bảo đó là xe vừa mới xuất hiện trên camera. |
| Hai xe cắt nhau / chồng lên nhau | Giữ ID theo hướng và vận tốc di chuyển của mỗi xe trước thời điểm giao nhau (ngoại suy vị trí dự kiến), không gán ID theo bbox nào gần nhất tại đúng frame giao nhau | Tại điểm giao nhau, hai bbox dễ chồng lấn/rất gần nhau nên gán theo khoảng cách gần nhất tại đúng frame đó rất dễ hoán đổi ID giữa hai xe. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `cạnh ngắn nhất của bbox lớn hơn hoặc bằng 15px và có thể phân biệt rõ là xe bốn bánh (không phải bóng/vệt sáng)` |
| Xe đang đỗ, không di chuyển | Vẫn vẽ bbox ở mỗi frame như bình thường, toạ độ gần như không đổi qua các frame liên tiếp |
| Keyframe đặt dày ở đâu | Dày nhất ở các đoạn có che khuất, giao cắt giữa hai xe, hoặc xe đổi tốc độ/hướng đột ngột; với clip_01 gán gần như mỗi frame (trung bình ~78–79 frame/track trên tổng 190 frame của clip, vì có 628 dòng / 8 track) do xe di chuyển liên tục và nhiều đoạn giao cắt |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_02 / frame 1–4 / ID 2`
- Tình huống: xe rất nhỏ ở góc trái khung hình, bbox thu nhỏ dần (rộng `~19px` ở frame 1 xuống còn `~4px` ở frame 3) rồi biến mất hoàn toàn từ frame 4 trở đi.
- Quyết định: kết thúc track tại frame 4, không giữ ID chờ xe xuất hiện lại.
- Lý do: Xe đã rời khỏi khung hình chứ không phải bị che khốt

### Ca 2
- Clip/frame/ID: `clip_02 / frame 1-60 / ID 3`
- Tình huống: toạ độ bbox giữ nguyên y hệt qua nhiều frame liên tiếp.
- Quyết định: giữ nguyên ID, "đóng băng" bbox tại vị trí cuối cùng quan sát được.
- Lý do: xe được cho là đang đỗ nên toạ độ không đổi là hợp lý.

### Ca 3
- Clip / frame / ID: `clip_01/frame 65/ID 7`
- Tình huống: bắt đầu xuất hiện trên ảnh nhưng bị xe che đi bởi `ID 4`.
- Quyết định: Gán bbox từ frame đầu tiên có thể xác định đó là xe bốn bánh, giữ ID 7 trong đoạn bị che và ghi chú đoạn này là ca bị che khuất; từ frame 93, khi xe nhìn rõ hơn, tiếp tục chỉnh bbox theo phần nhìn thấy.
- Lý do: Ở frame 65, xe đã đủ dấu hiệu để xác định nhưng phần lớn bị che bởi ID 4. Không tạo ID mới chỉ vì xe bị che một phần.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Khi xe bị che, phải kiểm tra frame ngay trước và ngay sau đoạn che khuất. Chỉ giữ ID cũ khi vị trí, kích thước và hướng di chuyển phù hợp; nếu không đủ bằng chứng thì tạo track mới và ghi lại ca mơ hồ.
- Khi xe không còn nhìn thấy hoặc chỉ còn là bóng/vệt sáng, phải kết thúc track; không kéo bbox theo suy đoán. Trước khi khóa nhãn, kiểm tra riêng các track có ghost, phân mảnh hoặc đổi ID.
