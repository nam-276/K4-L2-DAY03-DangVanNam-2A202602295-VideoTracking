# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: Đặng Văn Nam
Ngày: 15/9/2026

---

## 1. Quá trình gán nhãn

| Mục                                 | Giá trị |
| ------------------------------------ | --------- |
| Công cụ                            | CVAT      |
| Thời gian gán`clip_02` (warm-up) | 25 phút  |
| Thời gian gán`clip_01`           | 60 phút  |
| Số track đã vẽ trong`clip_01`  | 8         |
| Số keyframe trung bình mỗi track  | 72,675    |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe trắng nhỏ đi sát bus .** Buschiếm gần hết bề ngang; xe trắng chỉ lộ phần đầu. Bắt đầu track từ frame **85** khi đã chắc là xe bốn bánh; gold bắt đầu sớm hơn ở frame **79**. Giữ ID riêng, không gộp vào bus.
2. **Bus cắt xe bạc và xe tải .** Ba xe chồng bbox. Làm xong từng xe, bbox chỉ ôm phần nhìn thấy, không đổi ID khi cắt nhau.
3. **Entry/exit ở rìa ảnh.** Bus vào từ phải (ID 4, frame 52) rồi ra trái; xe đỏ (ID 8) ra phải. Bbox chạm rìa, không đoán ngoài khung. Gold kết thúc ID 4 ở 148 và ID 8 ở 168; bản của tôi còn sliver vài frame sau đó vì vật thể vẫn còn thấy.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: 8 ID ổn định, không nhấp nháy; SUV trắng đỗ (ID 3) giữ một ID suốt 190 frame; không gán người / xe máy / biển báo.
- Lượt 2: ID 1 kết thúc frame 11 (ra trái); ID 2 còn sliver ở rìa trái tới 45; ID 5/7 vào muộn hơn lúc xe đã rõ; ID 4/8 còn bbox nhỏ sau khi gold đã `outside`.
- Lượt 3: SUV đỗ ít trôi (ít keyframe); bus đổi hướng/cắt xe khác cần keyframe dày hơn; giữa hai keyframe của ID 4–5 bbox vẫn ôm phần nhìn thấy.

Kiểm chéo với: `tự QC (lab SOLO, không pair)`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `N/A (SOLO)`. Số lỗi tự QC ghi vào finding: `4`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Không có reviewer thứ hai. Khác với gold chủ yếu ở **ngưỡng bắt đầu xe nhỏ** (ID 5, 7) và **frame `outside` khi còn sliver ở rìa**. Đã bổ sung hai luật này vào `GUIDELINE_MINI.md`.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence                                               | Giá trị                                                            |
| ------------------------------------------------------ | -------------------------------------------------------------------- |
| SHA-256 từ`evidence/pre-gold/clip_01/manifest.json` | `4dc2fef83936644c6d01c85433dca7f8dd7e3d9ab7145f53a7fd9c1452cd3bb0` |
| Thời điểm khóa                                     | `2026-09-15T04:01:11.851118+00:00` (11:01 ICT)                     |
| Số row / frame / track trước khi mở reference      | `568` / `190` / `8` (ID 1–8)                                  |

`annotations/clip_01/gt.txt` **trùng** snapshot pre-gold (hash giống nhau). Không export MOT mới sau khi mở gold.

