# Guideline gán nhãn — v0.1_draft — Thành viên 204

> Bản guideline cá nhân của thành viên 204, xây dựng dựa trên khung `TEMPLATE_guideline.md`, slide bài giảng Day 07 và kết quả khảo sát toàn bộ 40 ảnh thực tế trong `data/images/`.
> Mọi quy tắc trong tài liệu này nhằm chuyển hóa các mô tả định tính mơ hồ của bản v0 thành các quy tắc định lượng, nhất quán, có thể kiểm chứng độc lập.

- Phiên bản: v0.1_draft (Khảo sát cá nhân từ v0)
- Ngày: 2026-11-16
- Người viết: Thành viên 204

---

## 1. Mục tiêu

- **Mô hình mục tiêu**: Huấn luyện mô hình phát hiện vật thể 2D (2D Bounding Box Object Detection - ví dụ kiến trúc YOLOv8/v10, Faster R-CNN, Deformable DETR, Grounding DINO) ứng dụng trong hệ thống hỗ trợ lái xe nâng cao (ADAS), hệ thống lái tự động (Autonomous Driving / Robotaxi) và hệ thống giám sát giao thông thông minh (ITS) tại môi trường đô thị hỗn hợp châu Á (Việt Nam, Thái Lan, Ấn Độ...).
- **Nhiệm vụ của mô hình**: Phát hiện chính xác vị trí không gian (bounding box 2D ôm sát phần nhìn thấy) và phân loại các đối tượng tham gia giao thông (người đi bộ, người điều khiển phương tiện, xe con, xe tải, xe van, xe máy) trong các điều kiện thực tế (ban ngày, chập tối, ban đêm, trời mưa, mặt đường ướt, che khuất nhau và mật độ dày đặc).
- **Hậu quả của các loại sai sót nhãn (tác động tới an toàn xe tự lái)**:
  - *Bỏ sót (False Negative)*: Xe tự lái không phát hiện người đi bộ hoặc xe máy đang di chuyển $\to$ Nguy cơ va chạm gây thương vong nghiêm trọng.
  - *Box lỏng / sai kích thước (Loose Box)*: Box bao thừa cỏ, mặt đường hoặc khoảng trống quá 5 px $\to$ Mô hình học nền như một phần của vật thể; thuật toán ước lượng khoảng cách dựa trên kích thước box sẽ tính sai cự ly phanh an toàn.
  - *Thừa box (False Positive) trên vật ảo / phản chiếu / hình in*: Gán box lên hình người trên biển quảng cáo (`img_018.jpg`), bóng xe phản chiếu qua gương (`img_020.jpg`) hoặc tượng đá (`img_017.jpg`, `img_027.jpg`) $\to$ Gây hiện tượng phanh ảo (phantom braking) nguy hiểm khi xe đang chạy tốc độ cao, dẫn đến nguy cơ va chạm dây chuyền từ phía sau.
  - *Sai lớp (Misclassification)*: Nhầm lẫn giữa xe con và xe tải/xe buýt $\to$ Tính toán sai động học, vùng mù và quỹ đạo dự đoán của phương tiện xung quanh.

---

## 2. Lớp và thuộc tính

### 2.1. Phép thử phân loại (Lớp vs Thuộc tính)
Áp dụng phép thử từ bài giảng: *"Mô hình có phải phân biệt và đưa ra thông tin này khi suy luận thực tế (inference) không?"*
- **CÓ** $\to$ Xếp vào **Lớp (Class)**: Dành cho các loại đối tượng có đặc điểm hình thái và hành vi giao thông riêng biệt mà hệ thống điều khiển xe cần biết để ra quyết định (`car`, `truck`, `van`, `motorcycle`, `person`).
- **KHÔNG** $\to$ Xếp vào **Thuộc tính (Attribute)**: Dành cho thông tin mô tả trạng thái quan sát của vật thể trong ảnh (`occluded_level`, `truncated`, `can_review`) phục vụ lọc mẫu khó, kiểm soát chất lượng (QC) và phân tích lỗi.

### 2.2. Bảng định nghĩa lớp (Classes)

Mỗi đối tượng vật lý chỉ thuộc về **đúng một lớp**.

