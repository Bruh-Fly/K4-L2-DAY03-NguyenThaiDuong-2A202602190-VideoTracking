# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Nguyễn Thái Dương`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục                               | Giá trị    |
| --------------------------------- | ---------- |
| Công cụ                           | `CVAT`     |
| Thời gian gán `clip_02` (warm-up) | `10 phút` |
| Thời gian gán `clip_01`           | `30 phút` |
| Số track đã vẽ trong `clip_01`    | `8`        |
| Số keyframe trung bình mỗi track  | `3`      |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `Xe bị che hoặc đi gần xe khác: xem thêm các frame trước và sau để xác định đúng xe và giữ nguyên ID.`
2. `Xe ở gần mép ảnh: bbox chỉ ôm phần xe nhìn thấy và có thể chạm mép ảnh.`
3. `Bbox bị lệch khi xe di chuyển: thêm keyframe khi thấy bbox bắt đầu trôi.`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

* `Lượt 1: kiểm tra ID có bị đổi hoặc bị tách thành nhiều track không.`
* `Lượt 2: kiểm tra frame đầu và cuối của từng track, tránh bbox bị treo.`
* `Lượt 3: kiểm tra các frame giữa để phát hiện bbox bị trôi.`

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.

Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`Điểm dễ khác nhau nhất là cách đặt bbox và thời điểm bắt đầu/kết thúc track. Cần quy định rõ hơn khi nào giữ ID và khi nào kết thúc track.`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence                                             | Giá trị                          |
| ---------------------------------------------------- | -------------------------------- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `...`                            |
| Thời điểm khóa                                       | `...`                            |
| Số row / frame / track trước khi mở reference        | `593 bbox / 190 frame / 8 track` |

|              |    HOTA |    DetA |    AssA |    LocA |    IDF1 |    MOTA |    MOTP |    FP |    FN |  IDSW |
| ------------ | ------: | ------: | ------: | ------: | ------: | ------: | ------: | ----: | ----: | ----: |
| Bản pre-gold |   `...` |   `...` |   `...` |   `...` |   `...` |   `...` |   `...` | `...` | `...` | `...` |
| Sau rework   | `0.785` | `0.758` | `0.815` | `0.876` | `0.942` | `0.881` | `0.860` |  `44` |  `24` |   `0` |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): `ĐẠT`

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi   |         Frame |  ID | Đã sửa thế nào                            |
| ---------- | ------------: | --: | ----------------------------------------- |
| Bbox treo  |      `79–100` | `5` | `Kết thúc bbox đúng lúc xe rời khung`     |
| Bbox treo  |     `149–151` | `4` | `Kết thúc bbox đúng lúc xe rời khung`     |
| Bbox trôi  |         `115` | `7` | `Kiểm tra lại bbox và thêm keyframe`      |
| Bbox trôi  |         `161` | `8` | `Kiểm tra lại bbox và thêm keyframe`      |
| Thiếu đoạn | `26/33 frame` | `8` | `Kiểm tra và bổ sung đoạn track bị thiếu` |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục                                | Giá trị                                                                |
| ---------------------------------- | ---------------------------------------------------------------------- |
| Python / ultralytics / torch / lap | `Python 3.13.15 / Ultralytics 8.4.145 / Torch 2.11.0+cpu / LAP 0.5.13` |
| weights / hai tracker              | `yolo26n.pt / ByteTrack và BoT-SORT + ReID`                            |
| conf / IoU / imgsz / classes       | `0.25 / 0.70 / 960 / [2, 5, 7]`                                        |
| device                             | `CPU`                                                                  |

| So sánh                           |    HOTA |    DetA |    AssA |    LocA |    IDF1 |    MOTA |    MOTP |    FP |   FN | IDSW |
| --------------------------------- | ------: | ------: | ------: | ------: | ------: | ------: | ------: | ----: | ---: | ---: |
| bạn vs gold                       | `0.785` | `0.758` | `0.815` | `0.876` | `0.942` | `0.881` | `0.860` |  `44` | `24` |  `0` |
| ByteTrack control vs gold         | `0.709` | `0.649` | `0.776` | `0.846` | `0.875` | `0.749` | `0.823` |  `88` | `54` |  `2` |
| BoT-SORT + ReID treatment vs gold | `0.763` | `0.711` | `0.820` | `0.872` | `0.900` | `0.792` | `0.860` |  `91` | `26` |  `2` |
| ReID vs bạn                       | `0.713` | `0.654` | `0.777` | `0.857` | `0.864` | `0.718` | `0.844` | `106` | `61` |  `0` |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`MOTA của mình là 0.881, thấp hơn IDF1 là 0.942. Điều này cho thấy ID được giữ khá tốt, trong khi MOTA còn bị ảnh hưởng bởi FP, FN và ID switch.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`BoT-SORT + ReID tốt hơn ByteTrack về IDF1 (0.900 so với 0.875) và AssA (0.820 so với 0.776). IDSW không thay đổi, cả hai đều có 2 ID switch. ByteTrack có lỗi ở frame 59 và 94, còn BoT-SORT + ReID có lỗi ở frame 87 và 113. Tuy nhiên đây không phải là phép đo causal effect riêng của ReID vì hai bên dùng hai tracker implementation khác nhau.`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`So với ByteTrack, BoT-SORT + ReID làm DetA tăng từ 0.649 lên 0.711 và FN giảm từ 54 xuống 26, nhưng FP tăng nhẹ từ 88 lên 91. Vì vẫn còn nhiều FP/FN nên lỗi không chỉ nằm ở association mà detector cũng còn ảnh hưởng. Cả hai tracker đều có 2 IDSW nên association vẫn còn lỗi.`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`Chưa có đủ bằng chứng hình ảnh để khẳng định chắc chắn một frame cụ thể. Có các track của ReID không khớp track tham chiếu, nhưng cần xem trực tiếp frame trước khi kết luận model sai.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`ReID có một số track không khớp với track tham chiếu, ví dụ track 7 ở frame 16–116. Đây là dấu hiệu cần xem lại đoạn đó, nhưng không nên sửa annotation chỉ dựa vào metric.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`Mình sẽ làm rõ ba điểm: khi xe rời khung thì kết thúc track đúng frame; khi bbox bắt đầu trôi thì thêm keyframe; khi hai xe cắt nhau hoặc bị che thì xem thêm frame trước và sau để giữ đúng ID. Khi kiểm tra, mình sẽ ưu tiên xem ID trước, sau đó xem đầu/cuối track và cuối cùng kiểm tra các frame giữa.`

## 7. Tệp đã nộp

* [x] `annotations/clip_01/gt.txt`
* [x] `annotations/clip_02/gt.txt`
* [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
* [x] `GUIDELINE_MINI.md`
* [x] `outputs/eval_vs_gold.json`
* [x] `outputs/model_bytetrack_clip_01.txt`
* [x] `outputs/model_reid_clip_01.txt`
* [x] `outputs/model_run_config.json`
* [x] `outputs/eval_bytetrack_vs_gold.json`
* [x] `outputs/eval_reid_vs_gold.json`
* [x] `outputs/eval_reid_vs_me.json`
* [ ] `reports/review_partner.md`
