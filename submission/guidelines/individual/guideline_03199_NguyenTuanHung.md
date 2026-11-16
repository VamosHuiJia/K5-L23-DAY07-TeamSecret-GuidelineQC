# Guideline gán nhãn cá nhân — phát hiện người và phương tiện

- Phiên bản: v0.2 — đề xuất cá nhân, chưa phải guideline nhóm đã thống nhất
- Ngày: 2026-11-16
- Người viết: Nguyễn Tuấn Hùng
- Phạm vi: 40 ảnh `img_001.jpg`–`img_040.jpg`
- Loại nhãn: bounding box 2D; công cụ CVAT, xuất COCO 1.0

## 1. Mục tiêu

Tạo nhãn nhất quán để huấn luyện và đánh giá mô hình phát hiện người và phương tiện giao thông nhìn thấy trong ảnh đường phố. Bỏ sót một người/xe thật, gán nhầm lớp hoặc vẽ box sai hình học đều ảnh hưởng đến mô hình. Bản cá nhân này đề xuất thêm các lớp `bus`, `bicycle`, `tuk_tuk` và `cyclo` vì các loại xe đó xuất hiện trong bộ ảnh. Đây là đề xuất để nhóm xem xét; chỉ thêm vào taxonomy/CVAT sau khi nhóm thống nhất.

## 2. Lớp và thuộc tính

### 2.1 Lớp

Mỗi vật thể thật thuộc phạm vi chỉ có **một box và một lớp**. Người ngồi trên xe được gán riêng với xe nếu nhìn thấy; không gộp người và xe thành một box. Bốn lớp có ghi dưới đây là bổ sung của bản cá nhân, chưa có trong taxonomy hiện tại.

| Lớp             | Định nghĩa và ví dụ                                                                                                                                                                                                                            | Không thuộc lớp / dễ nhầm                                                                                                                                                                      |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `car`           | Ô tô chở người có thân xe kín, như sedan, hatchback, wagon, SUV và taxi (ví dụ taxi xanh ở `img_039`). Gán cả xe đang chạy, đỗ hoặc được chở trên xe chuyên dụng nếu đó là ô tô thật và đủ nhìn thấy.                                          | Không gán xe đồ chơi (`img_038`), tượng xe (`img_027`), hình trên billboard (`img_018`) hoặc xe chỉ thấy qua gương (`img_020`). Bán tải có thùng hở dùng `truck`.                              |
| `truck`         | Xe tải chở hàng có phần chở hàng gắn với xe: xe tải thùng, xe đầu kéo/chở ô tô (`img_002`) và bán tải thùng hở (`img_012`, `img_036`). Box ôm cabin và thùng xe gắn liền, không gồm hàng hóa rời.                                              | Không dùng cho van có khoang kín liền thân hoặc xe buýt. Hàng hóa chở trên xe không phải phần của box; không suy ra lớp chỉ từ kích thước lớn.                                                 |
| `van`           | Xe van/minivan có cabin và khoang hành khách/hàng hóa kín, liền thân, thường nhỏ hơn xe tải nặng (ví dụ van chở khách ở `img_005`). Gán theo cấu tạo thân xe, không theo việc xe đang chở người hay hàng.                                      | Xe buýt dùng `bus`; pickup có thùng hàng hở dùng `truck`. Nếu hình dáng không đủ rõ để phân biệt, bật `can_review`.                                                                            |
| `bus`           | Xe chở nhiều hành khách, có thân dài và khoang ghế/cửa lên xuống đặc trưng của xe buýt; gồm xe buýt đô thị và minibus thiết kế như xe buýt (ví dụ ở `img_032`).                                                                                | Không dùng cho van/minivan thông thường; phân biệt theo cấu tạo xe buýt, không chỉ theo số người đang ở trong xe.                                                                              |
| `motorcycle`    | Xe máy/mô tô có động cơ, thông thường hai bánh; gán người lái/ngồi sau thành `person` riêng nếu nhìn thấy. Với xe máy kéo sidecar (`img_022`), vẽ một box cho cả cụm xe máy và sidecar.                                                        | Không gồm xe đạp, xích lô hoặc tuk-tuk/auto-rickshaw. Không tách bánh, sidecar hoặc người thành xe riêng.                                                                                      |
| `bicycle`       | Xe đạp không có động cơ, thường có hai bánh, bàn đạp và người điều khiển/ngồi trên xe (ví dụ xe đạp chở hàng ở `img_003`). Gán xe và người thành hai box riêng nếu nhìn thấy.                                                                  | Xe máy có động cơ dùng `motorcycle`; xích lô chở khách dùng `cyclo`. Không gán riêng bánh xe hoặc hàng hóa chở trên xe.                                                                        |
| `tuk_tuk`       | Phương tiện chở khách/chở hàng có động cơ và ba bánh, gồm tuk-tuk ở `img_004` và auto-rickshaw ở `img_035`. Một box cho toàn bộ xe; người lái/hành khách nhìn thấy được gán `person` riêng.                                                    | Không dùng cho xe máy hai bánh, xe đạp hoặc xích lô đạp. Không gán mái che/rơ-moóc rời thành xe riêng.                                                                                         |
| `cyclo`         | Xe ba bánh đạp bằng sức người, có chỗ cho hành khách phía trước và người đạp phía sau (ví dụ ở `img_006`, `img_023`). Gán một box cho toàn bộ xe; người đạp và hành khách là `person` riêng nếu nhìn thấy.                                     | Không dùng cho tuk-tuk có động cơ (`tuk_tuk`) hoặc xe đạp thường (`bicycle`).                                                                                                                  |
| `person`        | Người thật nhìn thấy trong ảnh, gồm người đi bộ, tài xế, người đi xe và người bán hàng; ví dụ người bán hàng ở `img_003` hoặc người ngồi xe máy ở `img_026`. Gán một box riêng cho từng người đủ lớn, kể cả khi một phần bị xe/người khác che. | Không gồm tượng (`img_017`), mannequin (`img_033`), hình in, đồ chơi hoặc người chỉ xuất hiện trong phản chiếu (`img_020`). Người mặc mascot chỉ gán khi thấy rõ đó là người thật trong bộ đồ. |
| `ignore_region` | Box vùng bỏ qua cho vật không thuộc các lớp mục tiêu hoặc cụm người/xe mục tiêu quá nhỏ/khó tách. Ôm vừa vật/cụm cần bỏ qua, không bao trùm cả cảnh.                                                                                           | Đây là vùng loại khỏi đánh giá, không phải lớp đối tượng. Không dùng để che cả ảnh hay vật hợp lệ chỉ vì box khó vẽ; không để vùng phủ lên người/xe mục tiêu bên cạnh. Không cần thuộc tính.   |