| Lớp | Định nghĩa (1 câu) | Ví dụ trong 40 ảnh | Phản ví dụ / Dễ nhầm với |
|---|---|---|---|
| `car` | Phương tiện cơ giới 4 bánh trở lên chở người (4–9 chỗ) bao gồm sedan, hatchback, SUV, crossover, coupe, station wagon, có nắp cốp hoặc khoang hành lý liền phía sau. | - Ô tô xanh trong trời mưa (`img_015.jpg`)<br>- Xe wagon AMC Pacer (`img_024.jpg`)<br>- 8 ô tô con trên sàn xe chở xe (`img_002.jpg`)<br>- Xe ô tô diễu hành (`img_019.jpg`) | - Xe bán tải thùng hở (`img_001.jpg`, `img_012.jpg` $\to$ thuộc `truck`)<br>- Tượng đài xe bán tải bằng đá (`img_027.jpg` $\to$ KHÔNG gán)<br>- Ô tô đồ chơi trong tủ kính (`img_038.jpg` $\to$ KHÔNG gán) |
| `truck` | Phương tiện cơ giới chuyên chở hàng hóa có kết cấu cabin điều khiển tách biệt vật lý với thùng hàng (thùng hở, thùng kín, sàn phẳng hoặc bồn), bao gồm cả xe bán tải (pickup) và xe chở chuyên dụng. | - Xe tải vận chuyển ô tô 2 tầng Renault 380 (`img_002.jpg`)<br>- Xe bán tải đậu quán ăn (`img_001.jpg`, `img_012.jpg`)<br>- Xe tải trong kho ban đêm (`img_007.jpg`)<br>- Xe tải nhỏ CMC Veryca (`img_036.jpg`) | - Xe van chở hàng thùng liền cabin (`van`)<br>- Bán tải có nắp thùng cao ngang cabin Master 75 (`img_037.jpg` $\to$ tạm gán `truck` kèm `can_review`)<br>- Tượng xe nghĩa trang (`img_027.jpg` $\to$ KHÔNG gán) |
| `van` | Phương tiện ô tô thân liền khối một khoang (monovolume/unibody) với cabin và khoang chở hàng/chở người thông suốt, không tách rời thùng hàng. | - Xe minivan chở khách nhỏ, xe van bưu chính chuyển phát nhanh chạy phố. | - Xe bán tải thùng hàng rời (`truck`)<br>- Xe buýt chở khách lớn (`img_009.jpg`, `img_032.jpg` $\to$ thiếu lớp trong v0)<br>- Xe SUV (`car`) |
| `motorcycle` | Phương tiện cơ giới 2 bánh (hoặc có gắn sidecar/thùng bên) có động cơ, người lái ngồi cưỡi lên yên xe và điều khiển bằng tay lái ghi-đông. | - Dòng xe máy trên phố (`img_005.jpg`, `img_008.jpg`, `img_014.jpg`, `img_021.jpg`, `img_026.jpg`, `img_030.jpg`)<br>- Mô tô phân khối lớn độ (`img_011.jpg`)<br>- Mô tô có sidecar (`img_022.jpg`)<br>- Xe máy chở hàng cồng kềnh (`img_029.jpg`) | - Xe đạp, xe xích lô đạp chân (`img_003.jpg`, `img_006.jpg`, `img_023.jpg`, `img_034.jpg`, `img_039.jpg` $\to$ KHÔNG gán phương tiện vì ngoài taxonomy v0)<br>- Xe 3 bánh tuk-tuk (`img_004.jpg`, `img_035.jpg` $\to$ thiếu lớp trong v0) |
| `person` | Cá thể người thật còn sống ở mọi tư thế (đứng, đi bộ, chạy, ngồi, cúi), bao gồm người đi bộ và người điều khiển/ngồi trên các phương tiện hở (xe máy, xe đạp, xích lô). | - Phụ nữ đứng cạnh xe bán tải (`img_001.jpg`)<br>- Người lái và ngồi sau xe máy (`img_005.jpg`, `img_026.jpg`)<br>- Bác xích lô và khách (`img_006.jpg`, `img_034.jpg`)<br>- Người bán hàng rong (`img_003.jpg`, `img_010.jpg`)<br>- Người hóa trang Predator (`img_011.jpg`)<br>- Người đi bộ trên phố (`img_025.jpg`, `img_032.jpg`) | - Tượng đồng/đá người cưỡi ngựa (`img_017.jpg` $\to$ KHÔNG gán)<br>- Chân dung in trên biển quảng cáo (`img_018.jpg` $\to$ KHÔNG gán)<br>- Ma-nơ-canh trong tủ kính (`img_033.jpg` $\to$ KHÔNG gán)<br>- Búp bê đồ chơi (`img_038.jpg` $\to$ KHÔNG gán)<br>- Người ngồi kín trong cabin ô tô (`img_001.jpg`, `img_015.jpg` $\to$ KHÔNG gán riêng person) |
| `ignore_region` | Vùng chữ nhật đánh dấu khu vực không gán chi tiết từng vật (vùng đám đông dày đặc ở xa, cụm xe/người có chiều cao $< 20$ px hoặc vùng quá tối/mờ không nhận diện được). | - Đám đông người và xe máy ở hậu cảnh xa xôi (`img_008.jpg`, `img_025.jpg`, `img_030.jpg`, `img_040.jpg`). | - Bất kỳ vật thể nào có chiều cao $\ge 20$ px và nhận dạng rõ ràng ở tiền cảnh thì BẮT BUỘC phải gán box riêng, không được gộp vào `ignore_region`. |