|               |   HOTA |   DetA |   AssA |   LocA |   IDF1 |   MOTA |   MOTP | FP | FN | IDSW |
| ------------- | -----: | -----: | -----: | -----: | -----: | -----: | -----: | -: | -: | ---: |
| Bản pre-gold | 0.8409 | 0.8276 | 0.8561 | 0.8881 | 0.9728 | 0.9459 | 0.8778 | 13 | 18 |    0 |
| Sau rework    | 0.8409 | 0.8276 | 0.8561 | 0.8881 | 0.9728 | 0.9459 | 0.8778 | 13 | 18 |    0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi              | Frame    | ID                     | Đã sửa thế nào                                                                                                         |
| ----------------------- | -------- | ---------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Gold gọi là bbox treo | 149–151 | 4                      | Không cắt track: frame 149 bus vẫn còn ở rìa trái. Coi gold kết thúc sớm 2–3 frame; giữ bbox phần nhìn thấy. |
| Gold gọi là bbox treo | 169–171 | 8 (xe đỏ, gold ID 8) | Không cắt 169–170 (còn sliver); frame 171 chỉ còn ~5 px — nếu gán lại sẽ`outside` sớm hơn 1–2 frame.        |
| Bắt đầu muộn        | 79–84   | 5 (xe trắng sau bus)  | Không re-export: đã qua cổng. Nếu gán lại, start từ lúc đầu xe trắng lộ (~79), không đợi 85.                |
| Bắt đầu muộn        | 101–111 | 7 (xe bạc, gold ID 6) | Không re-export. Xe bạc đã rõ ~101–106; bản này start 112.                                                          |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục                               | Giá trị                                                                                                                                          |
| ---------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Python / ultralytics / torch / lap | `3.13.15` / `8.4.145` / `2.11.0+cpu` / `0.5.13`                                                                                            |
| weights / hai tracker              | `yolo26n.pt` / ByteTrack `bytetrack.yaml` và BoT-SORT+ReID `configs/trackers/botsort-reid.yaml` (`with_reid: true`, `gmc_method: none`) |
| conf / IoU / imgsz / classes       | `0.25` / `0.7` / `960` / COCO `[2, 5, 7]` (car, bus, truck)                                                                                |
| device                             | `cpu` (`persist: true`, 190 frame)                                                                                                             |

| So sánh                  |   HOTA |  DetA |  AssA |  LocA |   IDF1 |  MOTA |   MOTP | FP | FN | IDSW |
| ------------------------- | -----: | ----: | ----: | ----: | -----: | ----: | -----: | -: | -: | ---: |
| bạn vs gold              | 0.8409 | 0.789 | 0.844 | 0.857 | 0.9728 | 0.941 | 0.8778 | 21 | 13 |    0 |
| ByteTrack control vs gold |  0.709 | 0.649 | 0.776 | 0.846 | 0.8746 | 0.749 | 0.8226 | 88 | 54 |    2 |
| BoT-SORT + ReID vs gold   |  0.763 | 0.711 | 0.820 | 0.872 | 0.9001 | 0.792 | 0.8595 | 91 | 26 |    2 |
| ReID vs bạn              |  0.769 | 0.707 | 0.838 | 0.880 | 0.9038 | 0.766 | 0.9183 | 96 | 39 |    1 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

IDF1 **cao hơn** MOTA: `0.971` vs `0.941`. IDSW = 0, AssA = 0.844 — identity ổn. Khoảng cách đến 1.0 nằm ở coverage: FP 21 + FN 13 (start muộn ID 5/7, endpoint khác gold ở ID 4/8). Đây **không** phải case “MOTA cao, IDF1 thấp”.

MOTA cộng `FP + FN + IDSW` rồi chia cho số bbox gold; mỗi ID switch chỉ đếm **một lần**, nên cắt một xe thành hai ID gần như không làm MOTA sụt, trong khi IDF1/AssA phạt cả quãng đời bị gán sai. Ở bản này không có IDSW nên MOTA thấp hơn IDF1 chỉ vì FP/FN, không vì đổi ID.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

So với gold, BoT-SORT + ReID tốt hơn ByteTrack ở:

- **IDF1:** 0.900 so với 0.875, tăng 0.025.
- **AssA:** 0.820 so với 0.776, tăng 0.044.
- **HOTA:** 0.763 so với 0.709, tăng 0.054.
- **MOTA:** 0.792 so với 0.749.
- **MOTP:** 0.860 so với 0.823.

