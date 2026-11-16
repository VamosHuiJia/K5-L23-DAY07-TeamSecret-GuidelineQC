# Guideline gán nhãn — v1.1 — Team Secret

- Phiên bản: `v1.1`
- Ngày: 17/11/2026
- Người viết / duyệt: Team Secret / `190` (chờ duyệt).
- Dữ liệu: 40 ảnh đường phố; taxonomy `v1.0`; gán bbox trên CVAT, xuất COCO 1.0.

## 1. Mục tiêu

Tạo nhãn người và phương tiện thật để huấn luyện mô hình phát hiện vật thể 2D. Cần tránh bỏ sót người, nhầm lớp xe và gán tượng hoặc phản chiếu thành vật thật. Bộ ảnh dùng cho thực hành, chưa đại diện đầy đủ cho giao thông thực tế.

## 2. Lớp và thuộc tính

### Lớp

**R-14:** mỗi vật chọn đúng một lớp theo bảng. Vật ngoài taxonomy không gán và không tự đưa vào vùng ignore.

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `person` | Người thật nhìn thấy và nhận dạng được. | Người đi bộ, người ngồi trên xe. | Tượng, ma-nơ-canh, ảnh người. |
| `car` | Ô tô thân chở người: sedan, hatchback, wagon, SUV. | Ô tô con, taxi. | Bán tải là `truck`. |
| `truck` | Xe chở hàng, gồm xe tải, đầu kéo và bán tải. | Xe tải, xe chở ô tô. | Van thân liền là `van`. |
| `van` | Xe thân kín liền khối dạng hộp, không có thùng hàng riêng. | Minivan, van giao hàng. | Bán tải có thùng riêng là `truck`. |
| `bus` | Xe buýt hoặc minibus có cấu tạo khoang hành khách kiểu buýt. | Xe buýt đô thị. | Van thông thường là `van`. |
| `motorcycle` | Xe máy, mô tô có động cơ hai bánh. | Xe tay ga, mô tô. | Xe đạp, tuk-tuk. |
| `bicycle` | Xe đạp không động cơ, vận hành bằng bàn đạp. | Xe đạp thường, xe đạp chở hàng. | Xích lô chở khách là `cyclo`. |
| `tuk_tuk` | Xe ba bánh có động cơ dạng tuk-tuk/auto-rickshaw. | Tuk-tuk chở người hoặc hàng. | Xích lô đạp là `cyclo`. |
| `cyclo` | Xe ba bánh chở khách vận hành bằng bàn đạp. | Xích lô đạp. | Tuk-tuk có động cơ. |
| `ignore_region` | Vùng chứa vật mục tiêu quá nhỏ hoặc cụm không tách được. | Vật cao dưới 20 px, đám đông dày. | Vật lớn rõ, tượng và phản chiếu. |

### Thuộc tính

| Thuộc tính | Các mức cố định | Ý nghĩa và ngưỡng |
|---|---|---|
| `occluded_level` — **R-11** | `0`, `1`, `2`, `3` | `0`: không che; `1`: trên 0–30%; `2`: trên 30–70%; `3`: trên 70% nhưng vẫn nhận dạng chắc. |
| `truncated` — **R-12** | `no`, `yes` | `yes` khi vật bị cắt ở mép ảnh; `no` khi không bị cắt. |
| `can_review` — **R-13** | `false`, `true` | `true` khi còn phân vân về lớp, box hoặc thuộc tính; `false` khi đã xác định theo guideline. |

Ước lượng mức che trên phần hình dạng vật dự kiến nằm trong ảnh, không tính phần ngoài khung. Che khuất và cắt mép được xét riêng. Tối hoặc nhòe không tự động tăng mức che. Gán đủ ba thuộc tính cho box đối tượng; `ignore_region` không cần thuộc tính. Chưa xác định được box thì ghi vấn đề cấp ảnh; chỉ tắt `can_review` sau khi xử lý xong ca đó.

## 3. Quy tắc vẽ box

