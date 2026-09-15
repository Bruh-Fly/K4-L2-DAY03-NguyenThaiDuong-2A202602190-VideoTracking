# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Nguyễn Thái Dương`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán                           | Không gán                                           |
| ----------------------------- | --------------------------------------------------- |
| xe con, SUV, taxi, xe bán tải | người đi bộ                                         |
| van, minivan                  | xe đạp                                              |
| xe buýt, minibus              | **xe máy / mô tô**                                  |
| xe tải, xe đầu kéo            | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): `Chỉ gán phương tiện thật đang xuất hiện trong ảnh. Nếu không xác định chắc chắn là xe bốn bánh thì không cố đoán.`

## 2. Luật ID — phần quan trọng nhất

| Tình huống                       | Luật của nhóm                                                                                          | Vì sao                                                     |
| -------------------------------- | ------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------- |
| Xe bị che một phần rồi hiện lại  | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps)            | xe vẫn có thể nhận ra là cùng một xe                       |
| Xe bị che lâu hơn ngưỡng trên    | kiểm tra các frame trước và sau; nếu vẫn xác định được cùng xe thì giữ ID, nếu không thì tạo track mới | tránh nối nhầm hai xe                                      |
| Xe rời khung hình rồi quay lại   | mặc định: **track mới**                                                                                | không đủ bằng chứng để chắc chắn là cùng một lần xuất hiện |
| Hai xe cắt nhau / chồng lên nhau | xem thêm các frame trước và sau đoạn cắt để giữ đúng ID                                                | đây là đoạn dễ đổi nhầm ID                                 |

## 3. Luật bbox

| Tình huống                             | Luật của nhóm                                                                                               |
| -------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| Xe bị cắt bởi rìa ảnh                  | bbox chạm đúng rìa, không đoán phần ngoài ảnh                                                               |
| Xe bị xe khác che một phần             | bbox ôm phần **nhìn thấy được**                                                                             |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `chưa đặt ngưỡng số cụ thể` |
| Xe đang đỗ, không di chuyển            | vẫn gán nếu xe thuộc phạm vi của bài                                                                        |
| Keyframe đặt dày ở đâu                 | đặt thêm khi xe đổi hướng nhanh, bị che hoặc bbox bắt đầu trôi                                              |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1

* Clip / frame / ID: `clip_01 / frame 115 / ID 7`
* Tình huống: `bbox bị lệch khi xe di chuyển`
* Quyết định: `kiểm tra lại bbox và thêm keyframe`
* Lý do: `bbox bắt đầu trôi giữa các keyframe`

### Ca 2

* Clip / frame / ID: `clip_01 / frame 161 / ID 8`
* Tình huống: `bbox bị lệch so với gold`
* Quyết định: `kiểm tra lại vị trí bbox và thêm keyframe nếu cần`
* Lý do: `tránh để bbox trôi trong đoạn xe di chuyển`

### Ca 3

* Clip / frame / ID: `clip_01 / frame 79–100 / ID 5`
* Tình huống: `bbox vẫn còn sau khi xe đã rời khỏi khung hình`
* Quyết định: `kết thúc track đúng lúc xe rời khung`
* Lý do: `tránh để bbox treo ở những frame không còn xe`

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

* `Khi xe rời khung hình, phải kiểm tra frame cuối cùng và kết thúc track đúng lúc.`
* `Khi bbox bắt đầu trôi giữa hai keyframe, thêm keyframe ở đoạn đó
