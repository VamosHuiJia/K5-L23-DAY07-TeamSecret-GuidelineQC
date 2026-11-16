# Guideline gán nhãn nhóm — v1.0

- Phiên bản: `v1.0`
- Ngày: 2026-11-16
- Phạm vi: 40 ảnh đường phố `img_001.jpg`–`img_040.jpg`
- Loại nhãn: bounding box 2D
- Công cụ / định dạng: CVAT, COCO 1.0
- Taxonomy tương ứng: `submission/taxonomy.json` (`v1.0`)

## 1. Mục tiêu

Tạo nhãn nhất quán cho mô hình phát hiện người và phương tiện thật nhìn thấy trong ảnh đường phố. Mỗi box phải xác định đúng một vật, đúng lớp và ôm sát phần nhìn thấy. Không suy đoán vật bị che hoàn toàn, không gán hình tượng hoặc hình ảnh phản chiếu như vật thật. Bộ dữ liệu thực hành dùng để học quy trình gán nhãn và QC, không tự nó chứng minh mô hình an toàn để triển khai giao thông thực tế.

## 2. Lớp và thuộc tính

### 2.1 Lớp

Một vật thể chỉ có một box và một lớp. Người lái, hành khách và phương tiện được gán thành các đối tượng riêng nếu nhìn thấy đủ rõ. `ignore_region` là vùng bỏ qua, không phải một loại vật thể.

| Lớp | Định nghĩa | Ví dụ | Không thuộc lớp / dễ nhầm |
|---|---|---|---|
| `person` | Người thật có phần cơ thể nhìn thấy và nhận dạng được, gồm người đi bộ, người bán hàng, tài xế và hành khách. | Người đi bộ; người ngồi trên xe máy; người nhìn thấy qua cửa kính. | Tượng, ma-nơ-canh, người trong hình in hoặc phản chiếu. |
| `car` | Ô tô thân xe kiểu chở người, như sedan, hatchback, wagon hoặc SUV; taxi vẫn thuộc lớp này. | Ô tô đi trên đường hoặc ô tô thật được chở trên xe chuyên dụng. | Bán tải có thùng hàng dùng `truck`; xe van dạng hộp dùng `van`; xe buýt dùng `bus`. |
| `truck` | Xe cơ giới thiết kế chở hàng, gồm xe tải, xe đầu kéo chở hàng/ô tô và bán tải. | Xe tải thùng; xe bán tải; xe chở ô tô. | Xe van thân liền dùng `van`. Hàng hóa rời không phải phần của xe. |
| `van` | Xe thân kín liền khối, dạng hộp, dùng chở người hoặc hàng; không có thùng hàng tách biệt như bán tải. | Minivan hoặc van giao hàng cỡ nhỏ. | Xe buýt có cấu tạo chở khách kiểu xe buýt dùng `bus`; bán tải dùng `truck`. |
| `bus` | Xe buýt hoặc minibus được thiết kế chở nhiều hành khách, nhận biết qua cấu tạo thân xe và khoang hành khách, không chỉ dựa vào kích thước. | Xe buýt đô thị; minibus kiểu xe buýt. | Van/minivan thông thường dùng `van`. |
| `motorcycle` | Xe máy/mô tô có động cơ hai bánh. Xe máy kéo sidecar được tính là một phương tiện. | Xe máy, xe tay ga, mô tô; xe máy có sidecar. | Xe đạp dùng `bicycle`; tuk-tuk ba bánh có động cơ dùng `tuk_tuk`. |
| `bicycle` | Xe đạp không có động cơ, được đẩy bằng bàn đạp; người đạp là đối tượng `person` riêng. | Xe đạp thường hoặc xe đạp chở hàng. | Xe máy có động cơ dùng `motorcycle`; xích lô chở khách dùng `cyclo`. |
| `tuk_tuk` | Xe ba bánh có động cơ, dạng tuk-tuk/auto-rickshaw, dùng chở người hoặc hàng. | Tuk-tuk; auto-rickshaw. | Xe máy hai bánh dùng `motorcycle`; xích lô đạp dùng `cyclo`. |
| `cyclo` | Xe ba bánh chở khách vận hành bằng bàn đạp, có người đạp và chỗ ngồi cho khách. | Xích lô đạp. | Tuk-tuk/auto-rickshaw có động cơ dùng `tuk_tuk`; xe đạp thường dùng `bicycle`. |
| `ignore_region` | Vùng ôm sát một vật mục tiêu quá nhỏ hoặc một cụm mục tiêu quá dày/không thể tách riêng đáng tin cậy; bỏ sót từng vật bên trong vùng không tính là lỗi. | Cụm người hoặc phương tiện ở xa, có box cao dưới 20 px. | Không dùng để che đối tượng mục tiêu lớn, rõ; không dùng cho tượng, hình in hoặc phản chiếu. |

