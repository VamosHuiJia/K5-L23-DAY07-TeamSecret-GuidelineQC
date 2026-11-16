# Guideline gán nhãn — v1.1 — Team Secret

- Phiên bản: `v1.1`
- Ngày: 17/11/2026
- Người viết / duyệt: Team Secret / `190` (chờ duyệt).
- Dữ liệu: 40 ảnh đường phố; taxonomy `v1.0`; gán bbox trên CVAT, xuất COCO 1.0.

## 1. Mục tiêu

Gán người và phương tiện thật để huấn luyện mô hình phát hiện vật thể 2D. Lỗi cần tránh: bỏ sót người, nhầm lớp xe và gán vật giả thành vật thật.

## 2. Lớp và thuộc tính

### Lớp

**R-14:** mỗi vật chọn một lớp. Vật ngoài taxonomy không gán, không đưa vào ignore.

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `person` | Người thật nhận dạng được. | Người đi bộ. | Tượng, ảnh người. |
| `car` | Ô tô con: sedan, hatchback, wagon, SUV. | Taxi. | Bán tải là `truck`. |
| `truck` | Xe chở hàng có thùng riêng hoặc đầu kéo. | Xe tải, bán tải. | Van thân liền. |
| `van` | Xe thân kín dạng hộp, không có thùng riêng. | Van giao hàng. | Bán tải. |
| `bus` | Xe buýt, kể cả minibus có khoang kiểu buýt. | Xe buýt đô thị. | Van thông thường. |
| `motorcycle` | Xe hai bánh có động cơ. | Xe tay ga. | Xe đạp. |
| `bicycle` | Xe đạp không động cơ, dùng bàn đạp. | Xe đạp chở hàng. | Xích lô chở khách. |
| `tuk_tuk` | Xe ba bánh có động cơ dạng tuk-tuk. | Tuk-tuk chở khách. | Xích lô đạp. |
| `cyclo` | Xe ba bánh chở khách bằng sức đạp. | Xích lô đạp. | Tuk-tuk. |
| `ignore_region` | Vùng vật mục tiêu quá nhỏ hoặc không tách được. | Vật cao dưới 20 px, đám đông dày. | Vật lớn rõ, vật giả. |

### Thuộc tính

| Thuộc tính | Các mức cố định | Ý nghĩa và ngưỡng |
|---|---|---|
| `occluded_level` — **R-11** | `0`, `1`, `2`, `3` | `0`: không che; `1`: 0% < che ≤ 30%; `2`: 30% < che ≤ 70%; `3`: che > 70%, vẫn nhận dạng chắc. |
| `truncated` — **R-12** | `no`, `yes` | `yes` khi vật bị cắt ở mép ảnh; `no` khi không bị cắt. |
| `can_review` — **R-13** | `false`, `true` | `true`: chưa chắc lớp, box hoặc thuộc tính; `false`: đã xác định theo guideline. |

Tính mức che trên hình dạng vật dự kiến trong ảnh, không tính phần ngoài khung; tối/nhòe không tính là che. Box đối tượng có đủ ba thuộc tính; `ignore_region` không cần.

## 3. Quy tắc vẽ box

