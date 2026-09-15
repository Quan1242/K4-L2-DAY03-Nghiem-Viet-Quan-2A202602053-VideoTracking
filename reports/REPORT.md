# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: Nghiem Viet Quan / 2A202602053
Ngày: 2026-09-15

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT local |
| Thời gian gán `clip_02` (warm-up) | 0 |
| Thời gian gán `clip_01` | 240 phút |
| Số track đã vẽ trong `clip_01` | 8 |
| Số keyframe trung bình mỗi track | 54.6 (437 keyframe thủ công / 8 track) |

Annotation dùng một label `vehicle`, Rectangle → Track, không thêm attribute. `clip_01`
có 190 frame, 617 bbox và 8 track.

Ba tình huống khó nhất khi gán clip này, và cách xử lý:

1. Xe bị che: giữ cùng track ID và chỉ vẽ phần xe nhìn thấy.
2. Xe ra/vào khung: dùng Outside khi xe rời cảnh; không kéo bbox vào vùng không quan sát được.
3. Nhiều xe chồng lấn: giữ bbox và ID riêng cho từng xe, cho phép bbox giao nhau.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua:

- Lượt 1: kiểm tra ID xuyên suốt clip, không phát hiện ID switch trong annotation.
- Lượt 2: kiểm tra frame đầu/cuối và Outside; cần xem lại cảnh báo track 2 ở frame 1–15 và 150–164.
- Lượt 3: kiểm tra midpoint và bbox ở các đoạn đổi hướng/che khuất; cần hoàn tất bằng mắt.

Kiểm chéo với: CHƯA GHI. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: CHƯA GHI. Số lỗi bạn ấy tìm được trong bản của bạn: CHƯA GHI.

Ca hai người quyết khác nhau và luật còn thiếu trong `GUIDELINE_MINI.md`:

CHƯA GHI SAU PEER REVIEW.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `2e916ce071f85934dc5338a38c4047e8250da9d042cc45c3f788dd28176a4933` |
| Thời điểm khóa | 2026-09-15, đã chạy `tools/lock_pre_gold.py` |
| Số row / frame / track trước khi mở reference | 617 / 190 / 8 |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | CHƯA CÓ | CHƯA CÓ | CHƯA CÓ | CHƯA CÓ | CHƯA CÓ | CHƯA CÓ | CHƯA CÓ | CHƯA CÓ | CHƯA CÓ | CHƯA CÓ |
| Bản pre-gold / nhãn bạn vs gold | 0.7852 | 0.7610 | 0.8113 | 0.8719 | 0.9378 | 0.8709 | 0.8587 | 59 | 15 | 0 |
| Sau rework | CHƯA CÓ | CHƯA CÓ | CHƯA CÓ | CHƯA CÓ | CHƯA CÓ | CHƯA CÓ | CHƯA CÓ | CHƯA CÓ | CHƯA CÓ | CHƯA CÓ |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**.

Các diagnostics chính của pre-gold: không có missed track hoặc fragmented GT track; có
ghost/biên track ở ID 4, 5, 6, 8 và một số bbox lỏng, nổi bật ở ID 5 quanh frame 80–108.
Chưa có bản export sau rework nên chưa thể báo chênh lệch trước/sau.

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | ---: | ---: | --- |
| Bbox IoU thấp | 80–108 | 5 | Chưa rework |
| Ghost trước/sau thời gian tham chiếu | 51–53, 149–151 | 4 | Chưa rework |
| Ghost trước thời gian tham chiếu | 75–79 | 5, 6 | Chưa rework |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13 |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml`, `botsort-reid.yaml` |
| conf / IoU / imgsz / classes | 0.25 / 0.70 / 960 / 2, 5, 7 |
| device | `0` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bạn vs gold | 0.7852 | 0.7610 | 0.8113 | 0.8719 | 0.9378 | 0.8709 | 0.8587 | 59 | 15 | 0 |
| ByteTrack control vs gold | 0.7085 | 0.6487 | 0.7761 | 0.8463 | 0.8746 | 0.7487 | 0.8226 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.7635 | 0.7110 | 0.8204 | 0.8721 | 0.9001 | 0.7923 | 0.8595 | 91 | 26 | 2 |
| ReID vs bạn | 0.7351 | 0.6773 | 0.8005 | 0.8738 | 0.8813 | 0.7634 | 0.8577 | 82 | 61 | 3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA và IDF1**

Với nhãn tay, MOTA là 0.8709 và IDF1 là 0.9378; cả hai đều cao. Điều này cho thấy
annotation vừa bao phủ tốt vừa giữ identity ổn định. MOTA tập trung vào lỗi phát hiện,
false positive và false negative; ID switch chỉ là một thành phần nên MOTA có thể vẫn cao
khi identity không ổn định. Trong dữ liệu này annotation có IDSW = 0.

**2. ByteTrack và BoT-SORT + ReID**

ReID tốt hơn ByteTrack trên gold: HOTA 0.7635 so với 0.7085, AssA 0.8204 so với
0.7761, IDF1 0.9001 so với 0.8746 và MOTA 0.7923 so với 0.7487. ReID giảm FN từ 54
xuống 26, nhưng FP tăng từ 88 lên 91; cả hai có IDSW = 2. Ví dụ, ReID vẫn có lỗi
fragmentation quanh track tham chiếu 5 ở frame 87 và track 6 ở frame 113. Đây là so sánh
hai system khác implementation, không cô lập causal effect của ReID riêng lẻ.

**3. DetA, FP và FN**

DetA tăng từ 0.6487 lên 0.7110 khi dùng ReID; FN giảm mạnh 54 xuống 26 nhưng FP tăng
88 lên 91. Vì IDSW và fragmentation vẫn còn, lỗi còn lại là cả detection lẫn association;
association nổi bật ở các đoạn xe bị che hoặc xuất hiện lại.

**4. Một chỗ annotation đúng và ReID sai**

Ở các đoạn có identity ổn định, annotation giữ đúng track liên tục với IDSW = 0. ReID
vẫn có fragmentation tại frame 87 của track 5 và frame 113 của track 6, nên model đã
đổi/đứt identity trong khi annotation giữ identity tham chiếu.

**5. Một chỗ ReID làm cần xem lại annotation**

ReID-vs-bạn cho thấy lỗi quanh frame 107–110 của track 6 và frame 87 của track 5. Đây
là các frame cần mở lại trong CVAT để kiểm tra bbox/Outside/Occluded; chưa đủ bằng chứng
để kết luận annotation sai chỉ từ metric model.

## 6. Nếu phải gán thêm 10 clip nữa

Tôi sẽ bổ sung quy tắc ghi rõ frame CVAT và frame MOT khi báo lỗi, quy tắc phân biệt
Occluded với Outside, và quy tắc xử lý xe quay lại sau khi đã rời khung. Quy trình sẽ là:
gán từng xe hoàn chỉnh, kiểm midpoint sau mỗi đoạn keyframe dài, kiểm entry/exit, Save và
reload, export MOT, validator, ba lượt QC, peer review, rồi mới khóa pre-gold.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [ ] `annotations/clip_02/gt.txt` — chưa kiểm tra/cập nhật
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [x] `reports/REPORT.md`