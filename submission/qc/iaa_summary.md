# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 3
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 252 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 252 | 0.906 | 98.8% | 54.4% | 89.3% | 24/103 | 0 | ĐẠT |
| 199|204 | 13 | 94 | 1.000 | 100.0% | 100.0% | 100.0% | 0/0 | 0 | ĐẠT |
| 199|215 | 13 | 101 | 0.847 | 99.0% | 22.8% | 78.2% | 13/59 | 0 | CHƯA ĐẠT |
| 204|215 | 14 | 57 | 0.856 | 96.5% | 35.1% | 91.2% | 11/44 | 0 | ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 321 ({'bo_sot_thua': 109, 'iou_thap': 53, 'sai_thuoc_tinh': 125, 'sai_lop': 3, 'co_can_review': 12, 'bat_dong_ignore': 19}), trên 25 ảnh cần calibration.