1. **R-01:** một vật một box, sát phần nhìn thấy; không lấy nền, bóng đổ hay bù phần bị che.
2. **R-02:** tách người và xe thành box riêng nếu nhận dạng được, cao ≥ 20 px. Người thấy qua kính vẫn gán; không suy đoán người trong xe kín.
3. **R-03:** box xe gồm bộ phận cố định: gương, bánh, sidecar. Xe máy kèm sidecar tính một `motorcycle`. Xe chở xe: một `truck` và từng `car` đủ điều kiện.
4. **R-04:** không gộp hàng rời, bao tải hoặc tải quá khổ vào box xe.
5. **R-05:** bị che vẫn gán phần thấy rõ; che hoàn toàn thì bỏ. Bị cắt mép: box dừng ở mép ảnh, `truncated=yes`.
6. **R-06:** vật cao ≥ 20 px gán riêng; dưới 20 px hoặc cụm không tách được dùng `ignore_region`, tránh phủ vật lớn rõ.
7. **R-07:** bỏ tượng, ma-nơ-canh, đồ chơi, mô hình, đạo cụ, hình in, màn hình và phản chiếu; không tạo ignore cho chúng. Xe thật đỗ/trang trí vẫn gán; chưa phân biệt với đạo cụ thì review.
8. **R-08:** xe đạp, xích lô, tuk-tuk, xe buýt dùng lớp riêng; không ép vật ngoài taxonomy vào lớp gần giống.
9. **R-09:** bỏ phần xe gắn camera (ego), như nắp ca-pô ở tiền cảnh.
10. **R-10:** giữ ảnh không có đối tượng trong COCO, không thêm box giả.

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_002.jpg` | Xe chở ô tô. | R-03 | Một `truck`, từng `car` đủ điều kiện riêng. | Các vật độc lập. |
| `img_004.jpg`, `img_035.jpg` | Tuk-tuk chở người, hàng. | R-02, R-04 | Gán `tuk_tuk`, tách người, bỏ hàng rời khỏi box xe. | Không gộp người/hàng vào xe. |
| `img_011.jpg` | Xe trang trí hay đạo cụ? | R-07, R-13 | Xe thật vẫn gán; chưa rõ thì chờ reviewer. | Trang trí chưa đủ để loại xe. |
| `img_006.jpg`, `img_023.jpg`, `img_034.jpg` | Xe đạp, xích lô, xe máy. | R-14 | Chọn lớp theo động cơ, bàn đạp và cấu tạo xe. | Không gộp chung thành xe máy. |
| `img_007.jpg`, `img_016.jpg`, `img_031.jpg` | C073, C138, C222 khác lớp. | R-14 | Chưa chốt; đối chiếu thân xe với bảng lớp. | Không chọn lớp chỉ theo kích thước. |
| `img_017.jpg`, `img_018.jpg`, `img_020.jpg`, `img_027.jpg`, `img_033.jpg`, `img_038.jpg` | Vật giả, phản chiếu. | R-07 | Loại vật giả; giữ vật thật nhìn trực tiếp. | Chỉ gán người, xe thật. |
| `img_002.jpg` | C007–C013 khác mức che. | R-11, R-12 | Chưa chốt; kiểm từng box theo ngưỡng che/cắt mép. | Không suy thuộc tính từ độ giống box. |
| `img_019.jpg` | C147–C149 khác vùng ignore. | R-06 | Chưa chốt; đo chiều cao vật, kiểm biên ignore. | Tránh bỏ vật đủ điều kiện gán riêng. |

Ca chưa chốt giữ `can_review`; chưa xác định được box thì ghi vấn đề cấp ảnh. Chỉ tắt cờ sau khi xử lý, ghi quyết định theo `case_id` trong `calibration_decisions.csv`.

## 5. QC và phiên bản

**R-15 — Ngưỡng QC:**

- Ghép box 1–1: IoU ≥ 0,50. Box cao ≥ 50 px cần IoU ≥ 0,85; nhỏ hơn cần ≥ 0,70.
- IAA: IoU trung bình ≥ 0,85, đồng thuận lớp ≥ 95% trên cặp đã ghép; kiểm riêng box chưa ghép.
- Reviewer `190` chốt ca mơ hồ cùng nhóm: bỏ sót/thừa, lệch box, sai lớp, sai thuộc tính, cờ review, ignore. Ghi người chốt, rule, bản sửa, kiểm lại và phạm vi `anh`/`lo`.
- Duyệt đủ 40 ảnh. Release: ảnh lỗi ≤ 2%, không lỗi lặp, đủ bằng chứng. Thiếu bằng chứng: Hold; vượt ngưỡng/lỗi lặp: Rework. Một ảnh lỗi/40 = 2,5%, phải sửa và duyệt lại.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| `v0` | Năm lớp đối tượng, thuộc tính định tính. | Thiếu quy tắc ca biên. |
| `v1.0` | Thêm bus, bicycle, tuk_tuk, cyclo; chốt ngưỡng che và box. | Xe chở xe, tuk-tuk, xích lô, vật giả. |
| `v1.1` | Giữ nghĩa v1.0, thêm R-01–R-15 và ca đối chiếu (`editorial`). | 321 ca trên 25 ảnh từ IAA, xem Mục 4. |

Bảng quyết định còn chờ nhóm hoàn thiện, chưa xác nhận xong calibration. Giữ nhãn và IAA trước sửa; nếu đổi nghĩa quy tắc, khai báo `semantic` và rà cả lô.