**Đối tượng ngoài phạm vi:** nếu một vật không thuộc các lớp trên nhìn rõ và có thể bị hiểu nhầm là bị bỏ sót, vẽ `ignore_region` ôm riêng vật/cụm đó. Không cần vẽ vùng cho phong cảnh, nhà cửa, biển hiệu hay vật không liên quan. Khi vùng vật nhỏ nằm cạnh người/xe hợp lệ, bảo đảm `ignore_region` không phủ quá nửa diện tích box hợp lệ đó.

### 2.2 Thuộc tính

Gán đủ ba thuộc tính cho từng box `car`, `truck`, `van`, `bus`, `motorcycle`, `bicycle`, `tuk_tuk`, `cyclo`, `person`. Không gán thuộc tính cho `ignore_region`.

| Thuộc tính       | Mức     | Quy tắc chọn mức                                                                                                                                                                                                          |
| ---------------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `occluded_level` | `0`     | Không bị vật khác che, hoặc phần bị che ước tính dưới 10% diện tích nhìn thấy kỳ vọng.                                                                                                                                    |
|                  | `1`     | Bị che từ 10% đến dưới 30%.                                                                                                                                                                                               |
|                  | `2`     | Bị che từ 30% đến 60%, tính cả hai đầu mút.                                                                                                                                                                               |
|                  | `3`     | Bị che trên 60%, nhưng vẫn xác định chắc chắn được lớp và vị trí của vật. Nếu không thể xác định chắc chắn là một đối tượng thuộc lớp nào thì không vẽ đối tượng đó; cân nhắc `ignore_region` cho cụm mơ hồ.              |
| `truncated`      | `no`    | Vật nằm hoàn toàn trong khung ảnh, dù có thể bị vật khác che.                                                                                                                                                             |
|                  | `yes`   | Phần vật thể đi ra ngoài một trong bốn cạnh ảnh. Chỉ bị che bởi vật khác không phải là truncated.                                                                                                                         |
| `can_review`     | bỏ chọn | Lớp, ranh giới và thuộc tính đều rõ theo quy tắc.                                                                                                                                                                         |
|                  | chọn    | Còn bất đồng hợp lý sau khi đã áp dụng quy tắc, ví dụ khó phân biệt van với xe tải hoặc không chắc vật có phải xe thật hay vật trưng bày. Ghi rõ ảnh/vùng/lý do trong ghi chú QC; không dùng thay cho việc đọc guideline. |

**Cách ước tính che khuất:** so phần bị che với toàn bộ hình dạng vật thể có thể suy ra từ phần còn nhìn thấy và ngữ cảnh. Chỉ tính che bởi người/vật khác; không tính phần nằm ngoài ảnh vào `occluded_level` (phần đó được ghi bằng `truncated=yes`). Nếu hai loại cùng xảy ra, gán cả mức che khuất và cờ cắt mép.