### 2.3. Quy tắc xử lý các vật thể nằm ngoài Taxonomy v0 (Phát hiện từ 40 ảnh)
1. **Xe ba bánh (Tuk-tuk, xe lam, ba gác máy - `img_004.jpg`, `img_016.jpg`, `img_035.jpg`)**:
   - *Vấn đề của v0*: Bản v0 hoàn toàn không có lớp cho xe 3 bánh.
   - *Đề xuất cá nhân cho Pha 2*: Nhóm cần bổ sung lớp `tuk_tuk` (hoặc `three_wheeler`).
   - *Tạm thời ở bản độc lập*: Gán 1 box bao trọn thân xe và mui vòm; xếp tạm vào lớp gần nhất (`motorcycle` do có tay lái ghi-đông hoặc `car` do có mui khoang) và BẮT BUỘC tick `can_review = true` để đưa ra thảo luận nhóm.
2. **Xe buýt chở khách lớn (Bus - `img_009.jpg`, `img_016.jpg`, `img_031.jpg`, `img_032.jpg`, `img_037.jpg`, `img_040.jpg`)**:
   - *Vấn đề của v0*: Bản v0 thiếu lớp `bus`. Xe buýt có kích thước, động học và số lượng hành khách hoàn toàn khác `car` và `truck`.
   - *Đề xuất cá nhân cho Pha 2*: Bổ sung lớp `bus` vào `taxonomy.json`.
   - *Tạm thời ở bản độc lập*: Tạm gán `truck` (phương tiện thương mại lớn) hoặc `van` (thân liền khối chở người) và tick `can_review = true`.
3. **Xe thô sơ, xe không động cơ (Xe đạp, xích lô, xe bò kéo - `img_003.jpg`, `img_006.jpg`, `img_010.jpg`, `img_023.jpg`, `img_034.jpg`, `img_039.jpg`)**:
   - *Quy tắc*: Xe đạp, xích lô, xe kéo nằm NGOÀI phạm vi taxonomy v0 $\to$ **KHÔNG gán box phương tiện**.
   - Tuy nhiên, người lái xe đạp, người đạp xích lô, người bán hàng rong đẩy xe và hành khách ngồi trên xe vẫn là người thật tham gia giao thông $\to$ **BẮT BUỘC gán box lớp `person`** cho từng cá thể người nhìn thấy.

### 2.4. Bảng thuộc tính định lượng (Attributes & Quantitative Thresholds)

Bản v0 dùng từ ngữ định tính mơ hồ ("che ít", "che nhiều", "không rõ"). Bản v0.1 này chuẩn hóa thành các ngưỡng phần trăm diện tích cụ thể:

