# Báo cáo QC — Team Secret

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Nhóm xây dựng nhãn người và phương tiện 2D trên 40 ảnh để thực hành guideline và QC; chưa dùng làm dữ liệu vận hành thực tế.
- Nguồn và phép sử dụng dữ liệu: Giữ tác giả, giấy phép và nguồn của từng ảnh trong data/ATTRIBUTION.csv.
- Manifest và checksum nhãn cuối: batch_manifest.json; SHA-256 của labels_calibrated.json là d3e2305ee9ed5f4fd072ff8679f1fea3c833b7dc71c3c96f9f0dcef74b33ead4.
- Guideline và taxonomy khi gán / khi bàn giao: Gán theo guideline v1.0, calibration theo v1.1; taxonomy giữ v1.0, không đổi rules.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80. Mã 199 gán 26 ảnh, 204 gán 27 ảnh, 215 gán 27 ảnh; 190 phụ trách review.
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 252 cặp; IoU TB 0.9062; đồng thuận lớp 249/252 = 98.81%. Chỉ tính trên cặp ghép được.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: Phía A 24, phía B 103; loại do cùng ignore 0; bất đồng ignore 19 ca. Đây là số trước sửa.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: img_018, cả hai gán xe Volkswagen thân hộp là car. Theo R-14 phải dùng van; đã ghi Q09 và sửa ở vòng 2.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: calibration_decisions.csv giữ 321 ca IAA và Q01–Q12; mỗi ca có rule, hướng sửa và mã người chốt 215. Xác nhận review độc lập của 190 còn thiếu.
- Tác động v1.1 lên nhãn và ảnh: Phạm vi sửa 32/40 ảnh; bản hiện tại có 404 annotation (392 đối tượng, 12 ignore). Giữ nguyên nhãn cá nhân và số IAA trước sửa.
- Tham chiếu bằng chứng khi Rework: round-01 ghi 11 lỗi quan sát trên 7 ảnh; các bản sửa ở round-02. Q12 giữ cờ kiểm biên người trong img_010; hai vòng đều được lưu.

## 4. Kết quả duyệt

- Vòng QC: 2
- Người review và ngày: 190 phụ trách; ngày lập báo cáo 17/11/2026. Chưa có xác nhận đã duyệt của 190.
- Đã duyệt / cả lô: 0/40 được reviewer xác nhận; đã chuẩn bị và đối chiếu ảnh chồng box cho 40/40 ảnh.
- Ảnh lỗi / ảnh đã duyệt: Chưa có tỉ lệ duyệt chính thức (mẫu số 0). Đối chiếu vòng 1 ghi vấn đề trên 7/40 ảnh; không coi đây là kết quả reviewer đã ký.
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: Chưa có tỉ lệ duyệt chính thức; vòng 2 có 392 đối tượng để kiểm, chưa có đối tượng được reviewer xác nhận.
- Vật bỏ sót, ảnh và vùng: Q11 bổ sung người sau cabin trong img_036, vùng [724,151,30,51]; vẫn giữ cờ để xác minh.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Phiếu gồm 25 ảnh bị cờ IAA và 15 ảnh còn lại. Còn 47 box can_review trên 18 ảnh. Lỗi truncated lặp ở img_001, img_005, img_008, img_021 đã sửa, chờ kiểm lại.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Script phát hiện bbox vượt biên; chọn van thay cho car trong img_018 cần nhìn cấu tạo xe.

## 5. Quyết định bàn giao

- Quyết định: Hold
- Lý do, người nhận và phạm vi sử dụng: Thiếu xác nhận reviewer, còn 47 cờ review và chưa có ghi chú cá nhân. Gói để coach xem bằng chứng thực hành, chưa xác nhận Release.
- Nếu còn việc: Thành viên tự viết individual_notes; 190 xác minh cờ và hoàn thiện phiếu review. Nhóm cần đối chiếu yêu cầu mỗi thành viên có COCO với việc 190 đang chỉ làm review. Nếu thay nhãn sau vòng 2, giữ vòng cũ và mở vòng mới.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Q09 → img_018 → R-14 → annotation 174: round-01/labels_snapshot.json ghi car, round-02/labels_snapshot.json ghi van. Chuỗi này đã đối chiếu; việc người ngoài nhóm kiểm thử gói chưa có xác nhận.
