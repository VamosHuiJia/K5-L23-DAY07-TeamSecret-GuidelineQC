# Guideline gán nhãn — v0.1 đề xuất — 190

> Bộ lớp trong `individual/classes.md` do người dùng định nghĩa và đã xác nhận. Các quy tắc và quyết định cho từng ca là bản đề xuất; người học cần tự xem ảnh và quyết định nhãn theo quy định Pha 1.

- Phiên bản: v0.1 đề xuất
- Ngày: 2026-11-16
- Trạng thái: Bản đề xuất; thành viên 190 và người duyệt chưa xác nhận
- Loại nhãn: bounding box trên ảnh đường phố
- Công cụ: CVAT, xuất COCO 1.0

## 1. Mục tiêu

Tạo nhãn 2D để huấn luyện mô hình phát hiện người và phương tiện thật trong ảnh đường phố. Bỏ sót người hoặc phương tiện có thể khiến hệ thống giao thông không nhận biết nguy hiểm; gán nhầm tượng, hình in hoặc phản chiếu làm tăng báo động giả. Bộ ảnh thực hành không đại diện cho mọi điều kiện giao thông và chưa đủ căn cứ để dùng trực tiếp cho hệ thống an toàn thực tế.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `person` | Một người thật có phần cơ thể nhìn thấy trong ảnh. | Người đi bộ, người lái hoặc người ngồi trên xe. | Tượng người, ma-nơ-canh, người trong hình in hoặc phản chiếu. |
| `truck` | Phương tiện cơ giới đường bộ được thiết kế chủ yếu để chở hàng. | Xe tải thùng, xe chở ô tô, xe bán tải có thùng hàng. | Ô tô chở khách `car`; xe buýt `other vehicle`. |
| `car` | Ô tô cơ giới thiết kế chủ yếu để chở người, không phải xe buýt. | Sedan, hatchback, SUV, xe van chở người. | Xe bán tải chở hàng `truck`; xe buýt `other vehicle`; xe đồ chơi. |
| `motor_bike` | Xe cơ giới hai bánh dùng để chở người. | Xe máy, xe tay ga, mô tô. | Xe đạp, xe ba bánh `other vehicle`; hình xe trên biển quảng cáo. |
| `other vehicle` | Phương tiện thật không thuộc `truck`, `car` hay `motor_bike`. | Xe buýt, tuk-tuk ba bánh, xe đạp, xích lô. | Xe tải `truck`; mô hình xe đồ chơi; phương tiện trong gương. |
| `ignore_object` | Vùng ảnh chứa vật mục tiêu thật quá nhỏ hoặc cụm vật mục tiêu không thể tách riêng đáng tin cậy. | Một người cao dưới 20 px; cụm người quá dày để tách từng người. | Tượng, phản chiếu và hình in được loại hẳn, không tạo vùng bỏ qua chỉ vì chúng tồn tại. |

Tên lớp được giữ đúng theo `individual/classes.md`, kể cả dấu cách trong `other vehicle`. Vật không thuộc năm lớp đối tượng thật thì không gán; không tự tạo lớp mới. Không dùng `ignore_object` thay cho quyết định lớp khi vật đã đủ lớn và tách được. Đối tượng không đủ bằng chứng là thật hay không, hoặc không xác định được phạm vi, ghi vấn đề cấp ảnh để người duyệt quyết định.

**Lưu ý tương thích:** bộ công cụ lab mặc định dùng `ignore_region` và schema yêu cầu tên lớp dạng `snake_case`; vì vậy `ignore_object` và `other vehicle` chưa chắc được script xử lý như dự kiến. Bộ lớp đã được người dùng xác nhận; vấn đề còn lại là tương thích kỹ thuật của taxonomy/CVAT và script, không phải duyệt lại các lớp. Không tự đổi tên lớp trong bản này.

### Thuộc tính

Áp dụng cho năm lớp đối tượng thật; `ignore_object` là vùng bỏ qua, không cần các thuộc tính đối tượng.

| Thuộc tính | Giá trị | Ý nghĩa và ngưỡng |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Ước lượng tỉ lệ diện tích hình chiếu của vật bị vật khác che trong phần nằm trong ảnh: `0` = 0%; `1` = lớn hơn 0% đến 30%; `2` = trên 30% đến 70%; `3` = trên 70%, nhưng vẫn nhận dạng chắc vật. Không tính phần nằm ngoài mép ảnh vào tỉ lệ che. Nếu không ước lượng được mức, chọn mức dự kiến và bật `can_review`. |
| `truncated` | `no`, `yes` | `yes` nếu vật tiếp tục ra ngoài mép ảnh; `no` nếu không. Che bởi vật khác không tự làm `truncated=yes`. |
| `can_review` | tick / không | Tick khi đã vẽ được box nhưng chưa chắc lớp, ranh giới hoặc thuộc tính; không tick khi quyết định chắc chắn. Nếu chưa thể vẽ box, ghi ảnh, vùng và lý do vào vấn đề cấp ảnh thay vì tạo box phỏng đoán. |

