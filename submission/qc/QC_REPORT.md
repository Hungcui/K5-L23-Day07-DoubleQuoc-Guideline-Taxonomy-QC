# Báo cáo QC — Nhóm DoubleQuoc

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Đặc tả và QC nhãn 2D cho bài toán phát hiện đối tượng giao thông đường phố (ADAS); giới hạn độ đại diện: tập 40 ảnh tuyển chọn các ca biên phức tạp, chưa đại diện đầy đủ cho phân phối thời tiết ban đêm và thời tiết cực đoan trong vận hành thực tế.
- Nguồn và phép sử dụng dữ liệu: Xem ATTRIBUTION.csv, ảnh tuân thủ giấy phép CC BY/CC0 tương ứng từ các tác giả nguồn.
- Manifest và checksum nhãn cuối: batch_manifest.json (SHA-256 nhãn cuối labels_calibrated.json: f178731fc95c2c4f42588f51c9845d857ac2b68e515149b91b842143fc0b3f38).
- Guideline và taxonomy khi gán / khi bàn giao: Khi gán độc lập dùng taxonomy v1.0 và guideline_v1.0_agreed.md; khi bàn giao nâng cấp guideline lên guideline_v1.1_calibrated.md (loại editorial làm rõ quy tắc R-10, R-11, R-12), giữ nguyên taxonomy v1.0.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80 (mỗi thành viên 038 và 039 hoàn thành đủ 40/40 ảnh được phân công).
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 285 cặp box ghép, IoU trung bình 0.966 (vượt chuẩn >= 0.85), đồng thuận nhãn lớp đạt 93.3% trên mẫu số 285 cặp box ghép.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 35 box chưa ghép của 038 và 30 box chưa ghép của 039; 5 box bỏ qua do ignore; 24 ca bất đồng liên quan đến ignore_region.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: Tại img_017.jpg (tượng Duke of Wellington) cả hai annotator có thể bị nhầm gán person/horse nếu không đọc kỹ quy tắc R-06 về tượng cố định; và tại img_018.jpg áp phích quảng cáo 2D không được gán person.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: 116 ca bất thường ghi nhận trong calibration_candidates.csv đã được xử lý 100% trong calibration_decisions.csv; áp dụng các rule ID R-01, R-02, R-03, R-05, R-06, R-09, R-10, R-11, R-12; bản sửa được gộp chuẩn xác và xác nhận bởi 039 (đạt).
- Tác động v1.1 lên nhãn và ảnh: Làm rõ cách phân biệt giữa motorcycle/bicycle với person theo R-02, bổ sung quy tắc R-10 (van vs car), R-11 (truncated) và R-12 (ưu tiên box cá thể rõ so với ignore); phạm vi áp dụng trên 27 ảnh có bất đồng trong lô.
- Tham chiếu bằng chứng khi Rework: Không phát sinh vòng Rework do các ca bất thường đã được thống nhất triệt để trong phiên họp calibration trước khi tạo bản nhãn cuối.

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: 039, 2026-11-16
- Đã duyệt / cả lô: 40/40 ảnh đã duyệt (100% toàn bộ lô)
- Ảnh lỗi / ảnh đã duyệt: 0/40 = 0.0%
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/350 = 0.0%
- Vật bỏ sót, ảnh và vùng: 0 vật bỏ sót sau khi hiệu chuẩn
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Ca khó duyệt 27/27 lỗi 0.0%, phần còn lại duyệt 13/13 lỗi 0.0%; không phát sinh lỗi lặp; chi tiết tại submission/qc/reviews/round-01/qc_review.csv
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Máy tự động chặn lỗi cú pháp và lệch schema/tọa độ ngoài khung hình; người cần đọc ảnh để phân biệt giữa tranh vẽ 2D / ma-nơ-canh (R-06) với người thật.

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: Chất lượng bộ nhãn đạt 100% yêu cầu sau calibration với tỷ lệ lỗi 0.0% trên toàn bộ 40 ảnh; bàn giao cho đội ngũ huấn luyện mô hình ADAS VinUni AI20k để phục vụ thực hành phát hiện đối tượng giao thông 2D.
- Nếu còn việc: Không còn tồn đọng; toàn bộ nhãn và tài liệu đã hoàn thiện và kiểm thử đạt chuẩn.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Ca C001 (img_004.jpg) áp dụng R-05 gán other_vehicle kết quả đạt; ca C007 (img_005.jpg) áp dụng R-02 gộp người vào motorcycle kết quả đạt; kiểm tra chéo toàn bộ 116 ca qua calibration_decisions.csv và labels_calibrated.json.