**Quyết định lớp v1.0:** dùng tên lớp theo dạng `snake_case` để khớp schema và CVAT; chọn `motorcycle` thay cho `motor_bike`, `ignore_region` thay cho `ignore_object`, và không dùng lớp tổng quát `other_vehicle`. Bổ sung riêng `bus`, `bicycle`, `tuk_tuk`, `cyclo` vì đây là các loại phương tiện xuất hiện trong bộ ảnh và có thể nhận diện theo đặc điểm hình thái.

### 2.2 Thuộc tính

Gán đủ thuộc tính cho từng box đối tượng (`person`, `car`, `truck`, `van`, `bus`, `motorcycle`, `bicycle`, `tuk_tuk`, `cyclo`). `ignore_region` không cần thuộc tính.

| Thuộc tính | Giá trị | Quy tắc |
|---|---|---|
| `occluded_level` | `0` | Không bị vật khác che (0%). |
|  | `1` | Bị che trên 0% đến 30%. |
|  | `2` | Bị che trên 30% đến 70%. |
|  | `3` | Bị che trên 70%, nhưng vẫn nhận dạng chắc chắn được vật và lớp. |
| `truncated` | `no` | Vật nằm hoàn toàn trong khung ảnh. Bị che bởi vật khác không phải là truncated. |
|  | `yes` | Vật kéo dài qua ít nhất một cạnh ảnh; box chỉ nằm trong khung ảnh. |
| `can_review` | `false` | Lớp, vị trí box và thuộc tính rõ theo quy tắc. |
|  | `true` | Đã vẽ được box nhưng còn phân vân về lớp, ranh giới hoặc thuộc tính; ghi ảnh, vùng và lý do để reviewer xử lý. |

Ước lượng phần trăm bị che trên hình dạng vật thể dự kiến nằm trong ảnh; chỉ tính phần bị vật khác che, không tính phần ngoài khung ảnh. Nếu không nhận dạng chắc vật hoặc lớp thì không gán một box đoán mò; ghi vấn đề cấp ảnh. Tối, mưa hoặc nhòe không tự động làm tăng mức occlusion.

## 3. Quy tắc vẽ box