| Thuộc tính | Kiểu dữ liệu | Giá trị cố định | Định nghĩa & Ngưỡng định lượng cụ thể |
|---|---|---|---|
| `occluded_level` | `select` | `0` | **Thấy rõ / Nguyên vẹn**: Vật thể hiển thị đầy đủ, không bị vật khác che khuất, diện tích che khuất $0\% \le \text{che} < 1\%$. |
| | | `1` | **Bị che nhẹ ($1\% \le \text{che} < 50\%$)**: Vật thể bị che khuất một phần bởi cột điện, người, xe khác hoặc đồ vật, nhưng thấy được phần lớn thân xe/cơ thể ($> 50\%$). Nhận dạng chắc chắn. |
| | | `2` | **Bị che nặng ($50\% \le \text{che} \le 90\%$)**: Vật thể bị che từ một nửa diện tích trở lên (ví dụ chỉ thấy nửa đầu xe, nửa thân sau, hoặc chỉ thấy từ ngực trở lên). Phần nhìn thấy vẫn đủ để nhận diện chắc chắn loại đối tượng. |
| | | `3` | **Không xác định / Bị che cực nặng ($> 90\%$)**: Chỉ nhìn thấy một phần rất nhỏ hoặc bị bóng mờ nhạt nhòa, không thể ước lượng chính xác ranh giới và tỉ lệ che khuất. |
| `truncated` | `select` | `no` | **Không cắt mép**: Toàn bộ đường viền nhìn thấy của vật thể nằm hoàn toàn bên trong 4 cạnh của khung ảnh. |
| | | `yes` | **Bị cắt mép ảnh**: Thân xe hoặc cơ thể người bị cắt ngang bởi mép viền trên, dưới, trái hoặc phải của bức ảnh (một phần đối tượng nằm ngoài khung hình). |
| `can_review` | `checkbox` | `false` | Bình thường: Người gán hoàn toàn tự tin về nhãn lớp, tọa độ box và các thuộc tính. |
| | | `true` | **Cần xem xét lại**: Người gán phân vân, gặp ca biên mơ hồ (chưa có quy tắc rõ ràng trong v0, ca tranh chấp giữa 2 lớp, hoặc ranh giới box mờ do thời tiết). Đánh dấu để họp calibration. |

---

## 3. Quy tắc vẽ box (Bounding Box Rules)

1. **Box ôm sát phần nhìn thấy (Tight Visible Bounding Box)**:
   - Box chữ nhật phải ôm khít các điểm ngoài cùng của **phần nhìn thấy** của vật thể.
   - Không chừa lề trống (padding $< 5$ px), không bao gồm mặt đường thừa, vỉa hè hoặc bầu trời (tránh lỗi box lỏng khiến mô hình học nhầm bối cảnh - slide 21, 22).
   - Với vật bị che khuất: **CHỈ bao phần nhìn thấy** (visible extent). Tuyệt đối KHÔNG tự ý vẽ phỏng đoán phần bị che khuất (amodal box) vì mô hình 2D chỉ học đặc trưng điểm ảnh nhìn thấy được (slide 21, 24).
2. **Quy tắc gương chiếu hậu (Side-view Mirrors)**:
   - Theo thống nhất tại slide 30: **Gương chiếu hậu của xe KHÔNG bao vào box xe**. Box của `car`, `truck`, `van` chỉ ôm sát thân vỏ chính của xe, bỏ qua phần gương chìa ra hai bên.
3. **Quy tắc người và xe máy (Riders & Vehicles)**:
   - Mô hình ADAS cần phát hiện độc lập người và phương tiện (slide 25, 36, 38).
   - Mỗi người ngồi trên xe máy được gán **2 box độc lập**:
     - 1 box `motorcycle` ôm trọn thân xe máy nhìn thấy (khung xe, bánh xe, tay lái, ống xả).
     - 1 box `person` ôm trọn cơ thể người nhìn thấy (từ đỉnh mũ bảo hiểm đến bàn chân/bàn đạp).
     - Việc hai box này chồng lấn diện tích (overlap) lên nhau là đúng chuẩn.
   - Nếu xe máy chở nhiều người (`img_005.jpg`, `img_026.jpg`): Gán **1 box `motorcycle`** và **$N$ box `person`** cho từng người (người lái + các hành khách ngồi sau).