Đặt `occluded_level` bằng thuộc tính riêng trong CVAT; cờ che khuất mặc định của CVAT không thay thế thuộc tính này.

## 3. Quy tắc vẽ box

1. **R-01 — Một vật, một lớp:** gán riêng từng người và phương tiện thật đủ điều kiện; mỗi vật chỉ có một lớp. Dùng chức năng hình chữ nhật, các cạnh song song mép ảnh.
2. **R-02 — Phần nhìn thấy:** box nhỏ nhất bao toàn bộ phần nhìn thấy của cùng một vật, kể cả các phần rời nhau do bị che; không mở rộng để đoán phần ẩn. Không bao bóng đổ, người ngồi trên xe hay hàng hóa rời theo xe.
3. **R-03 — Che và cắt mép:** nếu vẫn nhận dạng chắc và tách được phần nhìn thấy, vẽ theo R-02, đặt mức che và `truncated` riêng. Vật bị che hoàn toàn không có box. Nếu không đủ bằng chứng để xác định vật hoặc ranh giới, ghi vấn đề cấp ảnh; không suy đoán.
4. **R-04 — Người và xe:** mỗi người có một box `person`; mỗi xe có một box theo loại xe. Ví dụ một người lái xe máy tạo hai box `person` và `motor_bike`; người chở thêm tạo thêm một box `person`. Các box có thể chồng nhau nhưng không cố ý gộp người vào box xe.
5. **R-05 — Xe chở xe và hàng:** xe chở hàng có box riêng; mỗi xe thật được chở có box riêng nếu đủ điều kiện. Box xe chở chỉ ôm phần nhìn thấy của bản thân xe và kết cấu cố định, không mở rộng để bao xe hoặc hàng rời. Hàng hóa thông thường không được gán nhãn.
6. **R-06 — Ngoài phạm vi:** không gán tượng, ma-nơ-canh, đồ chơi, hình in, hình trên màn hình hoặc vật chỉ xuất hiện qua phản chiếu; không tạo `ignore_object` cho riêng các vật đó. Không gán các bộ phận của xe chủ đang mang camera.
7. **R-07 — Vật nhỏ và đám đông:** đo chiều cao box phần nhìn thấy theo pixel ảnh gốc. Nếu cao dưới 20 px, không gán box lớp đối tượng; vẽ `ignore_object` ôm sát vật nhỏ đó hoặc cụm vật nhỏ liền kề. Nếu đám đông không thể tách từng người, vẽ vùng `ignore_object` ôm sát cụm không tách được; người đủ lớn và tách được ở ngoài vùng vẫn gán riêng. Không cố ý phủ vùng bỏ qua lên box đối tượng đã gán. Nếu không thể tránh chồng lấn, ghi vấn đề cấp ảnh để duyệt.
8. **R-08 — Phân loại phương tiện:** phân biệt xe chở người và chở hàng theo thiết kế nhìn thấy; xe bán tải có thùng hàng thuộc `truck`, xe buýt, tuk-tuk, xe đạp và xích lô thuộc `other vehicle`. Xe van chở hàng thuộc `truck`, van chở người thuộc `car`. Không xác định chắc công dụng thì chọn lớp dự kiến, tick `can_review` và ghi lý do.
9. **R-09 — Ảnh không có đối tượng:** giữ ảnh trong tập dữ liệu dù không có box đối tượng hoặc vùng bỏ qua; không tạo nhãn giả để lấp ảnh trống.
10. **R-10 — Vật được hóa trang hoặc cải biến:** nếu vẫn là người hoặc phương tiện thật, gán theo loại vật gốc, không theo hình dạng nhân vật hay vật được hóa trang. Box ôm phần nhìn thấy của vật cùng trang phục hoặc bộ phận trang trí gắn liền; không bao vật rời hoặc vật giả riêng biệt. Nếu không xác định chắc vật gốc, ghi vấn đề cấp ảnh hoặc tick `can_review` khi đã vẽ được box.

## 4. Ca khó