1. **Một box cho một vật:** vẽ hình chữ nhật nhỏ nhất ôm phần nhìn thấy của đối tượng. Không bao khoảng trống, bóng đổ, mặt đường hoặc vật thể bên cạnh; không kéo box vào phần bị che mà không nhìn thấy.
2. **Người và xe:** vẽ riêng người và phương tiện. Người lái/ngồi sau xe máy, xe đạp, xích lô hoặc tuk-tuk là `person` riêng nếu nhìn thấy, nhận dạng được và box cao ít nhất 20 px. Người nhìn thấy rõ qua cửa kính xe kín vẫn được gán; không suy đoán người chỉ có thể đang ở trong xe.
3. **Các phần xe:** box xe gồm các bộ phận gắn cố định, như gương chiếu hậu và sidecar gắn với xe máy. Một xe máy có sidecar là một box `motorcycle`, không tách sidecar thành xe khác. Với xe đầu kéo/chở ô tô, vẽ một box `truck` cho xe chở và box `car` riêng cho từng ô tô thật được chở nếu đủ điều kiện.
4. **Hàng hóa:** không gộp hàng hóa rời, bao tải hoặc vật chở quá khổ vào box xe. Box chỉ bao phương tiện và kết cấu cố định của nó; vật ngoài taxonomy không tự động thành đối tượng gán nhãn.
5. **Che khuất và cắt mép:** vật còn nhận dạng chắc thì vẽ phần nhìn thấy và gán thuộc tính che/cắt độc lập. Vật bị che hoàn toàn không có box. Với vật bị cắt mép, box dừng ở biên ảnh và đặt `truncated=yes`.
6. **Kích thước tối thiểu:** box đối tượng có chiều cao từ 20 px trở lên. Đối tượng mục tiêu nhỏ hơn 20 px hoặc cụm mục tiêu không tách được dùng một `ignore_region` ôm sát vùng đó. Không để vùng ignore che box đối tượng hợp lệ khác.
7. **Phương tiện thật và vật trang trí:** không gán tượng, ma-nơ-canh, xe đồ chơi/mô hình, đạo cụ/xe mô phỏng chỉ dùng để trưng bày, hình in, hình trên màn hình hoặc người/xe chỉ xuất hiện trong phản chiếu. Không tạo `ignore_region` riêng cho các vật này. Phương tiện thật nguyên kích thước vẫn được gán dù đang đỗ, dùng trong nghi lễ hoặc có thể không hoạt động; không kết luận là đạo cụ chỉ vì xe được trang trí, không có người lái hoặc trông như không chạy được. Nếu chưa phân biệt chắc xe thật với đạo cụ, bật `can_review` khi đã xác định được box; nếu chưa xác định được vật có thuộc lớp hay không, ghi vấn đề cấp ảnh để reviewer chốt.
8. **Phương tiện không thuộc lớp:** các loại phương tiện trong phạm vi phải dùng đúng lớp ở Mục 2; không gộp xe đạp, xích lô, tuk-tuk hoặc xe buýt vào `motorcycle` hay `car`. Phương tiện khác không thuộc taxonomy không được ép vào lớp gần giống.
9. **Xe chủ:** không gán bộ phận của chính xe gắn camera (ví dụ nắp ca-pô hoặc mép xe xuất hiện ở tiền cảnh).
10. **Ảnh trống:** giữ ảnh trong COCO dù không có đối tượng; không tạo box giả để lấp ảnh.

## 4. Ca khó

Các quyết định sau làm rõ các điểm bất đồng hoặc chưa được v0 quy định; áp dụng cùng cách cho các ảnh tương tự.