4. **Quy tắc người trong khoang kín**:
   - Người ngồi hoàn toàn bên trong khoang kính kín của ô tô (`car`), xe tải (`truck`), xe van (`van`) $\to$ **KHÔNG gán box `person`** (người là một phần nội thất của xe - slide 45).
   - Người ngồi trên xe hở mui, xe máy, xe đạp, xích lô, xe đẩy $\to$ BẮT BUỘC gán `person`.
   - Với xe ba bánh tuk-tuk (`img_004.jpg`, `img_035.jpg`): Người ngồi hoàn toàn dưới mui vòm phía sau $\to$ KHÔNG gán person; người lái ngồi phía trước có phần cơ thể lộ ra ngoài mui $\to$ Gán `person` (slide 33).
5. **Quy tắc hàng hóa cồng kềnh gắn liền với phương tiện (Attached Cargo)**:
   - Theo slide 34, 37, 39: Đối với xe ba bánh chở hàng (`img_035.jpg`) hoặc xe máy chở hàng cồng kềnh (`img_029.jpg`), hàng hóa chất trên nóc/sau xe gắn liền vật lý với xe $\to$ **Box xe ôm trọn cả thân xe và khối hàng hóa**.
   - *Lý do an toàn ADAS*: Hàng hóa chiếm dụng diện tích làn đường và là chướng ngại vật thực tế; xe tự lái phải nhận diện toàn bộ khối thể tích để né tránh.
6. **Quy tắc xe chở ô tô (Car Carrier - `img_002.jpg`)**:
   - Gán **1 box `truck`** bao toàn bộ xe tải chở hàng (đầu kéo + toàn bộ rơ-moóc 2 tầng).
   - Gán riêng **8 box `car`** cho 8 chiếc ô tô con được chở trên sàn xe, vì mỗi chiếc ô tô con có đầy đủ đặc trưng hình học của một chiếc xe con mà mô hình cần học.
7. **Quy tắc loại trừ vật thể phi thực / 2D / Mô hình (Strict Exclusion Rules - slide 26)**:
   - **Tượng điêu khắc / Bia mộ (`img_017.jpg`, `img_027.jpg`)**: **KHÔNG GÁN**. Tượng đồng Duke of Wellington cưỡi ngựa (`img_017.jpg`) hay tượng đài xe bán tải chạm khắc bằng đá ở nghĩa trang (`img_027.jpg`) là công trình điêu khắc tĩnh, không phải người thật hay phương tiện giao thông.
   - **Hình in 2D / Biển quảng cáo (`img_018.jpg`)**: **KHÔNG GÁN**. Hình ảnh chân dung người hoặc hình vẽ xe in trên billboard, áp phích tường, biển hiệu cửa hàng đều bị loại bỏ.
   - **Phản chiếu gương và mặt kính (`img_020.jpg`)**: **KHÔNG GÁN**. Hình ảnh người hoặc xe phản chiếu qua gương chiếu hậu hoặc cửa kính tòa nhà không có thực thể vật lý trên mặt đường.
   - **Ma-nơ-canh và đồ chơi mô hình (`img_033.jpg`, `img_038.jpg`)**: **KHÔNG GÁN**. Ma-nơ-canh trưng bày quần áo và xe mô hình trong tủ kính không tham gia giao thông.
   - **Trang phục hóa trang người thật (`img_011.jpg`)**: Người thật mặc trang phục cosplay Predator $\to$ Vẫn là người thật di chuyển $\to$ **GÁN `person`**.
8. **Quy tắc loại trừ xe chủ (Ego-Vehicle Exclusion - slide 26)**:
   - Nắp ca-pô, cần gạt nước hoặc mép gương chiếu hậu của chính chiếc xe gắn camera thu thập dữ liệu (thường xuất hiện ở cạnh đáy ảnh như `img_015.jpg`, `img_020.jpg`) $\to$ **KHÔNG GÁN**.
9. **Quy tắc ngưỡng kích thước và vùng ignore (< 20 px / Đám đông - slide 16, 30)**:
   - Đối tượng có chiều cao $< 20$ px: **KHÔNG gán từng box riêng lẻ**.
   - Nếu các đối tượng $< 20$ px hoặc người/xe dày đặc ở hậu cảnh xa xôi (`img_008.jpg`, `img_025.jpg`, `img_030.jpg`, `img_040.jpg`): Vẽ một box chữ nhật lớp `ignore_region` bao quanh khu vực đó. Các đối tượng nằm trong vùng này sẽ được bỏ qua khi tính lỗi bỏ sót.