1. **R-01:** một vật một box, ôm sát phần nhìn thấy; không kéo rộng sang nền, bóng đổ hoặc bù phần bị che.
2. **R-02:** tách người lái, người ngồi sau và phương tiện thành các box riêng nếu nhận dạng được, cao từ 20 px. Người thấy qua kính vẫn gán; không suy đoán người trong xe kín.
3. **R-03:** box xe gồm các bộ phận gắn cố định như gương, bánh, sidecar. Xe máy có sidecar tính một `motorcycle`. Xe chở xe: một `truck` và từng `car` đủ điều kiện riêng.
4. **R-04:** không gộp hàng hóa rời, bao tải hoặc tải chở quá khổ vào box xe.
5. **R-05:** vật bị che vẫn gán phần nhìn thấy nếu nhận dạng chắc. Vật bị che hoàn toàn không gán. Box dừng ở mép ảnh khi bị cắt, đặt `truncated=yes`.
6. **R-06:** box đối tượng cao từ 20 px. Vật mục tiêu dưới 20 px hoặc cụm quá dày không tách được dùng `ignore_region` ôm sát; tránh phủ vật lớn, rõ bên cạnh.
7. **R-07:** loại tượng, ma-nơ-canh, đồ chơi, mô hình, đạo cụ trưng bày, hình in, màn hình và phản chiếu; không tạo ignore riêng cho chúng. Xe thật vẫn gán khi đỗ hoặc trang trí; chưa phân biệt được với đạo cụ thì ghi review.
8. **R-08:** xe đạp, xích lô, tuk-tuk, xe buýt dùng lớp riêng; không ép phương tiện ngoài taxonomy vào lớp gần giống.
9. **R-09:** không gán phần xe chủ gắn camera như nắp ca-pô hoặc mép xe ở tiền cảnh.
10. **R-10:** giữ ảnh không có đối tượng trong COCO; không thêm box giả.

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_002.jpg` | Xe chở ô tô. | R-03, R-06 | Một `truck`; từng ô tô thật đủ điều kiện là `car` riêng. | Xe chở và xe được chở là các vật riêng. |
| `img_004.jpg`, `img_035.jpg` | Tuk-tuk chở người, hàng. | R-02, R-04, R-14 | Gán `tuk_tuk`, tách người; box xe không bao hàng rời. | Phân biệt xe, người và tải hàng. |
| `img_011.jpg` | Phương tiện trang trí, nghi là đạo cụ. | R-07, R-13 | Xe thật vẫn gán; chưa phân biệt được thì cần reviewer chốt. | Trang trí hoặc nghi không hoạt động chưa đủ để loại xe. |
| `img_006.jpg`, `img_023.jpg`, `img_034.jpg` | Xe đạp, xích lô, xe máy. | R-02, R-14 | Chọn lớp theo cấu tạo; người được gán riêng nếu đủ điều kiện. | Phân biệt phương tiện có động cơ và vận hành bằng bàn đạp. |
| `img_007.jpg`, `img_016.jpg`, `img_031.jpg` | C073, C138, C222 khác lớp. | R-14 | Chưa chốt từng ca; đối chiếu thân xe với định nghĩa car/truck/bus. | Không phân lớp chỉ theo kích thước. |
| `img_017.jpg`, `img_018.jpg`, `img_020.jpg`, `img_027.jpg`, `img_033.jpg`, `img_038.jpg` | Tượng, hình in, phản chiếu, đồ chơi. | R-07 | Loại các hình thể giả; xét riêng vật thật nhìn trực tiếp trong ảnh. | Phạm vi chỉ gồm người và phương tiện thật. |
| `img_002.jpg` | C007–C013 khác mức che. | R-11, R-12 | Xem lại phần bị che, áp dụng ngưỡng; chốt giá trị riêng từng ca. | Box gần nhau vẫn có thể khác thuộc tính. |
| `img_019.jpg` | C147–C149 bất đồng vùng ignore. | R-06 | Chưa chốt từng vùng; kiểm chiều cao vật và biên vùng ignore. | Tránh bỏ qua vật đủ rõ để gán riêng. |

Ca chưa chốt giữ `can_review` hoặc ghi vấn đề cấp ảnh. Quyết định cụ thể được ghi theo `case_id` trong `calibration_decisions.csv`.

## 5. QC và phiên bản

**R-15 — Ngưỡng QC:**

- Ghép box 1–1 với IoU ≥ 0,50. Đối chiếu vị trí: box cao ≥ 50 px cần IoU ≥ 0,85; thấp hơn 50 px cần ≥ 0,70.
- IAA mục tiêu: IoU trung bình ≥ 0,85 và đồng thuận lớp ≥ 95%, tính trên các cặp đã ghép; box chưa ghép xem riêng.
- Reviewer `190` cùng nhóm kiểm sáu loại ca: bỏ sót/thừa, lệch box, sai thuộc tính, sai lớp, cờ review và bất đồng ignore. Ghi người chốt, mã quy tắc, bản sửa, kết quả kiểm lại và phạm vi `anh`/`lo`.
- Duyệt đủ 40 ảnh trước khi bàn giao. Release khi ảnh lỗi ≤ 2%, không lỗi lặp và đủ bằng chứng; thiếu bằng chứng thì Hold, vượt ngưỡng hoặc lỗi lặp thì Rework. Một ảnh lỗi/40 ảnh = 2,5%, cần sửa và duyệt vòng mới.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| `v0` | Bản khởi đầu, năm lớp đối tượng, thuộc tính còn định tính. | Thiếu quy tắc ca biên. |
| `v1.0` | Thêm bus, bicycle, tuk_tuk, cyclo; thống nhất mức che và phạm vi box. | Xe chở xe, tuk-tuk, xe đạp/xích lô, xe buýt và vật giả. |
| `v1.1` | Giữ nguyên ý nghĩa v1.0; gắn R-01–R-15 và bổ sung ca cần đối chiếu. Loại thay đổi: `editorial`. | IAA có 321 ca trên 25 ảnh, gồm các ca ở Mục 4. |

Bảng quyết định calibration đang chờ nhóm hoàn thiện. Giữ nguyên nhãn độc lập và IAA trước sửa; chỉ xác nhận hoàn tất khi các ca đã được chốt và kiểm lại. Nếu đổi ý nghĩa quy tắc, cập nhật `guideline_change.json` thành `semantic` và rà cả lô.