Các tình huống dưới đây do người dùng chỉ định. Quyết định là đề xuất áp dụng quy tắc theo mô tả, **chưa được kiểm tra trực tiếp trên ảnh**.

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_002.jpg` | Vật nằm trên hoặc trong vật lớn hơn: xe tải chở ô tô. | R-05 | Đề xuất box `truck` cho xe chở và box `car` riêng cho từng ô tô đủ điều kiện nhìn thấy. | Các xe là vật riêng, dù box có thể chồng nhau. |
| `img_011.jpg` | Vật được cải biến hoặc hóa trang. | R-10 | Nếu là người hoặc phương tiện thật, đề xuất gán lớp theo vật gốc; không gán lớp theo hình dạng hóa trang. | Hóa trang không tự biến vật thật thành vật giả hay thành lớp khác. |
| `img_019.jpg` | Vật được cải biến hoặc hóa trang. | R-10 | Nếu là người hoặc phương tiện thật, đề xuất gán lớp theo vật gốc và ôm phần trang trí gắn liền. | Phân biệt vật thật đã cải biến với vật giả riêng biệt. |
| `img_021.jpg` | Có quá nhiều đối tượng. | R-07 | Đề xuất gán riêng vật đủ lớn, tách được; dùng `ignore_object` cho vật quá nhỏ hoặc cụm không tách được. | Số lượng lớn không phải lý do bỏ qua toàn bộ ảnh. |
| `img_024.jpg` | Vật giả nằm trên vật thật. | R-05, R-06 | Đề xuất gán vật thật theo lớp của nó; không gán vật giả và không mở rộng box để bao vật giả rời. | Vật giả không trở thành đối tượng thật chỉ vì nằm trên vật thật. |
| `img_027.jpg` | Vật giả giống xe bán tải. | R-06 | Đề xuất không gán `truck` cho vật giả. | Vật giả không phải phương tiện thật. |
| `img_028.jpg` | Vật không thuộc các lớp mục tiêu nằm trên vật thật. | R-02, R-05 | Đề xuất gán riêng vật thật nếu thuộc lớp mục tiêu; không gán vật không thuộc lớp và không bao vật rời vào box. | Phân biệt đối tượng mục tiêu với vật ngoài phạm vi. |
| `img_030.jpg` | Có quá nhiều đối tượng. | R-07 | Đề xuất gán riêng vật đủ lớn, tách được; dùng `ignore_object` cho vật quá nhỏ hoặc cụm không tách được. | Giữ nhãn riêng khi khả thi, chỉ bỏ qua vùng thực sự không thể gán riêng. |
| `img_033.jpg` | Đối tượng dễ gây nhầm lẫn. | R-03, R-06 | Nếu là ma-nơ-canh hoặc vật giả, không gán `person`; nếu không xác định chắc vật thật hay giả, ghi vấn đề cấp ảnh. | Hình dạng giống người không đủ để kết luận là người thật. |
| `img_037.jpg` | Phương tiện dễ nhầm giữa bán tải có nắp thùng cao và xe van. | R-08 | Nếu xác định được thùng hàng của bán tải, đề xuất `truck`; nếu chưa chắc, tick `can_review`. | Phân lớp theo thiết kế chở hàng, không chỉ theo nắp thùng. |
| `img_038.jpg` | Đối tượng dễ gây nhầm lẫn với phương tiện thật. | R-03, R-06 | Nếu là xe đồ chơi, không gán lớp phương tiện; nếu chưa xác định được thật hay giả, ghi vấn đề cấp ảnh. | Hình dạng giống xe không đủ để kết luận là phương tiện thật. |

Nếu chưa chốt, tick `can_review` trên box hoặc ghi vấn đề cấp ảnh khi chưa vẽ được box. Dẫn Rule ID trong nhật ký QC; người học tự bổ sung và kiểm chứng ca người đi xe máy, người bị che và vật nhỏ trên ảnh gốc.

## 5. QC và phiên bản

- Nhóm chốt ca mơ hồ sau khi các bản độc lập đã được nộp; người duyệt nên khác người sửa, nhóm hai người đổi vai và ghi rõ.
- Trước calibration, cần đủ 100% lượt gán. Ghép box 1–1 với IoU ≥ 0,5; mục tiêu IoU trung bình ≥ 0,85 và cùng lớp ≥ 95% trên các cặp ghép. Không có cặp thì ghi N/A; box chưa ghép và vùng bỏ qua bất đồng được kiểm riêng.
- Khi kiểm từng box, ngưỡng IoU là 0,85 nếu box cao ≥ 50 px, 0,7 nếu thấp hơn. Script không thay thế người kiểm ảnh và không phát hiện chắc chắn vật mà mọi người cùng bỏ sót.
- Duyệt toàn lô trước bàn giao: Hold nếu thiếu bằng chứng, còn ca chưa giải quyết hoặc chỉ lấy mẫu demo; Rework nếu trên 2% ảnh lỗi hoặc lỗi lặp ≥ 3 và chiếm ≥ 50% tổng lỗi; Release khi đủ bằng chứng, không lỗi lặp và ≤ 2% ảnh lỗi. Với 40 ảnh, một ảnh lỗi là 2,5%.
- Mọi thay đổi làm đổi lựa chọn vật, lớp, box hoặc thuộc tính phải tăng phiên bản, ghi quyết định và kiểm lại phạm vi ảnh bị ảnh hưởng; thay đổi ngữ nghĩa theo quy trình lab cần rà cả lô.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên. | Chưa quy định đủ các ca biên. |
| v0.1 đề xuất | Sử dụng sáu lớp đã được người dùng xác nhận; bổ sung ngưỡng, quy tắc và các ca do người dùng chỉ định. | Làm rõ vật lồng nhau, hóa trang, vật giả, đám đông và các ca dễ nhầm; các quyết định trên ảnh chưa được kiểm chứng trực tiếp. |