10. **Quy tắc ảnh mờ, trời mưa, ban đêm (Slide 30, 41)**:
    - Điều kiện thời tiết bất lợi (mưa `img_013.jpg`, `img_015.jpg`; tối `img_007.jpg`, `img_008.jpg`; mờ nhòe `img_014.jpg`): Vẫn áp dụng các quy tắc gán nhãn tiêu chuẩn nếu mắt người nhận dạng được đối tượng.
    - Nếu đối tượng bị nhòe chuyển động quá nặng hoặc quá tối không xác định được chính xác ranh giới $\to$ Gán box ước lượng tốt nhất, chọn `occluded_level = 2` hoặc `3` và tick `can_review = true`.

---

## 4. Ca khó (Edge Cases Catalog từ 40 ảnh)

Dưới đây là bảng 15 ca khó điển hình được phát hiện sau khi khảo sát toàn bộ 40 ảnh, phân biệt rõ giữa **ca loại theo quy tắc** (đã có quy tắc chuẩn giải quyết) và **ca cần hỏi reviewer / họp nhóm** (ca biên cần thống nhất ở Pha 2).

| Ảnh | Tình huống cụ thể | Rule ID | Quyết định cá nhân | Lý do chi tiết | Phân loại ca |
|---|---|---|---|---|---|
| `img_001.jpg` | Xe bán tải cũ đậu trước quán ăn | `R-09` | Gán `truck`, `occluded_level=0`, `truncated=no`. Gán người phụ nữ đứng cạnh là `person`. | Xe bán tải có cabin và thùng hàng riêng biệt phía sau, thuộc định nghĩa `truck`. Người phụ nữ đứng ngoài xe là người thật. | Quy tắc |
| `img_002.jpg` | Xe tải Renault 380 chở 8 ô tô con trên 2 tầng sàn | `R-07` | Gán 1 box `truck` cho toàn bộ xe chở xe; gán 8 box `car` riêng cho 8 xe con trên sàn chở. | Xe tải chở hàng là 1 phương tiện chuyên dụng lớn; các xe con trên sàn là vật thể cản trở có đầy đủ hình thái xe con mà xe tự lái cần nhận diện. | Quy tắc |
| `img_003.jpg` | Người bán mít đẩy xe đạp trên đường phố Hà Nội | `R-06` | Gán 1 box `person` cho người bán hàng; KHÔNG gán xe đạp (ngoài taxonomy v0). | Xe đạp không thuộc taxonomy v0. Người bán hàng là người tham gia giao thông trên mặt đường nên phải được phát hiện để tránh va chạm. | Quy tắc |
| `img_004.jpg` | Hai xe tuk-tuk 3 bánh màu xanh đậu trên vỉa hè | `R-08` | Tạm gán 2 box `motorcycle` (hoặc `car`) bao trọn thân và mui; tick cờ `can_review = true`. | v0 chưa có lớp `tuk_tuk` / `three_wheeler`. Cần đề xuất nhóm bổ sung lớp mới ở Pha 2. | **Cần hỏi Reviewer** |
| `img_005.jpg` | Dòng người đi xe máy tại TP.HCM, người và xe che nhau | `R-06` | Mỗi xe máy gán 1 `motorcycle`; mỗi người ngồi trên xe gán 1 `person` riêng. Đo `occluded_level` theo phần thấy. | Tách độc lập người và phương tiện theo chuẩn ADAS; box chỉ ôm phần nhìn thấy, không đoán phần bị che. | Quy tắc |
| `img_011.jpg` | Người mặc trang phục hóa trang Predator cạnh dàn xe mô tô | `R-05` | Gán `person` cho người hóa trang Predator (`occluded_level=0`); gán `motorcycle` cho các xe mô tô. | Người hóa trang vẫn là cơ thể người thật còn sống di chuyển trên đường, không phải tượng tĩnh hay ma-nơ-canh. | Quy tắc |
| `img_014.jpg` | Người đi bộ qua đường Sài Gòn, ảnh bị nhòe chuyển động (motion blur) | `R-10` | Gán `person` và `motorcycle` cho các đối tượng nhận dạng được, chọn `occluded_level=2` hoặc `3`, tick `can_review = true`. | Ảnh mờ nhòe do tốc độ màn trập chậm; vẫn gán các vật thể nhận biết được nhưng đánh dấu cờ để reviewer kiểm tra lại ranh giới box. | **Cần hỏi Reviewer** |
| `img_017.jpg` | Tượng đồng Duke of Wellington cưỡi ngựa đội nón giao thông | `R-01` | **KHÔNG GÁN** bất kỳ box nào (0 annotation). | Đây là tượng điêu khắc bằng kim loại/đá đặt trên bệ cao, không phải người thật hay phương tiện giao thông. Gán person ở đây là lỗi thừa box (False Positive). | Quy tắc |
| `img_018.jpg` | Biển quảng cáo chính trị tấm lớn in hình chân dung ứng viên | `R-02` | **KHÔNG GÁN** (0 annotation). | Hình in 2D trên biển quảng cáo phẳng, không có thực thể người 3D trên mặt đường. Xe tự lái không được phanh trước ảnh chụp 2D. | Quy tắc |
| `img_020.jpg` | Gương chiếu hậu ô tô phản chiếu cầu vồng và cảnh vật | `R-03` | **KHÔNG GÁN** (0 annotation). | Gương chiếu hậu thuộc xe chủ (ego-vehicle) không gán; hình ảnh trong gương là phản chiếu quang học, không gán. | Quy tắc |
| `img_027.jpg` | Tượng đài xe bán tải chạm khắc bằng đá nguyên khối trong nghĩa trang | `R-01` | **KHÔNG GÁN** (0 annotation). | Tác phẩm điêu khắc bia mộ bằng đá tĩnh, không phải xe cơ giới thật đang vận hành trên đường. | Quy tắc |
| `img_033.jpg` | Ma-nơ-canh mặc áo vest trưng bày trong tủ kính cửa hàng | `R-05` | **KHÔNG GÁN** (0 annotation). | Ma-nơ-canh tĩnh bằng nhựa/thạch cao phía sau kính trưng bày, không phải người thật tham gia giao thông. | Quy tắc |
| `img_035.jpg` | Xe tuk-tuk chở hàng hóa bao tải chất cao khổng lồ tại Hyderabad | `R-08` | Box ôm trọn thân xe tuk-tuk VÀ toàn bộ khối hàng bao tải phía trên; tick `can_review = true`. | Hàng hóa dính liền với xe chiếm dụng không gian vật lý lớn; xe tự lái phải tránh cả khối hàng để không gây tai nạn. Tick can_review vì v0 thiếu lớp tuk-tuk. | **Cần hỏi Reviewer** |
| `img_037.jpg` | Xe bán tải Ford gắn nắp thùng cao ngang cabin Master 75 | `R-09` | Tạm gán `truck`, `occluded_level=0`, `truncated=no`, tick `can_review = true`. Phía sau có xe buýt $\to$ tick `can_review`. | Ca biên gây tranh cãi: chassis bán tải nhưng nắp thùng cao kín như xe van. Tạm chọn `truck` (dựa trên khung gầm pickup) và đưa ra họp Pha 2 để chốt quy tắc nhóm. | **Cần hỏi Reviewer** |
| `img_038.jpg` | Tủ kính cửa hàng đồ chơi bày hàng chục xe ô tô và búp bê mô hình | `R-05` | **KHÔNG GÁN** bất kỳ box nào (0 annotation). | Toàn bộ là đồ chơi mô hình thu nhỏ trưng bày sau kính, không phải phương tiện hay người thật. | Quy tắc |