| Ảnh | Tình huống | Quyết định v1.0 | Lý do |
|---|---|---|---|
| `img_002` | Xe chở ô tô nhiều tầng. | Một box `truck` cho xe chở; một box `car` cho mỗi ô tô thật đủ nhìn thấy và cao từ 20 px. | Xe chở và các xe được chở là các vật thể riêng. |
| `img_004`, `img_035` | Tuk-tuk/auto-rickshaw; một xe chở hàng cồng kềnh. | Gán `tuk_tuk`; gán người nhìn thấy riêng. Không kéo box xe lên phần bao tải/hàng hóa rời. | Nhận diện theo động cơ và cấu tạo ba bánh; phân biệt xe với tải hàng. |
| `img_011` | Tuk-tuk trang trí dùng trong nghi lễ, nghi ngờ không hoạt động. | Không loại chỉ vì xe được trang trí hoặc có thể không chạy được. Nếu là phương tiện thật nguyên kích thước, gán `tuk_tuk`; nếu reviewer xác nhận đây là đạo cụ trưng bày thay vì phương tiện thật thì không gán. Khi chưa phân biệt được, đánh dấu `can_review` hoặc ghi vấn đề cấp ảnh trước khi chốt nhãn cuối. | Khả năng hoạt động không thể xác định chắc chỉ từ ảnh; cần phân biệt bản thân phương tiện với đạo cụ mô phỏng. |
| `img_006`, `img_023`, `img_034` | Xe đạp, xích lô, xe máy và người đi đường. | Dùng `bicycle`, `cyclo`, `motorcycle` tương ứng; mỗi người là một box `person` riêng nếu đủ điều kiện. | Không ép phương tiện không động cơ thành xe máy; tách người khỏi phương tiện. |
| `img_009`, `img_031`, `img_032`, `img_037` | Xe buýt/minibus cạnh xe con, van hoặc bán tải. | Xe có cấu tạo xe buýt dùng `bus`; van thân kín liền khối dùng `van`; pickup có thùng hàng dùng `truck`. Khi cấu tạo bị che và không đủ căn cứ, bật `can_review`. | Không phân lớp chỉ theo kích thước hoặc số người trên xe. |
| `img_017`, `img_018`, `img_020`, `img_027`, `img_033`, `img_038` | Tượng, ảnh quảng cáo, phản chiếu, ma-nơ-canh và xe đồ chơi. | Không gán các hình thể này thành `person` hoặc phương tiện; chỉ gán vật thật nhìn trực tiếp ở ngoài chúng nếu có. | Hình dáng giống người/xe không đủ để xem là đối tượng thật. |

## 5. QC và lịch sử phiên bản

### 5.1 QC

- Kiểm tra đủ 40 ảnh và đúng tên file; xem lại lớp, box, thuộc tính, box bị cắt mép và các `ignore_region`.
- Ngưỡng lab giữ nguyên theo `taxonomy.json`: chiều cao tối thiểu 20 px; ngưỡng IoU box cao từ 50 px là 0.85, thấp hơn 50 px là 0.70; ngưỡng ghép IAA là 0.50; IAA mục tiêu IoU trung bình ít nhất 0.85 và đồng thuận lớp ít nhất 95%; tỷ lệ ảnh lỗi tối đa 2%.
- Ghi số liệu dưới dạng tử số/mẫu số. IAA chỉ mô tả các box ghép được; box bỏ sót/thừa và vùng ignore bất đồng phải được xem riêng. Đồng thuận cao không chứng minh hai người không cùng sai.
- Duyệt toàn lô trước Release. Thiếu bằng chứng hoặc chưa chốt quy tắc: Hold. Vượt ngưỡng lỗi hoặc có lỗi lặp: Rework. Chỉ Release khi reviewer xác nhận đủ bằng chứng và đạt các ngưỡng lab.
- Người gán đánh dấu `can_review` hoặc ghi vấn đề cấp ảnh; nhóm chốt ca mơ hồ sau khi hoàn tất gán độc lập. Mọi thay đổi lớp, box hoặc thuộc tính sau đó phải truy được tới quyết định và phạm vi kiểm lại.

### 5.2 Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do / bằng chứng |
|---|---|---|
| `v0` | Bản khởi đầu gồm `car`, `truck`, `van`, `motorcycle`, `person`, `ignore_region`; định nghĩa và thuộc tính còn định tính, thiếu quy tắc ca biên. | Bản starter của bài lab. |
| `v1.0` | Hợp nhất bốn bản cá nhân; thêm `bus`, `bicycle`, `tuk_tuk`, `cyclo`; chuẩn hóa tên lớp theo `snake_case`; định lượng mức occlusion; thống nhất box phần nhìn thấy, phân biệt người/xe, hàng hóa, vật giả, vật nhỏ, xe chủ và xe trang trí. | Các phương tiện bổ sung xuất hiện trong ảnh; các bản cá nhân khác nhau về cách xử lý tuk-tuk, xe buýt, xe đạp/xích lô, người trong xe kín và hàng chở. Làm rõ không loại xe thật chỉ vì nghi không hoạt động; chỉ loại đạo cụ/mô hình, với ca `img_011` cần reviewer phân biệt. |