Tuy nhiên **IDSW không giảm**: cả hai đều có 2 ID switch. ByteTrack đổi ID ở frame 59 và 94; ReID đổi ID ở frame 87 và 113. ReID cũng có 16 track, bằng ByteTrack, nên không loại bỏ hoàn toàn hiện tượng tách track.

Một sequence đáng chú ý là quanh **frame 104–113**, nơi ReID vẫn có lỗi association nhưng cách theo dõi khác ByteTrack: ReID có ID switch ở frame 113 của gold track 6, trong khi ByteTrack có ID switch ở frame 94 của gold track 5. Điều này cho thấy treatment thay đổi association nhưng không đảm bảo loại bỏ ID switch.

Quan trọng: **không thể kết luận toàn bộ chênh lệch là causal effect của ReID**, vì notebook dùng hai tracker implementation khác nhau: ByteTrack và BoT-SORT + ReID. Đây là system comparison chứ không phải một thí nghiệm cô lập riêng tác động của ReID.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

So với ByteTrack, ReID có **DetA tăng từ 0.649 lên 0.711** và **FN giảm mạnh từ 54 xuống 26**, nhưng **FP tăng nhẹ từ 88 lên 91**. Điều này cho thấy ReID treatment trong lần chạy này không chỉ khác về association mà còn tạo ra kết quả detection/track output khác, vì toàn bộ tracker implementation ảnh hưởng đến các detection được giữ trong output.

Các lỗi còn lại có cả hai nhóm:

- **Detector/coverage:** FN và bbox lệch; ReID còn 1 trường hợp model bắt thiếu đoạn của gold track 6, trong khi ByteTrack có 3 trường hợp bắt thiếu đoạn.
- **Association:** cả hai đều có 2 ID switch và 3 trường hợp tách track.

Vì vậy không nên kết luận lỗi còn lại chỉ là detector hoặc chỉ association. Với ByteTrack, DetA thấp và FN cao cho thấy vấn đề coverage/detection khá rõ; với ReID, association vẫn còn lỗi dù AssA/IDF1 tốt hơn.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Một ví dụ rõ nhất từ evidence là **frame 149–151, ID 4** trong annotation của tôi: evaluator xác định đây là **bbox treo**, tức ID vẫn tồn tại sau khi xe đã rời khung. Đây là lỗi annotation đã được phát hiện và xử lý.

Tuy nhiên, notebook **không lưu evidence frame-by-frame trực tiếp để đối chiếu annotation với ReID ở frame 149–151**, nên không đủ căn cứ để khẳng định ReID sai ở chính frame này. Không nên tự tạo một ví dụ model-vs-annotation khi notebook không cung cấp evidence.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

ReID-vs-me cho thấy bất đồng rõ ở **frame 87**, nơi track bản A 5 chuyển từ ID 17 sang ID 18; đây là một ID mismatch được evaluator ghi nhận. Ngoài ra, có nhiều bbox chỉ xuất hiện ở model, đặc biệt ID 7 trong frame 16–116, ID 27 trong frame 106–121 và ID 38 trong frame 158–178.

Những track model-only kéo dài nhiều frame cần được xem lại bằng hình ảnh gốc trước khi sửa annotation. Theo guideline của notebook, một track model tồn tại lâu nhưng không tương ứng với xe thật có thể là FP, ví dụ vật thể tĩnh bị detector nhầm thành xe. Vì notebook không lưu ảnh kiểm tra kết luận cuối cùng cho từng track trên, tôi không sửa annotation chỉ dựa vào việc ReID khác annotation

## 7. Tệp đã nộp

- [X] `annotations/clip_01/gt.txt`
- [X] `annotations/clip_02/gt.txt`
- [X] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [X] `GUIDELINE_MINI.md` đã điền
- [X] `outputs/eval_vs_gold.json`
- [X] `outputs/model_bytetrack_clip_01.txt`
- [X] `outputs/model_reid_clip_01.txt`
- [X] `outputs/model_run_config.json`
- [X] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [X] `reports/review_partner.md`
- [X] `reports/REPORT.md` (file này)
