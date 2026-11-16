# Guideline gán nhãn nhóm — v1.1

- Phiên bản: `v1.1`
- Ngày: 2026-11-17
- Phạm vi: 40 ảnh đường phố `img_001.jpg`–`img_040.jpg`
- Loại nhãn: bounding box 2D
- Công cụ / định dạng: CVAT, COCO 1.0
- Taxonomy tương ứng: `submission/taxonomy.json` (`v1.0`)

- Loại thay đổi: `editorial` — giữ nguyên ý nghĩa quy tắc v1.0.
- Trạng thái: bản hiệu đính phục vụ calibration, chờ nhóm xác nhận từng ca và kiểm lại nhãn.

> Bản này được biên tập với sự hỗ trợ của AI từ guideline v1.0 và báo cáo IAA. Tại thời điểm soạn, bảng calibration có 321 ca trên 25 ảnh và chưa có dòng quyết định nào điền đầy đủ. Các mã quy tắc dưới đây dùng để dẫn chiếu; nội dung tài liệu không thay cho việc người gán và reviewer đối chiếu ảnh gốc, chốt ca hoặc xác nhận sửa xong.

Taxonomy bàn giao tiếp tục là `submission/taxonomy.json` phiên bản `v1.0`: giữ nguyên 10 lớp, các thuộc tính và toàn bộ mục `rules`. Nhãn độc lập và guideline v1.0 được giữ nguyên làm bằng chứng trước calibration. Nếu buổi họp quyết định đổi ý nghĩa quy tắc, nhóm phải cập nhật nội dung v1.1 và `guideline_change.json` sang `semantic`, ghi phạm vi tác động và rà lại cả lô theo quy trình của lab.

## 1. Mục tiêu

Tạo nhãn nhất quán cho mô hình phát hiện người và phương tiện thật nhìn thấy trong ảnh đường phố. Mỗi box phải xác định đúng một vật, đúng lớp và ôm sát phần nhìn thấy. Không suy đoán vật bị che hoàn toàn, không gán hình tượng hoặc hình ảnh phản chiếu như vật thật. Bộ dữ liệu thực hành dùng để học quy trình gán nhãn và QC, không tự nó chứng minh mô hình an toàn để triển khai giao thông thực tế.

## 2. Lớp và thuộc tính

### 2.1 Lớp — R-14

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

Mã dẫn chiếu cho phần thuộc tính:

- **R-11 — Mức che khuất:** áp dụng đúng các mức `occluded_level` và cách ước lượng trong bảng bên dưới.
- **R-12 — Cắt mép ảnh:** áp dụng `truncated` độc lập với che khuất; vật bị che bên trong ảnh không tự động là vật bị cắt mép.
- **R-13 — Cần review:** cờ `can_review` thể hiện ca chưa chắc. Người gán và reviewer phải xử lý ca theo quy tắc trước khi xác nhận nhãn cuối; không coi giá trị mặc định là bằng chứng đã xem lại.

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

