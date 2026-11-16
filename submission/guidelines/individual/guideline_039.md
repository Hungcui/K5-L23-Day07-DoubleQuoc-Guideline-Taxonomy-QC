# Guideline gán nhãn — v0 (bản khởi đầu)

> **Đây là bản do giảng viên viết nhanh. Nó CHƯA đủ dùng.**
> Đừng sửa file này. Hãy copy sang `submission/guidelines/individual/` rồi sửa bản của bạn
> (xem `GUIDE.md`, Pha 1).

- Phiên bản: v0
- Loại nhãn: bounding box (hộp chữ nhật) trên ảnh đường phố
- Công cụ: CVAT, xuất COCO 1.0

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện **người và phương tiện** trên ảnh đường phố
(ví dụ cho camera giao thông hoặc xe tự lái).

## 2. Lớp và thuộc tính

### Lớp (mỗi vật thuộc đúng một lớp)

| Lớp | Định nghĩa |
|---|---|
| `car` | Ô tô con. |
| `truck` | Xe tải. |
| `van` | Xe thùng kín cỡ nhỏ. |
| `motorcycle` | Xe máy, mô tô. |
|`bicycle`|Xe đạp|
| `person` | Người. |
| `ignore_region` | Vùng không cần gán từng vật. |
|`bus`|Xe chở người, phương tiện cong cộng cỡ lớn.|
|`other_vehicle`|Các phương tiện khác (không xuất hiện ở trên).|

### Thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Mức bị che: 0 là thấy rõ, 1 là bị che ít, 2 là bị che nhiều, 3 là không rõ. |
| `truncated` | `no`, `yes` | Vật bị cắt ở mép ảnh. |
| `can_review` | tick / không | Tick khi bạn không chắc, để người khác xem lại. |

## 3. Quy tắc vẽ box

1. Gán box cho mọi người và phương tiện trong ảnh.
2. Box ôm sát vật.
3. Chọn đúng lớp và thuộc tính.
4. Nếu người điều khiển xe máy/xe đạp thì vẽ 1 box bicycle/motorcycle bao gồm cả phương tiện và người điều khiển.
5. Không gắn box cho vật không có thật (hình ảnh quảng cáo, hình ảnh phản chiếu qua gương, mô hình, tượng,...).
6. Đối với vật/người bị che khuất quá 80% - không gắn box.
7. Vẽ box ôm sát cơ thể người (không bao gồm balo, túi,...).
8. Đối với người đi xe máy/xe đạp chở theo hàng cồng kềnh, box chỉ vẽ đủ (ôm sát người, phương tiện, không bao gồm hàng hoá).
9. Đối với xe tải chở hàng, box chỉ bao gồm thân xe, không chứa hàng hoá.
10. Vật/người bị che khuát một phần, vẽ box ước lượng được cả phần che khuất, không chỉ vẽ mỗi phần nhìn thấy.

## 4. Ca khó

(chưa có)

## 5. QC và phiên bản

- Mọi thay đổi làm đổi nhãn đều tăng phiên bản guideline.
- Ca mơ hồ: tick `can_review`.