---

## 5. QC và phiên bản

### 5.1. Bộ ngưỡng định lượng QC áp dụng (Quality Metrics)
Toàn bộ quy trình kiểm soát chất lượng của nhóm tuân thủ nghiêm ngặt các ngưỡng đo định lượng sau:
- **IoU đơn lẻ của box ghép cặp**:
  - Đối tượng lớn (chiều cao $\ge 50$ px): $\text{IoU} \ge 0.85$.
  - Đối tượng nhỏ (chiều cao từ 20 đến 50 px): $\text{IoU} \ge 0.70$.
- **Ngưỡng ghép cặp box (Match IoU)**: $\text{IoU} \ge 0.50$ (hai box của 2 annotator có $\text{IoU} \ge 0.50$ được coi là cùng gán một đối tượng).
- **Chỉ số đồng thuận liên annotator (IAA - Inter-Annotator Agreement)**:
  - $\text{IoU trung bình trên các cặp ghép} \ge 0.85$.
  - $\text{Độ đồng thuận nhãn lớp (Class Agreement)} \ge 95\%$.
  - *Lưu ý*: IAA chỉ tính trên các cặp box ghép được 1–1. Các box bỏ sót/thừa được theo dõi ở danh mục lỗi riêng (`bo_sot_thua`).
- **Ngưỡng nghiệm thu xuất xưởng (Release Threshold)**:
  - Tỉ lệ ảnh có lỗi: $\le 2.0\%$ trên tổng số ảnh đã duyệt ($\le 2\%$ nghĩa là trong lô 40 ảnh, chỉ cần $\ge 1$ ảnh bị lỗi $= 2.5\%$ là BỊ TỪ CHỐI Release $\to$ buộc phải chuyển sang **Rework**).
  - Không có lỗi hệ thống lặp lại ($\ge 3$ lỗi cùng loại và chiếm $\ge 50\%$ tổng số lỗi).

