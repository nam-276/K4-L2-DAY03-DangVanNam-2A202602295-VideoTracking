
# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Đặng Văn Nam`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán                            | Không gán                                                   |
| ------------------------------- | ------------------------------------------------------------- |
| xe con, SUV, taxi, xe bán tải | người đi bộ                                               |
| van, minivan                    | xe đạp                                                      |
| xe buýt, minibus               | **xe máy / mô tô**                                   |
| xe tải, xe đầu kéo          | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có):

* Chỉ gán đối tượng là phương tiện bốn bánh thực sự xuất hiện trong cảnh chính.
* Không dùng kết quả detector/model để quyết định một đối tượng có phải `vehicle` hay không; model chỉ được dùng để hỗ trợ tìm vùng cần kiểm tra sau khi annotation đã được khóa.

## 2. Luật ID — phần quan trọng nhất

| Tình huống                          | Luật của nhóm                                                                                                                                                   | Vì sao                                                                                                                           |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che**dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps)                                                        | Xe vẫn có thể được nhận dạng là cùng một đối tượng dựa trên vị trí, hình dạng và chuyển động liên tục. |
| Xe bị che lâu hơn ngưỡng trên   | Nếu không còn đủ bằng chứng để xác định chắc chắn đó là cùng xe, kết thúc track cũ và tạo**track mới** khi xe xuất hiện lại.      | Tránh gán nhầm ID cho một xe khác sau thời gian bị che quá lâu.                                                          |
| Xe rời khung hình rồi quay lại    | mặc định:**track mới**                                                                                                                                   | Không có bằng chứng liên tục giữa hai lần xuất hiện; ưu tiên tính nhất quán và tránh nối nhầm identity.        |
| Hai xe cắt nhau / chồng lên nhau   | giữ nguyên ID dựa trên vị trí, hướng chuyển động và đặc điểm hình ảnh trước/sau khi giao nhau; kiểm tra frame trước và sau vùng giao nhau | Tránh đổi ID khi hai xe đi qua nhau hoặc bị che một phần.                                                                 |

## 3. Luật bbox

| Tình huống                                   | Luật của nhóm                                                                                                                                                                                                                                                   |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Xe bị cắt bởi rìa ảnh                     | bbox chạm đúng rìa, không đoán phần ngoài ảnh                                                                                                                                                                                                            |
| Xe bị xe khác che một phần                 | bbox ôm phần**nhìn thấy được**                                                                                                                                                                                                                        |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn:**chỉ bắt đầu khi có đủ hình dạng/đặc điểm để xác định chắc chắn là vehicle**, không dùng một ngưỡng kích thước pixel cứng' |
| Xe đang đỗ, không di chuyển               | vẫn gán và giữ cùng ID nếu xe tiếp tục là đối tượng quan sát được trong cảnh                                                                                                                                                                     |
| Keyframe đặt dày ở đâu                   | đặt dày hơn tại đoạn xe đổi hướng, tăng/giảm kích thước nhanh, bị che, đi sát xe khác, hoặc bbox bắt đầu trôi; đoạn chuyển động ổn định có thể dùng khoảng cách keyframe thưa hơn                                          |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1

* Clip / frame / ID: `clip_01 / frame 149–151 / ID 4`
* Tình huống: Xe đã rời khỏi vùng quan sát nhưng bbox của track vẫn còn tồn tại.
* Quyết định: Đánh dấu `outside` và kết thúc track tại thời điểm xe thực sự rời khỏi khung.
* Lý do: Không được để bbox tiếp tục tồn tại sau khi đối tượng không còn nhìn thấy; nếu không sẽ tạo false positive/bbox treo.

### Ca 2

* Clip / frame / ID: `clip_01 / frame 83 / ID 5`
* Tình huống: Bbox của xe bắt đầu lệch/trôi so với vị trí thực tế của xe.
* Quyết định: Kiểm tra lại vị trí bbox và bổ sung/chỉnh keyframe tại vùng bbox thay đổi.
* Lý do: Bbox phải bám theo phần xe thực sự nhìn thấy; không để nội suy kéo bbox lệch khỏi đối tượng.

### Ca 3

* Clip / frame / ID: `clip_01 / frame 115–122 / ID 6`
* Tình huống: Xe chuyển động làm bbox thay đổi rõ rệt và có nguy cơ bị trôi nếu chỉ dựa vào các keyframe thưa.
* Quyết định: Kiểm tra liên tiếp các frame trong vùng này và bổ sung/chỉnh keyframe để bbox bám sát xe.
* Lý do: Đây là vùng chuyển động cần kiểm tra dày hơn để giữ localization ổn định.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

* **Quy tắc `outside` cần ghi rõ hơn:** khi xe rời khỏi ảnh thì phải kết thúc track tại frame phù hợp, không để bbox treo thêm các frame sau đó.
* **Quy tắc keyframe cần cụ thể hơn:** phải tăng mật độ keyframe tại các đoạn bbox trôi, xe đổi hướng, thay đổi kích thước nhanh hoặc có occlusion; không chỉ đặt keyframe đều theo thời gian.
* **Cần kiểm tra frame giữa:** không chỉ kiểm tra frame đầu và cuối của track. Phải tua qua frame giữa để phát hiện bbox drift và lỗi ID.
* **Cần giữ ID xuyên qua occlusion ngắn:** mặc định giữ ID trong tối đa 25 frame nếu có đủ bằng chứng đó vẫn là cùng xe.
* **Không nối lại ID sau khi xe rời khung:** khi xe xuất hiện lại sau khi đã rời khỏi frame, mặc định tạo track mới.
* **Model không phải ground truth:** kết quả ByteTrack/ReID chỉ dùng làm evidence để tìm điểm bất thường; quyết định cuối cùng phải dựa trên frame/video gốc và guideline.
* **Khi hai xe giao nhau:** phải kiểm tra cả đoạn trước và sau vùng giao nhau để bảo đảm identity không bị đổi nhầm.