1. **R-01 — Một box cho một vật:** vẽ hình chữ nhật nhỏ nhất ôm phần nhìn thấy của đối tượng. Không bao khoảng trống, bóng đổ, mặt đường hoặc vật thể bên cạnh; không kéo box vào phần bị che mà không nhìn thấy.
2. **R-02 — Người và xe:** vẽ riêng người và phương tiện. Người lái/ngồi sau xe máy, xe đạp, xích lô hoặc tuk-tuk là `person` riêng nếu nhìn thấy, nhận dạng được và box cao ít nhất 20 px. Người nhìn thấy rõ qua cửa kính xe kín vẫn được gán; không suy đoán người chỉ có thể đang ở trong xe.
3. **R-03 — Các phần xe:** box xe gồm các bộ phận gắn cố định, như gương chiếu hậu và sidecar gắn với xe máy. Một xe máy có sidecar là một box `motorcycle`, không tách sidecar thành xe khác. Với xe đầu kéo/chở ô tô, vẽ một box `truck` cho xe chở và box `car` riêng cho từng ô tô thật được chở nếu đủ điều kiện.
4. **R-04 — Hàng hóa:** không gộp hàng hóa rời, bao tải hoặc vật chở quá khổ vào box xe. Box chỉ bao phương tiện và kết cấu cố định của nó; vật ngoài taxonomy không tự động thành đối tượng gán nhãn.
5. **R-05 — Che khuất và cắt mép:** vật còn nhận dạng chắc thì vẽ phần nhìn thấy và gán thuộc tính che/cắt độc lập. Vật bị che hoàn toàn không có box. Với vật bị cắt mép, box dừng ở biên ảnh và đặt `truncated=yes`.
6. **R-06 — Kích thước tối thiểu:** box đối tượng có chiều cao từ 20 px trở lên. Đối tượng mục tiêu nhỏ hơn 20 px hoặc cụm mục tiêu không tách được dùng một `ignore_region` ôm sát vùng đó. Không để vùng ignore che box đối tượng hợp lệ khác.
7. **R-07 — Phương tiện thật và vật trang trí:** không gán tượng, ma-nơ-canh, xe đồ chơi/mô hình, đạo cụ/xe mô phỏng chỉ dùng để trưng bày, hình in, hình trên màn hình hoặc người/xe chỉ xuất hiện trong phản chiếu. Không tạo `ignore_region` riêng cho các vật này. Phương tiện thật nguyên kích thước vẫn được gán dù đang đỗ, dùng trong nghi lễ hoặc có thể không hoạt động; không kết luận là đạo cụ chỉ vì xe được trang trí, không có người lái hoặc trông như không chạy được. Nếu chưa phân biệt chắc xe thật với đạo cụ, bật `can_review` khi đã xác định được box; nếu chưa xác định được vật có thuộc lớp hay không, ghi vấn đề cấp ảnh để reviewer chốt.
8. **R-08 — Phương tiện không thuộc lớp:** các loại phương tiện trong phạm vi phải dùng đúng lớp ở Mục 2; không gộp xe đạp, xích lô, tuk-tuk hoặc xe buýt vào `motorcycle` hay `car`. Phương tiện khác không thuộc taxonomy không được ép vào lớp gần giống.
9. **R-09 — Xe chủ:** không gán bộ phận của chính xe gắn camera (ví dụ nắp ca-pô hoặc mép xe xuất hiện ở tiền cảnh).
10. **R-10 — Ảnh trống:** giữ ảnh trong COCO dù không có đối tượng; không tạo box giả để lấp ảnh.

## 4. Ca khó

Bảng dưới kế thừa nguyên ý nghĩa các quy tắc ca khó đã ghi trong v1.0, bổ sung mã dẫn chiếu cho calibration. Bảng không khẳng định các ca IAA liên quan đã được người review chốt xong.

| Ảnh | Tình huống | Rule ID | Quy tắc kế thừa v1.0 | Lý do |
|---|---|---|---|---|
| `img_002` | Xe chở ô tô nhiều tầng. | R-03, R-06 | Một box `truck` cho xe chở; một box `car` cho mỗi ô tô thật đủ nhìn thấy và cao từ 20 px. | Xe chở và các xe được chở là các vật thể riêng. |
| `img_004`, `img_035` | Tuk-tuk/auto-rickshaw; một xe chở hàng cồng kềnh. | R-02, R-04, R-14 | Gán `tuk_tuk`; gán người nhìn thấy riêng. Không kéo box xe lên phần bao tải/hàng hóa rời. | Nhận diện theo động cơ và cấu tạo ba bánh; phân biệt xe với tải hàng. |
| `img_011` | Tuk-tuk trang trí dùng trong nghi lễ, nghi ngờ không hoạt động. | R-07, R-13 | Không loại chỉ vì xe được trang trí hoặc có thể không chạy được. Nếu là phương tiện thật nguyên kích thước, gán `tuk_tuk`; nếu reviewer xác nhận đây là đạo cụ trưng bày thay vì phương tiện thật thì không gán. Khi chưa phân biệt được, đánh dấu `can_review` hoặc ghi vấn đề cấp ảnh trước khi chốt nhãn cuối. | Khả năng hoạt động không thể xác định chắc chỉ từ ảnh; cần phân biệt bản thân phương tiện với đạo cụ mô phỏng. |
| `img_006`, `img_023`, `img_034` | Xe đạp, xích lô, xe máy và người đi đường. | R-02, R-06, R-14 | Dùng `bicycle`, `cyclo`, `motorcycle` tương ứng; mỗi người là một box `person` riêng nếu đủ điều kiện. | Không ép phương tiện không động cơ thành xe máy; tách người khỏi phương tiện. |
| `img_009`, `img_031`, `img_032`, `img_037` | Xe buýt/minibus cạnh xe con, van hoặc bán tải. | R-13, R-14 | Xe có cấu tạo xe buýt dùng `bus`; van thân kín liền khối dùng `van`; pickup có thùng hàng dùng `truck`. Khi cấu tạo bị che và không đủ căn cứ, bật `can_review`. | Không phân lớp chỉ theo kích thước hoặc số người trên xe. |
| `img_017`, `img_018`, `img_020`, `img_027`, `img_033`, `img_038` | Tượng, ảnh quảng cáo, phản chiếu, ma-nơ-canh và xe đồ chơi. | R-07 | Không gán các hình thể này thành `person` hoặc phương tiện; chỉ gán vật thật nhìn trực tiếp ở ngoài chúng nếu có. | Hình dáng giống người/xe không đủ để xem là đối tượng thật. |