### 5.2. Quy trình xử lý ca mơ hồ (Escalation Process)
1. Khi phát hiện tình huống chưa rõ ràng hoặc gây tranh cãi: **Tuyệt đối không tự suy đoán hoặc hỏi bài nhau trong lúc gán độc lập**.
2. Annotator vẽ box theo nhận định cá nhân tốt nhất, tick chọn thuộc tính `can_review = true`.
3. Nếu đối tượng không thể vẽ box (ví dụ vỡ hình, không xác định được loại vật thể): Ghi lại mã ảnh, tọa độ vùng và lý do vào nhật ký cá nhân để đưa vào `submission/qc/image_issues.csv`.
4. Trong buổi họp Calibration (Pha 5): Cả nhóm mở danh sách các box có `can_review = true`, cùng thảo luận trên ảnh phóng to, thống nhất một quyết định duy nhất và nâng cấp guideline lên phiên bản `v1.1`.

### 5.3. Bảng lịch sử thay đổi (Changelog)

| Phiên bản | Ngày | Người thực hiện | Nội dung thay đổi chính | Lý do / Ca dẫn chứng |
|---|---|---|---|---|
| `v0` | 2026-11-16 | Giảng viên | Bản khởi đầu sơ sài: 5 lớp cơ bản, thuộc tính định tính không có ngưỡng phần trăm, chưa có quy tắc ca biên. | Khung bài lab ban đầu. |
| `v0.1_draft` | 2026-11-16 | Thành viên 101 | - Viết đủ 5 phần chuẩn theo `TEMPLATE_guideline.md`.<br>- Định lượng hóa 4 mức `occluded_level` bằng ngưỡng $\%$ diện tích cụ thể ($0\%$, $1-50\%$, $50-90\%$, $>90\%$).<br>- Bổ sung quy tắc chặt chẽ: loại trừ tượng điêu khắc (`R-01`), hình in 2D (`R-02`), phản chiếu gương (`R-03`), xe chủ (`R-04`), ma-nơ-canh & đồ chơi (`R-05`).<br>- Quy định rõ: người đi xe máy gán 2 box độc lập (`R-06`); xe chở ô tô gán 1 truck + 8 car (`R-07`); xe chở hàng cồng kềnh box bao cả hàng (`R-08`); bán tải thùng hở gán truck (`R-09`).<br>- Quy định vùng `ignore_region` cho vật $< 20$ px và đám đông dày đặc (`R-10`).<br>- Lập danh mục 15 ca khó với ID ảnh thực tế từ 40 ảnh và đề xuất bổ sung lớp `tuk_tuk`, `bus` cho Pha 2. | Khảo sát thực tế 40 ảnh trong `data/images/` phát hiện nhiều ca biên mà v0 bỏ ngỏ (`img_002.jpg`, `img_004.jpg`, `img_011.jpg`, `img_014.jpg`, `img_017.jpg`, `img_018.jpg`, `img_020.jpg`, `img_027.jpg`, `img_033.jpg`, `img_035.jpg`, `img_037.jpg`, `img_038.jpg`). |