## 3. Quy tắc vẽ box

### 3.1 Chọn đối tượng và xác định box

1. Quét ảnh theo thứ tự trái sang phải, trên xuống dưới; kiểm tra lại các vùng đông người/xe và mép ảnh để tránh bỏ sót.
2. Box chữ nhật ôm sát **phần nhìn thấy** của một vật thể; không bao gồm bóng, khoảng trống, hàng hóa rời hoặc vật thể bên cạnh. Không suy diễn phần bị che để kéo box qua vùng không nhìn thấy.
3. Với xe có nhiều phần gắn liền (cab + thùng xe tải, đầu kéo + rơ-moóc chở xe, xe máy + sidecar), vẽ một box cho toàn bộ cụm phương tiện đang hoạt động như một xe. Những ô tô được chở trên xe chở ô tô vẫn là các xe riêng, mỗi chiếc một box nếu đủ kích thước nhìn thấy và tách được.
4. Người và phương tiện là hai đối tượng riêng. Box `person` ôm phần cơ thể nhìn thấy, bao gồm mũ bảo hiểm/quần áo đang mặc nhưng không bao gồm xe. Người lái bị kính xe che vẫn là `person` nếu thấy và xác định được người.
5. Đồ vật chở trên xe không nhập vào box xe; nếu là một vật ngoài taxonomy thì không cần box riêng, trừ khi nó làm vùng đối tượng không thể chấm rõ.
6. Box không được có chiều cao dưới 20 px. Đối tượng mục tiêu rõ nhưng cao dưới 20 px được gộp vào một `ignore_region` vừa đủ bao cụm vật nhỏ; không tạo box mục tiêu nhỏ. Nếu chỉ là một đối tượng nhỏ tách biệt, vùng ignore ôm riêng đối tượng đó.
7. Với vật cao từ 20 px trở lên nhưng bị che nhiều, vẫn vẽ nếu nhận diện chắc chắn; áp dụng `occluded_level`. Nếu không chắc nó có tồn tại/là lớp nào, không đoán; ghi `can_review` hoặc vùng ignore phù hợp.
8. Box bị cắt bởi biên ảnh chỉ bao phần nằm trong ảnh và đặt `truncated=yes`. Không đặt `truncated=yes` cho vật bị che hoàn toàn bởi xe/người khác.
9. Các phương tiện đỗ vẫn được gán nhãn như phương tiện đang chạy. Trạng thái đỗ, xe cũ, trời tối, mưa, ảnh mờ hoặc chở hàng không làm thay đổi lớp nếu vẫn nhận diện được.

### 3.2 Hình ảnh không phải đối tượng thật

- **Không gán** người/xe xuất hiện trong gương, cửa kính phản chiếu, ảnh chụp, áp phích, tranh, màn hình hoặc biển quảng cáo.
- **Không gán** tượng người/ngựa/xe, mannequin, mô hình thu nhỏ và xe đồ chơi; đây không phải người hoặc phương tiện giao thông thật trong cảnh. Gán `ignore_region` nếu vật đó có thể bị coi là mục tiêu bị bỏ sót.
- Với xe thật đang làm vật trưng bày (ví dụ xe trong bảo tàng), chỉ gán nếu là xe nguyên kích thước có thể nhận diện như phương tiện thật; không gán khung vỏ trang trí/xe trưng bày đặt chồng trên nóc xe khác như một xe đang tham gia giao thông. Nếu khó phân biệt, đánh dấu `can_review`.
- Đạo cụ hoặc mascot trong sự kiện: chỉ gán `person` khi thấy rõ người thật đang mặc đồ hóa trang; không gán phần đầu mascot như một người riêng. Xe diễu hành có hình dáng/khung gầm của ô tô thật và bánh xe thật có thể gán `car`; nếu chỉ là mô hình/đạo cụ không vận hành, dùng `ignore_region`.

### 3.3 Đám đông và phương tiện ngoài phạm vi

- Từng người/xe đủ lớn và tách được phải có box riêng, kể cả trong đám đông.
- Không vẽ một box `person` bao nhiều người. Với đám đông xa, nơi từng người thấp dưới 20 px hoặc không thể tách ranh giới, dùng một `ignore_region` cho cụm đó.
- Dùng `bus`, `bicycle`, `tuk_tuk` và `cyclo` cho đúng loại phương tiện theo bảng lớp; không ép chúng sang `car`, `truck`, `van` hay `motorcycle`. Phương tiện khác chưa có lớp thì dùng `ignore_region` nếu cần loại khỏi đánh giá.
- Không đặt `ignore_region` đè lên box mục tiêu hợp lệ. Nếu phần ngoài taxonomy chồng lấn, vẽ box mục tiêu trước và giới hạn vùng ignore vào phần ngoài taxonomy; khi không thể tránh che lấn đáng kể, đánh dấu `can_review` và ghi vùng.