### 4.1. Các nhóm bất đồng cần đối chiếu

Kết quả trước calibration: đủ **80/80 lượt gán** trên 40 ảnh, có **252 cặp box ghép**, IoU trung bình **0,906**, đồng thuận lớp **98,8%**. Đây là số liệu từ `../qc/iaa_summary.md`, không phải số liệu sau sửa nhãn. Cặp `199|215` có IoU trung bình **0,847**, còn dưới ngưỡng 0,85. Mức cùng `occluded_level` tổng thể là **54,4%**, cùng `truncated` là **89,3%**; các bất đồng thuộc tính cần được xem dù đồng thuận lớp tổng thể đạt ngưỡng.

| Nhóm ca trong báo cáo | Số ca | Quy tắc cần dẫn chiếu | Hướng đối chiếu của người review |
|---|---:|---|---|
| `bo_sot_thua` | 109 | R-01, R-02, R-06, R-07, R-08, R-09 | Xem vùng được nêu trên ảnh gốc để xác định vật đủ điều kiện gán hay phải loại; ghi quyết định giữ, thêm hoặc bỏ box và lý do. Chỉ một người vẽ box chưa đủ chứng minh người đó đúng hay sai. |
| `iou_thap` | 53 | R-01, R-03, R-04, R-05 | Đối chiếu ranh giới phần nhìn thấy, các bộ phận xe, hàng hóa và mép ảnh. Mã ca chỉ báo độ lệch, không tự chọn một trong hai box làm đáp án. |
| `sai_thuoc_tinh` | 125 | R-11, R-12 | Xem vật bị che/cắt như thế nào rồi đối chiếu ngưỡng đã có; ghi giá trị cuối và lý do. Không lấy giá trị mặc định hoặc giá trị của một thành viên làm chuẩn chung. |
| `sai_lop` | 3 | R-14 | Đọc định nghĩa lớp và nhìn cấu tạo vật trong ảnh. Các ca đang được gắn cờ gồm C073 (`img_007.jpg`: truck/car), C138 (`img_016.jpg`: truck/car), C222 (`img_031.jpg`: bus/truck); tài liệu này chưa chọn lớp cuối cho các ca đó. |
| `co_can_review` | 12 | R-13 | Ghi rõ điểm chưa chắc, người chốt và kết quả kiểm lại; chỉ xác nhận hết cờ sau khi ca thực sự được xử lý. |
| `bat_dong_ignore` | 19 | R-06 | Đối chiếu vật nhỏ, cụm dày và vùng ignore có phủ vật hợp lệ khác hay không; chốt phạm vi vùng theo ảnh gốc. |

### 4.2. Ghi quyết định từng ca

Mỗi `case_id` trong `../qc/calibration_candidates.csv` cần một dòng tương ứng trong `../qc/calibration_decisions.csv`. Một ảnh có thể có nhiều ca; không gộp 321 ca thành 25 quyết định chỉ theo số ảnh.

Dòng quyết định ghi đủ: `case_id`, `image`, `quyet_dinh`, `quy_tac_ghi_vao_guideline`, `phien_ban_guideline`, `nguoi_chot`, `phieu_sua`, `kiem_lai`, `pham_vi`. Dẫn mã `R-xx` phù hợp ở trên, phiên bản `v1.1` và mã thành viên ba chữ số. `phieu_sua` phải chỉ ra vị trí/bản sửa; `kiem_lai` ghi đúng trạng thái và người đã kiểm, không tự nhận đã sửa hoặc đã duyệt khi việc đó chưa thực hiện.

