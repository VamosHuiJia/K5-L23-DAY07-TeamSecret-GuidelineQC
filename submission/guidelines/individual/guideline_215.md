# Guideline gán nhãn — v0.2 cá nhân — 215

- Phiên bản: v0.2 — bản nháp cá nhân trước khi họp nhóm
- Ngày: 16/11/2026
- Người viết / duyệt: 215 / chưa duyệt, chưa họp thống nhất nhóm
- Công cụ dự kiến: CVAT, bounding box hình chữ nhật, xuất COCO 1.0

> Bản này được soạn từ guideline v0 và nội dung đề bài. Các tình huống có ID ảnh bên dưới lấy từ mô tả trong đề, chưa phải kết quả em tự khảo sát đủ 40 ảnh. Em cần mở ảnh, kiểm tra và sửa lại các ý kiến này trước khi nộp bản cá nhân. Các lớp và quy tắc thêm vào đây là đề xuất để thảo luận, chưa thay thế taxonomy chung.

Em tham khảo thêm [hướng dẫn trợ giảng — Pha 1](https://site-ivory-mu-57.vercel.app/#pha1). Cách em triển khai là đọc v0, xem từng ảnh rồi hỏi: nếu làm đúng theo v0 thì em sẽ gán gì, chỗ nào chưa có câu trả lời, và người khác có thể hiểu khác em ở đâu? Với ca chưa chắc, em ghi rõ “chưa chốt, cần họp” thay vì coi ý kiến ban đầu là quy tắc nhóm.

## 1. Mục tiêu

Theo em, bộ nhãn dùng để huấn luyện mô hình phát hiện người và phương tiện thật trong ảnh đường phố 2D. Nhãn cần cho biết có vật gì và vật đó nằm ở đâu trên ảnh. Em không suy đoán các thông tin không nhìn thấy như người bên trong xe kín, hãng xe hoặc mục đích sử dụng xe.

Nếu bỏ sót người hoặc phương tiện thật, mô hình có thể học thiếu các tình huống giao thông cần phát hiện. Ngược lại, gán tượng, hình in hoặc phản chiếu thành vật thật sẽ tạo ví dụ sai. Box quá rộng hoặc suy đoán thêm phần không thấy cũng khiến vị trí mục tiêu không nhất quán. Vì vậy em muốn làm rõ cả điều kiện chọn vật, chọn lớp và phạm vi box.

Em ưu tiên quy tắc có thể áp dụng giống nhau giữa các thành viên. Ví dụ, nếu chỉ ghi “box ôm sát vật” thì vẫn chưa rõ có cần bao cả phần bị che hay không. Trong bản này, em chọn ôm phần nhìn thấy và ghi mức che bằng thuộc tính riêng.

Bộ 40 ảnh dùng cho thực hành xây dựng guideline và QC. Kết quả của bài chưa đủ để kết luận mô hình có thể sử dụng an toàn trong giao thông thực tế.

## 2. Lớp và thuộc tính

### Lớp

Mỗi vật thật được gán một lớp. Một người ngồi trên xe và chiếc xe là hai vật riêng. `ignore_region` là vùng bỏ qua, không phải một loại phương tiện.

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con, sedan, hatchback hoặc SUV, có thân xe chủ yếu dành cho chở người. | Ô tô con đỗ bên đường. | Không gán xe bán tải có thùng hàng riêng vào lớp này. |
| `truck` | Xe có cabin và phần thùng hoặc sàn chở hàng riêng; gồm cả xe bán tải. | Xe tải, xe chở ô tô. | Không coi mọi xe kích thước lớn đều là `truck`; xe buýt thuộc lớp khác. |
| `van` | Xe có thân kín liền khối, dạng hộp, dùng chở người hoặc hàng; không có thùng hàng tách riêng như bán tải. | Xe van nhỏ. | Bán tải lắp nắp thùng cao vẫn đề xuất là `truck` nếu nhìn rõ kết cấu bán tải. |
| `motorcycle` | Xe máy hoặc mô tô hai bánh có động cơ. | Xe máy đang chạy hoặc đỗ. | Xe đạp không thuộc lớp này; tuk-tuk không ép vào lớp này. |
| `person` | Người thật có phần cơ thể nhìn thấy và đủ cơ sở nhận dạng. | Người đi bộ hoặc người ngồi trên xe máy. | Tượng người, ma-nơ-canh, ảnh chân dung không phải `person`. |
| `bus` — đề xuất thêm | Xe chở khách kiểu xe buýt, có thân xe dài hoặc cao và dãy cửa sổ khoang hành khách, nhận dạng từ tổ hợp đặc điểm nhìn thấy. | Xe buýt trong tình huống đề mô tả ở `img_009.jpg`. | Không phân loại chỉ dựa vào kích thước; xe van chở khách chưa chắc là bus. |
| `tuk_tuk` — đề xuất thêm | Xe ba bánh có động cơ, có vị trí điều khiển và khoang hoặc sàn chở người/hàng phía sau kiểu tuk-tuk. | Tình huống đề mô tả ở `img_004.jpg`, `img_035.jpg`. | Xe đạp ba bánh, xích lô không động cơ không thuộc lớp này. |
| `ignore_region` | Vùng có vật thuộc phạm vi nhưng quá nhỏ hoặc quá dày để tách từng vật đáng tin cậy. | Cụm người nhỏ ở xa, các vật cao dưới 20 px. | Không dùng để che một vật lớn, rõ nhưng em chưa biết chọn lớp nào. |

Em đề xuất thêm `bus` và `tuk_tuk` vì đây là loại vật mà mô hình cần phân biệt. Việc thêm lớp phải được nhóm thống nhất ở Pha 2 rồi cập nhật taxonomy và nhãn CVAT, không tự thêm khi đang gán độc lập.

Với vật ngoài các lớp đã chốt, như xe đạp hoặc xích lô không động cơ, em đề xuất không gán box và không tự đổi thành lớp gần giống. Nếu nhóm muốn đưa chúng vào phạm vi, cần bổ sung lớp trước khi khóa v1.0. Vật ngoài phạm vi không tự động trở thành vùng ignore.

Khi dễ nhầm `car` / `van` / `truck` / `bus`, em ưu tiên kết cấu thân xe nhìn thấy, không chọn lớp theo màu, hãng hay biển số. Bán tải có cabin và thùng hàng riêng nghiêng về `truck`, kể cả khi lắp nắp thùng. Thân kín liền khối dạng hộp nghiêng về `van`; dấu hiệu xe buýt phải đến từ cấu trúc khoang hành khách và thân xe. Nếu ảnh che hoặc mờ đến mức không phân biệt được, em ghi “chưa chốt, cần họp”, không đoán kết cấu bị khuất.

Em dùng hai hướng xử lý riêng:

- **Loại theo quy tắc:** đã biết chắc vật là tượng, hình in, phản chiếu hoặc thuộc loại ngoài phạm vi. Em ghi lý do loại; không gán một lớp gần giống và không dùng ignore để thay cho quyết định loại.
- **Chưa chốt, cần họp:** chưa đủ căn cứ phân biệt lớp, chưa rõ đó là xe hoàn chỉnh hay thân vỏ, hoặc nhóm chưa chốt việc thêm lớp. Em giữ câu hỏi để họp Pha 2. Khi đã sang Pha 3, ca phát sinh được đánh dấu review hoặc ghi vấn đề cấp ảnh theo guideline v1.0.

### Thuộc tính

**`occluded_level`: mức bị vật khác che khuất**

Em đề xuất ước lượng phần diện tích hình chiếu của vật bị che, không lấy diện tích hình chữ nhật bbox làm diện tích vật. Phần nằm ngoài mép ảnh được xử lý bằng `truncated`, không cộng vào mức che. Nếu không đủ cơ sở ước lượng thì ghi là ca cần review.

Cụ thể, em xét diện tích vật dự kiến nằm trong ảnh nếu bỏ vật che ra. Tỷ lệ che là phần diện tích bị vật khác che chia cho diện tích dự kiến đó. Đây là cách ước lượng thuộc tính; box vẫn chỉ bao phần nhìn thấy theo R-01. Em không vẽ thêm phần suy đoán để làm box đầy đủ.

| Giá trị | Ngưỡng đề xuất |
|---|---|
| `0` | Không bị vật khác che: 0%. |
| `1` | Bị che trên 0% đến 30%. |
| `2` | Bị che trên 30% đến 70%. |
| `3` | Bị che trên 70%, nhưng vẫn xác định chắc được vật và lớp. |

Các mức này vẫn là ước lượng bằng mắt. Em muốn nhóm chọn thêm ảnh minh họa gần ranh giới 30% và 70% để hạn chế mỗi người hiểu một cách. Mờ ảnh hoặc tối ảnh không tự động là mức che `3`.

Ví dụ minh họa ngưỡng, không phải số đo từ bộ ảnh: che khoảng 25% là mức `1`, đúng 30% vẫn là `1`, 50% là `2`, đúng 70% vẫn là `2`, 80% là `3` nếu còn nhận dạng chắc được vật. Em viết rõ các mốc bằng nhau để tránh hai người gán khác mức ở ranh giới.

**`truncated`: vật bị cắt ở mép ảnh**

- `yes`: thấy rõ vật còn kéo dài ra ngoài ít nhất một mép ảnh.
- `no`: vật không bị cắt, kể cả khi bị vật khác che bên trong ảnh.
- Một vật có thể vừa bị che, vừa bị cắt; điền hai thuộc tính độc lập.

**`can_review`: cần xem lại**

- `false`: em đủ chắc về lớp, box và thuộc tính theo quy tắc.
- `true`: đã vẽ được box cho vật trong phạm vi nhưng còn phân vân về lớp, biên box hoặc mức thuộc tính.
- Nếu không xác định được cả vị trí box, em ghi vấn đề cấp ảnh kèm vùng và lý do để tổng hợp sau khi khóa nhãn độc lập. Không tạo một box tùy ý chỉ để đánh dấu.
- Cờ này phục vụ quy trình QC, không phải đặc điểm mà mô hình cần dự đoán. Nhãn cuối chỉ bỏ cờ sau khi ca đã được giải quyết.

Với `ignore_region`, em đề xuất thống nhất các thuộc tính ở giá trị mặc định `0`, `no`, `false`; ý nghĩa bỏ qua do lớp vùng này quyết định.

## 3. Quy tắc vẽ box

Các mã `R-xx` dưới đây là mã tạm trong bản cá nhân. Nhóm có thể đổi số khi hợp nhất, nhưng cần giữ được nội dung quyết định.

Với mỗi vật nghi là đối tượng cần gán, em kiểm theo thứ tự: (1) vật thật hay hình/tượng/phản chiếu; (2) có thuộc xe chủ không; (3) thuộc lớp nào trong phạm vi đã chốt; (4) phần nhìn thấy có đạt chiều cao 20 px không; (5) box bao những phần nào; (6) mức che, cắt mép và cờ review là gì. Một vật đủ điều kiện vẫn phải gán dù đang đỗ, bị che một phần hoặc nhìn qua kính. Việc đang chuyển động không phải điều kiện bắt buộc.

- **R-01 — Ôm sát phần nhìn thấy:** dùng hình chữ nhật nhỏ nhất bao các phần nhìn thấy của cùng một vật. Không kéo box ra để tưởng tượng phần bị che. Nếu vật có nhiều phần lộ ra ở hai phía vật che, box bao cả các phần đó và có thể chứa vật che ở giữa. Không tách một vật thành nhiều box.
- **R-02 — Người và xe máy:** vẽ box `person` cho phần cơ thể nhìn thấy và box `motorcycle` cho xe, không dùng một box chung cho cả hai. Người lái và người ngồi sau là hai người riêng nếu đủ nhận dạng. Nếu chỉ thấy xe thì không suy ra box người. Cần tự thử cùng quy tắc trên ít nhất hai ảnh trước khi chốt bản cá nhân.
- **R-03 — Xe chở xe:** gán xe tải chở hàng là một `truck`; mỗi ô tô thật được chở là một `car` riêng nếu đủ điều kiện kích thước và nhận dạng. Box truck bao cabin và kết cấu xe chở, không cố bao toàn bộ hàng hóa nhô ra ngoài. Nếu chỉ thấy thân vỏ hoặc phụ tùng, chưa đủ xác định đó là một ô tô riêng thì đưa ra review.
- **R-04 — Vật không thật và phản chiếu:** không gán tượng, ma-nơ-canh, đồ chơi, người/xe trong hình in, trên màn hình hoặc qua phản chiếu. Nếu thấy cả vật thật và ảnh phản chiếu của nó thì chỉ gán vật thật. Vật thật nhìn xuyên qua kính vẫn có thể gán; cần phân biệt với phản chiếu trên kính.
- **R-05 — Che khuất và cắt mép:** vật còn nhận dạng được thì gán phần nhìn thấy theo R-01 và điền thuộc tính. Khi cắt mép, box dừng ở biên ảnh, không vượt ra ngoài kích thước ảnh. Nếu chỉ còn mảnh quá nhỏ và không chắc là vật gì thì ghi vấn đề để review.
- **R-06 — Vật nhỏ và đám đông:** đo chiều cao box phần nhìn thấy. Dưới 20 px thì không gán như một vật riêng; dùng `ignore_region` ôm gọn phần cần bỏ qua. Từ 20 px trở lên thì gán riêng nếu nhận dạng được. Đám đông chỉ dùng ignore ở phần không thể tách đáng tin cậy, tránh phủ lên những người hoặc xe lớn, rõ bên cạnh.
- **R-07 — Xe chủ:** không gán phần xe gắn camera đang xuất hiện trong ảnh như nắp capo hoặc gương xe chủ. Người và phương tiện khác ngoài đường vẫn xét theo quy tắc bình thường.
- **R-08 — Vật gắn kèm:** box người gồm cơ thể, quần áo và đồ đội sát người như mũ bảo hiểm; không cố bao cả hành lý rời, bóng đổ hoặc nền. Box xe gồm thân xe, bánh, gương và kết cấu gắn cố định; không bao bóng đổ. Hàng chở quá khổ được phân biệt với kết cấu xe như ở R-03.
- **R-09 — Không đủ chắc:** ảnh mờ không có nghĩa là phải bỏ qua mọi vật. Vật còn đủ cơ sở nhận dạng thì gán và đánh dấu review ở phần chưa chắc; trường hợp không có cơ sở chọn lớp thì ghi vấn đề cấp ảnh. Không dùng `ignore_region` chỉ để tăng đồng thuận.
- **R-10 — Lớp và phạm vi:** chọn lớp theo định nghĩa ở Mục 2. `bus` và `tuk_tuk` là đề xuất bổ sung của bản cá nhân, cần họp chốt trước khi gán. Xe đạp hoặc xích lô không động cơ được đề xuất ngoài phạm vi; không đổi thành `motorcycle`. Nếu không rõ đó là loại ngoài phạm vi hay một loại đang được gán, xử lý như ca chưa chắc theo R-09.

## 4. Ca khó

Các quyết định trong bảng là ý kiến đề xuất từ tình huống được đề bài mô tả. Em chưa ghi số box thực tế hay coi các ca này là đã kiểm tra xong. Trong cột Quyết định, “loại theo quy tắc” thể hiện cách xử lý nếu xác nhận đúng tình huống mô tả; “chưa chốt, cần họp” là việc em muốn đưa ra Pha 2.

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_002.jpg` | Xe chở ô tô. | R-03 | Gán xe chở và từng ô tô thật đủ điều kiện thành các vật riêng. | Cần mở ảnh kiểm tra số xe nhìn thấy, phần che và biên từng box; không lấy số box trong ví dụ làm kết quả của mình. |
| `img_004.jpg`, `img_035.jpg` | Tuk-tuk. | R-08, R-10 | Chưa chốt, cần họp: em đề xuất thêm `tuk_tuk`, không ép vào `motorcycle`; không bao hàng hóa nhô ra chỉ vì hàng được chở trên xe. | Cần kiểm tra cấu tạo xe và ranh giới hàng/kết cấu. Nếu nhóm không thêm lớp thì phải chốt cách xử lý trước Pha 3. |
| `img_009.jpg`, `img_037.jpg` | Xe buýt. | R-10 | Chưa chốt, cần họp: em đề xuất thêm `bus`. | Cần đối chiếu hình dáng thực tế và ranh giới với van, không dựa riêng vào kích thước. |
| `img_011.jpg`, `img_017.jpg`, `img_027.jpg` | Tượng người hoặc tượng xe. | R-04 | Loại theo quy tắc: không gán tượng thành người/xe thật. | Quyết định dựa vào vật có thật trong phạm vi mô hình, không chỉ dựa vào hình dáng giống người/xe. |
| `img_018.jpg` | Chân dung trên quảng cáo. | R-04 | Loại theo quy tắc: không gán người trong hình in. | Người thật đứng gần biển, nếu có và đủ điều kiện, vẫn phải xét riêng. |
| `img_020.jpg` | Phản chiếu qua gương. | R-04, R-07 | Loại hình phản chiếu theo quy tắc; kiểm tra gương có thuộc xe chủ không. | Cần phân biệt ảnh phản chiếu với vật thật nhìn trực tiếp. |
| `img_033.jpg`, `img_038.jpg` | Ma-nơ-canh hoặc đồ chơi trong tủ kính. | R-04 | Loại theo quy tắc: không gán vật giả thành `person` hoặc xe thật. | Kính không phải lý do loại; lý do là bản chất vật. Cần phân biệt người/xe thật nhìn qua kính. |
| `img_037.jpg` | Bán tải có nắp thùng cao. | R-09, R-10 | Chưa chốt, cần họp: em nghiêng về `truck` nếu xác định được kết cấu bán tải. | Nắp thùng kín có thể làm nhầm với `van`; cần thống nhất dấu hiệu phân biệt trên ảnh. |
| `img_006.jpg`, `img_029.jpg`, `img_039.jpg` | Xe đạp hoặc xích lô không động cơ. | R-10 | Chưa chốt, cần họp về phạm vi: em đề xuất loại khỏi taxonomy của bài, không gán thành xe máy. | Nếu nhóm muốn giữ các vật này thì phải thêm lớp trước khi khóa v1.0. |

Em còn cần tự chọn một ảnh có người đi xe máy và một ảnh khác để thử lại R-02. Chưa có kết quả khảo sát thì em chưa điền ID ảnh, số người hay số box cho ca đó.

Trước buổi họp, em cần tự bổ sung từ ảnh gốc: một ca người đi xe máy để ghi rõ số box và phạm vi từng box; một ca che hoặc cắt mép để giải thích thuộc tính; một ca vật nhỏ/đám đông để giải thích có dùng ignore hay không. Sau đó em thử các quy tắc đó trên ảnh khác. Những ví dụ bổ sung này phải do em kiểm tra, không lấy ý kiến đồng thuận của nhóm điền ngược thành nhận định ban đầu.

## 5. QC và phiên bản

### 5.1. Ngưỡng áp dụng

Em giữ nguyên các ngưỡng trong `submission/taxonomy.json`:

| Nội dung | Ngưỡng |
|---|---|
| Chiều cao box vật để gán riêng | Từ 20 px. |
| Ghép box giữa hai người | Ghép 1–1, IoU từ 0.5, chưa phân biệt lớp ở bước ghép. |
| Đối chiếu vị trí box | Box cao từ 50 px: IoU từ 0.85; thấp hơn 50 px: từ 0.7. |
| Mức IAA mục tiêu trước calibration | IoU trung bình từ 0.85; đồng thuận lớp từ 95%, trên các cặp ghép được. |
| Tỷ lệ ảnh lỗi để xét Release | Không quá 2%, kèm đủ điều kiện và bằng chứng QC khác của lab. |

Nhóm có bốn người nên mỗi ảnh cần hai người gán độc lập theo phân công. Mọi người phải dùng cùng guideline v1.0 đã khóa. Đồng thuận cao chưa chứng minh nhãn đúng vì hai người có thể cùng bỏ sót hoặc cùng hiểu sai.

### 5.2. Cách em tự kiểm và xử lý ca chưa rõ

Trước khi gửi nhãn, em cần kiểm đúng ảnh được giao, lớp, thuộc tính, box trong biên ảnh và các vật có thể bỏ sót. Script kiểm cấu trúc và tính toán; việc box có đúng vật, đúng lớp, ôm sát hay không vẫn cần người nhìn ảnh.

Khi gán độc lập, em giữ cờ hoặc ghi vấn đề để review, không xem nhãn của bạn để sửa theo. Sau khi cả nhóm khóa bản độc lập, nhóm mới họp calibration. Mỗi ca cần có quyết định, mã quy tắc, người chốt và dấu vết kiểm lại. Nếu thay đổi cách chọn vật, lớp hoặc box thì cần rà lại phạm vi bị ảnh hưởng; quy tắc đổi nghĩa áp dụng toàn lô thì phải kiểm cả lô.

QC cuối phải duyệt đủ 40 ảnh, ưu tiên người review khác người sửa. Một ảnh lỗi trên 40 ảnh là 2.5%, đã vượt ngưỡng 2%. Thiếu bằng chứng thì Hold; lỗi vượt ngưỡng hoặc lỗi lặp thì Rework; đủ bằng chứng và đạt điều kiện mới Release. Em chưa ghi số liệu hay quyết định bàn giao vì chưa thực hiện các bước này.

Trang hướng dẫn trợ giảng minh họa duyệt mẫu 20 ảnh. Tuy nhiên, [GUIDE.md trong repo hiện tại](../../../GUIDE.md) quy định duyệt toàn lô theo vòng; tùy chọn lấy mẫu chỉ để demo và giữ Hold khi bàn giao. Em dùng yêu cầu 40/40 của repo cho phần QC trong bản này. Quy ước tên file có họ tên trên trang cũng khác repo; bản cá nhân hiện giữ tên đã được người làm bài lựa chọn, còn tên file COCO dùng với script cần được đối chiếu riêng trước Pha 3.

### 5.3. Chuyển từ bản cá nhân sang bản nhóm

Sau khi tự kiểm ảnh và hoàn thiện bản này, em gửi/nộp theo kênh đã được coach xác nhận, giữ tính độc lập đến khi cả nhóm hoàn thành Pha 1. Mỗi thành viên tự commit phần của mình theo hướng dẫn của lớp. Sau buổi họp, em giữ nguyên bản cá nhân để nhìn lại các điểm khác nhau; quyết định chung được viết vào `submission/guidelines/guideline_v1.0_agreed.md` và đồng bộ với `submission/taxonomy.json`.

Ba nội dung em muốn đưa vào buổi họp là: có thêm `bus` và `tuk_tuk` không; phân biệt bán tải nắp thùng cao với van bằng dấu hiệu nào; và phạm vi box xe chở xe/hàng nhô ra được tính đến đâu. Nhóm cần giải quyết các điểm này trước khi khóa v1.0 và bắt đầu gán độc lập. Những thay đổi sau calibration viết vào v1.1, không sửa lại ý kiến ban đầu để làm các bản giống nhau.

### 5.4. Lịch sử phiên bản

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu do giảng viên cung cấp. | Định nghĩa và quy tắc còn ngắn, có nhiều cách hiểu. |
| v0.1 — bản nháp 215 | Đề xuất ngưỡng che, quy tắc box, phân biệt vật thật/giả, bổ sung bus/tuk-tuk và các ca cần kiểm tra. | Chuẩn bị ý kiến cá nhân để tự đối chiếu 40 ảnh trước khi họp nhóm. Chưa phải bản đã khảo sát hoặc guideline nhóm đã khóa. |
| v0.2 — làm rõ bản nháp trước khi họp | Đưa tiêu đề/cột về mẫu, bổ sung thứ tự xét vật, cách tính mức che, phân biệt ca loại và ca chưa chốt, cùng hướng chuyển sang v1.0. | Tham khảo hướng dẫn trợ giảng; làm rõ các câu hỏi ở `img_002.jpg`, `img_004.jpg`, `img_035.jpg`, `img_037.jpg`. Chưa có thêm kết quả khảo sát ảnh. |

Sau khi tự xem ảnh, em cần sửa lại những quyết định không phù hợp, bổ sung ví dụ mình đã kiểm chứng và giữ bản cá nhân để đối chiếu với kết quả họp nhóm.
