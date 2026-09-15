# Báo cáo Ngày 3 — Tracking Annotation0.711

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: Trần Tuấn Anh
Ngày: 15/09/2026

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `không` |
| Thời gian gán `clip_02` (warm-up) | 30 phút |
| Thời gian gán `clip_01` | 60 phút |
| Số track đã vẽ trong `clip_01` | 8 |
| Số keyframe trung bình mỗi track | 78 |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Đối tượng bị che khuất: xem thêm các frame trước và sau rồi nối cùng một ID.
2. Các đối tượng đứng gần nhau: kiểm tra vị trí và hướng di chuyển để tránh đổi ID.
3. Đối tượng ra vào khung hình: đặt frame bắt đầu/kết thúc rõ ràng và không kéo track ngoài vùng quan sát.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: kiểm tra ID có bị đổi hoặc trùng không.
- Lượt 2: kiểm tra frame đầu và cuối của từng track.
- Lượt 3: kiểm tra các frame giữa, nhất là đoạn có che khuất hoặc giao nhau.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `b242e42875685dcb8dd038bac670909e84af5e71f8f305fb8ae67eb566596172` |
| Thời điểm khóa | `2026-09-15T04:23:54.119815+00:00` |
| Số row / frame / track trước khi mở reference | `628/190/8` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.814 | 0.789 | 0.843 | 0.890 | 0.949 | 0.893 | 0.879 | 58 | 3 | 0 |
| Sau rework | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.74 | 0.823 | 88 | 54 | 2 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **chưa**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID: 

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
ID Switch (Đứt gãy / Nhảy ID) | 87 | GT 5 (Pred 17 & 18) | Gộp (merge) Pred track 17 và Pred track 18 lại thành 1 ID duy nhất để duy trì toàn vẹn cho đối tượng số 5. |
Ghost track (Nhận diện sai/Rác) | 16-116 | Pred 7 | Xóa bỏ hoàn toàn track 7 vì đây là track "bóng ma", không khớp với bất kỳ đối tượng thật nào trong nhãn chuẩn. |
Ghost track (Nhận diện sai/Rác) | 106-121 | 158-178 | Pred 27, Pred 38 | Xóa các track rác 27 và 38 (có thể xóa luôn các track rác ngắn 1 frame: 24, 26, 28 ở frame 104-107). |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / bytetrack.yaml và /content/Day3-Lab/configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25 / 0.7 / 960 / [2, 5, 7]` |
| device | `0` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
|---|---|---|---|---|---|---|---|---|---|---|
| **ban_vs_gold** | 0.814 | 0.789 | 0.843 | 0.890 | 0.949 | 0.893 | 0.879 | 58 | 3 | 0 |
| **bytetrack_vs_gold** | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| **reid_vs_gold** | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| **reid_vs_ban** | 0.766 | 0.705 | 0.833 | 0.911 | 0.871 | 0.740 | 0.904 | 86 | 76 | 1 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

Kết quả của em thì MOTA bé hơn IDF1 (0.893 < 0.949>)

Nếu MOTA cao nhưng IDF1 thấp, nghĩa là mô hình phát hiện đúng vị trí đối tượng và ít bỏ sót, nhưng thường xuyên đổi hoặc gán nhầm ID. MOTA không phạt nặng lỗi này vì công thức chủ yếu đếm tổng FP + FN + IDSW; mỗi lần đổi ID chỉ được tính như một lỗi IDSW, không tính toàn bộ các frame tiếp theo bị gán sai danh tính. Ngược lại, IDF1 đánh giá trực tiếp độ nhất quán của ID qua các frame nên nhạy hơn với lỗi identity switch và track fragmentation.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

So với ByteTrack, BoT-SORT + ReID có **IDF1 tăng từ 0.875 lên 0.900** và **AssA tăng từ 0.776 lên 0.820**, cho thấy treatment giữ liên kết danh tính và association tốt hơn ở mức tổng thể. Tuy nhiên **IDSW không đổi: cả hai đều có 2 lần**, nên ReID không loại bỏ hoàn toàn lỗi đổi ID.

Ví dụ ở chuỗi **frame 87-95**: với một đối tượng chính, ByteTrack duy trì track 15 còn ReID duy trì track 9. Tuy nhiên, ở đối tượng GT track 5, ByteTrack bị phân mảnh và đổi từ pred 23 sang 32 tại frame 94; ReID cũng có một track phụ 18 xuất hiện từ frame 87, và diagnostics ghi nhận ID switch tại frame 87 (17 -> 18). Vì vậy, ở chuỗi này ReID không tốt hơn rõ rệt về IDSW; lợi ích của nó thể hiện ở điểm số association tổng thể cao hơn, không phải ở việc mọi đoạn đều giữ một ID duy nhất.

Kết luận này **không cô lập causal effect của ReID**: hai thí nghiệm dùng hai tracker implementation khác nhau (ByteTrack và BoT-SORT), nên chênh lệch có thể đến từ cả cơ chế tracker, tham số và ReID, chứ không thể quy toàn bộ cho ReID.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

So với ByteTrack, BoT-SORT + ReID làm `DetA` tăng từ **0.649 lên 0.711**. `FN` giảm mạnh từ **54 xuống 26**, cho thấy treatment bắt được nhiều bounding box của đối tượng thật hơn và cải thiện độ bao phủ phát hiện. Ngược lại, `FP` tăng nhẹ từ **88 lên 91**, nghĩa là vẫn còn một số box/ghost track không khớp với ground truth; ReID không tự sửa được lỗi detector và còn có thể giữ lại một số track sai.

Vì vậy, phần cải thiện chính là ở detection/coverage, nhưng lỗi còn lại không chỉ là detector. `AssA` vẫn chỉ đạt **0.820**, `IDF1` là **0.900** và vẫn có **2 IDSW** (ở frame 87 và 113), nên association/duy trì ID vẫn là nút thắt chính. Kết luận thận trọng là ReID giúp cả DetA và association tốt hơn ở mức tổng thể, nhưng còn một ít lỗi false positive của detector và lỗi phân mảnh/đổi ID của association.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Ở **frame 87, GT ID 5**, nhãn của em vẫn giữ đối tượng là một track duy nhất. ReID lại ghi nhận một lần đổi ID từ **Pred 17 sang Pred 18** tại frame này; track 18 chỉ xuất hiện thêm một frame trong đoạn bị phân mảnh. Vì vậy đây là lỗi association/fragmentation của ReID, không phải lỗi annotation: bbox của ReID vẫn nằm gần đối tượng, nhưng danh tính bị đứt gãy.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Ở **frames 16-116, Pred ID 7**, ReID duy trì một track dài 43 frame nhưng diagnostics `reid_vs_me` đánh dấu đây là ghost track, không khớp với track nào trong nhãn của em. Ví dụ tại frame 87, Pred 7 nằm ở vùng khoảng `x=491, y=214`, trong khi các đối tượng được gán nhãn tại frame này nằm ở các vùng khác. Vì track này không có đối tượng tương ứng trong annotation và kéo dài nhiều frame, evidence nghiêng về việc **model sai (false positive/ghost track)** chứ chưa đủ cơ sở để sửa nhãn.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

Em sẽ bổ sung quy tắc: không đổi ID khi đối tượng bị che khuất ngắn; kiểm tra frame trước và sau khi nối track; kết thúc track ngay khi đối tượng ra khỏi khung hình; và đánh dấu các track nghi là ghost để kiểm tra lại.

Trong quy trình, em sẽ xem toàn bộ clip một lượt trước khi gán, sau đó kiểm tra riêng frame đầu/cuối, các đoạn che khuất và các đoạn đối tượng đi gần nhau. Cuối cùng em sẽ chạy kiểm tra ID/ghost track trước khi khóa bản nhãn.

## 7. Tệp đã nộp

- [X] `annotations/clip_01/gt.txt`
- [X] `annotations/clip_02/gt.txt`
- [X] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [X] `GUIDELINE_MINI.md` đã điền
- [X] `outputs/eval_vs_gold.json`
- [X] `outputs/model_reid_clip_01.txt`
- [X] `outputs/model_run_config.json`
- [X] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [X] `reports/review_partner.md`
- [X] `reports/REPORT.md` (file này)