- `pham_vi=anh`: quyết định áp dụng cho ảnh được ghi trong dòng, theo quy tắc hiện hành.
- `pham_vi=lo`: thay đổi quy tắc có tác động cả lô; phải mô tả tác động và kiểm lại 40 ảnh theo yêu cầu lab.
- Giữ nguyên kết quả IAA trước calibration. Không sửa tay các artifact do script sinh để làm số liệu đạt ngưỡng.

## 5. QC và lịch sử phiên bản

### 5.1 QC — R-15

- Kiểm tra đủ 40 ảnh và đúng tên file; xem lại lớp, box, thuộc tính, box bị cắt mép và các `ignore_region`.
- Ngưỡng lab giữ nguyên theo `taxonomy.json`: chiều cao tối thiểu 20 px; ngưỡng IoU box cao từ 50 px là 0.85, thấp hơn 50 px là 0.70; ngưỡng ghép IAA là 0.50; IAA mục tiêu IoU trung bình ít nhất 0.85 và đồng thuận lớp ít nhất 95%; tỷ lệ ảnh lỗi tối đa 2%.
- Ghi số liệu dưới dạng tử số/mẫu số. IAA chỉ mô tả các box ghép được; box bỏ sót/thừa và vùng ignore bất đồng phải được xem riêng. Đồng thuận cao không chứng minh hai người không cùng sai.
- Duyệt toàn lô trước Release. Thiếu bằng chứng hoặc chưa chốt quy tắc: Hold. Vượt ngưỡng lỗi hoặc có lỗi lặp: Rework. Chỉ Release khi reviewer xác nhận đủ bằng chứng và đạt các ngưỡng lab.
- Người gán đánh dấu `can_review` hoặc ghi vấn đề cấp ảnh; nhóm chốt ca mơ hồ sau khi hoàn tất gán độc lập. Mọi thay đổi lớp, box hoặc thuộc tính sau đó phải truy được tới quyết định và phạm vi kiểm lại.

### 5.2. Bằng chứng trước khi bàn giao

Bảng quyết định hiện chưa được điền đầy đủ, nên bản hiệu đính này chưa xác nhận calibration hoàn tất. Sau khi nhóm chốt và sửa nhãn, cần có COCO sau sửa đúng phiên bản, `labels_calibrated.json`, `labels_calibrated.csv`, `batch_manifest.json`, bằng chứng vòng QC và `QC_REPORT.md`.

Duyệt QC vẫn áp dụng cho toàn bộ 40 ảnh. Lưu ảnh, phiếu và kết quả theo từng vòng; khi đổi nhãn cuối thì mở vòng mới để giữ truy vết. Nhãn còn `can_review` hoặc thiếu bằng chứng tiếp tục được xử lý theo quy tắc Hold của lab, không suy ra Release chỉ từ việc IAA tổng thể đạt ngưỡng.

### 5.3 Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do / bằng chứng |
|---|---|---|
| `v0` | Bản khởi đầu gồm `car`, `truck`, `van`, `motorcycle`, `person`, `ignore_region`; định nghĩa và thuộc tính còn định tính, thiếu quy tắc ca biên. | Bản starter của bài lab. |
| `v1.0` | Hợp nhất bốn bản cá nhân; thêm `bus`, `bicycle`, `tuk_tuk`, `cyclo`; chuẩn hóa tên lớp theo `snake_case`; định lượng mức occlusion; thống nhất box phần nhìn thấy, phân biệt người/xe, hàng hóa, vật giả, vật nhỏ, xe chủ và xe trang trí. | Các phương tiện bổ sung xuất hiện trong ảnh; các bản cá nhân khác nhau về cách xử lý tuk-tuk, xe buýt, xe đạp/xích lô, người trong xe kín và hàng chở. Làm rõ không loại xe thật chỉ vì nghi không hoạt động; chỉ loại đạo cụ/mô hình, với ca `img_011` cần reviewer phân biệt. |
| `v1.1` (editorial, chờ xác nhận calibration) | Gắn mã R-01 đến R-15 cho các quy tắc kế thừa v1.0; bổ sung hướng dẫn dẫn chiếu sáu nhóm bất đồng, phạm vi và bằng chứng kiểm lại. Giữ nguyên lớp, thuộc tính, ngưỡng và ý nghĩa quy tắc. | Báo cáo IAA có 321 ca trên 25 ảnh; chưa có dòng quyết định nào điền đầy đủ tại thời điểm soạn. Không coi các ca này là đã được xử lý. |