### 3.4 Kiểm tra trước khi lưu

- Mỗi box có đúng một lớp, ôm đúng một vật/cụm cần ignore, không bao trùm nền.
- Mọi đối tượng mục tiêu cao từ 20 px trở lên đều được xem xét; mọi vật thấp hơn ngưỡng được xử lý theo quy tắc ignore.
- `occluded_level` phản ánh che khuất, `truncated` chỉ phản ánh chạm/cắt mép ảnh.
- Không có box cho phản chiếu, hình in, tượng hoặc đồ chơi.
- Ảnh không có đối tượng mục tiêu vẫn phải có trong COCO với danh sách annotation rỗng.

## 4. Ca khó tiêu biểu

Các ví dụ dưới đây minh họa quy tắc, không khẳng định số lượng box thực tế. Các lớp bổ sung là đề xuất cá nhân, cần nhóm thống nhất trước khi đưa vào taxonomy/CVAT.

| Ảnh                  | Tình huống cần chú ý                                           | Quyết định gán nhãn và lý do                                                                                                                                 |
| -------------------- | -------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `img_002`            | Xe chở ô tô nhiều tầng, các xe che lẫn nhau.                   | Gán `truck` cho xe chuyên chở và `car` riêng cho từng ô tô thật đủ ngưỡng; gán mức che khuất riêng cho từng box.                                             |
| `img_004`, `img_035` | Tuk-tuk và auto-rickshaw ba bánh.                              | Dùng `tuk_tuk` cho mỗi xe hoàn chỉnh; người lái/hành khách nhìn thấy được gán `person` riêng, không dùng `motorcycle`.                                       |
| `img_006`            | Dòng giao thông có xe đạp, xích lô, xe máy và người.           | Phân biệt lần lượt `bicycle`, `cyclo`, `motorcycle`; gán người thành box riêng. Dùng `ignore_region` cho cụm nào không tách được hoặc thấp hơn ngưỡng 20 px. |
| `img_031`            | Phố đêm có ô tô, xe buýt, xe máy và phản chiếu trên mặt đường. | Gán xe thật theo lớp `car`, `bus`, `motorcycle`; không gán phản chiếu trên đường.                                                                            |
| `img_020`            | Cảnh xe và cầu vồng chỉ xuất hiện trong gương chiếu hậu.       | Không gán người/xe trong phản chiếu; chúng không phải đối tượng trực tiếp trong cảnh.                                                                        |

## 5. QC và phiên bản

### 5.1 Tự kiểm / kiểm chéo

- Kiểm tra đủ 40 ảnh theo đúng tên file; ảnh không có annotation vẫn phải có trong COCO.
- Kiểm tra thủ công các ảnh có rủi ro cao: `img_002`, `img_004`–`img_006`, `img_020`, `img_031`, `img_035`.
- Soát các box ở mép ảnh, vật nhỏ, vùng chồng lấn và mọi `ignore_region`; xác nhận vùng ignore không che đối tượng hợp lệ.
- Ngưỡng QC của bài được giữ nguyên theo taxonomy: chiều cao tối thiểu 20 px; ngưỡng IoU box cao từ 50 px là 0.85, box thấp hơn 50 px là 0.70; cặp ghép IAA dùng IoU tối thiểu 0.50; IAA mục tiêu IoU trung bình ≥ 0.85 và cùng lớp ≥ 95%. Báo số lỗi theo ảnh với mẫu số rõ ràng; ngưỡng lỗi ảnh tối đa là 2%.
- Lỗi lặp trên cùng loại tình huống cần xem lại rule, kể cả khi tỷ lệ lỗi tổng còn thấp. Không sửa số liệu QC hoặc ghi nhận một lần kiểm chưa thực hiện.
- Mọi box `can_review` cần có ảnh, vùng và câu hỏi cụ thể để reviewer có thể quyết định; sau khi chốt phải cập nhật nhật ký quyết định theo yêu cầu của nhóm.

### 5.2 Nhật ký thay đổi

| Phiên bản | Thay đổi                                                                                    | Lý do / ca tham chiếu                                                                                                                       |
| --------- | ------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| v0.2      | Đề xuất thêm `bus`, `bicycle`, `tuk_tuk`, `cyclo`; rút phần ca khó xuống 5 ví dụ tiêu biểu. | Các phương tiện này xuất hiện trong bộ ảnh nhưng chưa có lớp trong taxonomy hiện tại. Cần nhóm thống nhất trước khi thêm vào taxonomy/CVAT. |

Đây là đề xuất cá nhân trước khi họp nhóm. Nếu nhóm chốt quy tắc khác hoặc phát hiện ca chưa giải quyết được, ghi quyết định và lý do trong bản guideline nhóm / nhật ký calibration; không sửa artifact của người khác.
