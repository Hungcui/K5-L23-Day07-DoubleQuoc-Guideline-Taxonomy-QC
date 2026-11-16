# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 2
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 285 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 285 | 0.966 | 93.3% | 100.0% | 98.9% | 35/30 | 5 | CHƯA ĐẠT |
| 038|039 | 40 | 285 | 0.966 | 93.3% | 100.0% | 98.9% | 35/30 | 5 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 116 ({'sai_lop': 19, 'bo_sot_thua': 51, 'iou_thap': 19, 'bat_dong_ignore': 24, 'sai_thuoc_tinh': 3}), trên 27 ảnh cần calibration.
