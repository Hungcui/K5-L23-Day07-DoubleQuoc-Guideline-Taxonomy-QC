# Guideline gán nhãn nhóm — v1.1 (Calibrated) — Nhóm DoubleQuoc

- Phiên bản: v1.1 (Bản hiệu chỉnh và làm rõ sau họp Calibration)
- Ngày: 2026-11-16
- Thành viên tham gia: 
  - Thành viên 038: [Họ và tên: Nguyễn Quốc Hưng] [Mã học viên: 2A202604038]
  - Thành viên 039: [Họ và tên: Hoàng Quốc Chung] [Mã học viên: 2A202604039]
- Trạng thái: **CALIBRATED** (Đã hoàn thiện sau phân tích độ đồng thuận IAA và họp Calibration)

---

## 1. Mục tiêu và Phạm vi

- **Mục tiêu mô hình:** Phát hiện vật thể 2D (Object Detection) thời gian thực phục vụ hệ thống hỗ trợ lái xe nâng cao (ADAS) và camera giám sát giao thông thông minh.
- **Rủi ro trọng yếu:**
  - *Bỏ sót (False Negative):* Bỏ sót người đi bộ hoặc phương tiện hai bánh gây nguy cơ đâm va trực tiếp.
  - *Báo giả (False Positive):* Báo giả vật thể tĩnh (tượng, áp phích, ma-nơ-canh, ảnh phản chiếu) khiến xe phanh gấp bất ngờ (phantom braking).
  - *Sai lệch kích thước/lớp:* Nhầm xe con và xe tải/xe buýt làm sai lệch tính toán khoảng cách an toàn.

---

## 2. Giải quyết Bất đồng & Thống nhất Ý kiến Nhóm

Sau khi thảo luận kỹ lưỡng dựa trên bản cá nhân của thành viên 038 và 039, cả nhóm đã đi đến thống nhất chung:

| Vấn đề | Đề xuất của 039 | Đề xuất của 038 | Quyết định chốt của Team v1.0 | Lập luận kỹ thuật của Team |
|---|---|---|---|---|
| **1. Người lái xe máy** | Gộp chung 1 box gồm cả người và xe. | Tách riêng box người và box xe máy. | **GỘP CHUNG 1 BOX `motorcycle`** bao gồm cả xe và người ngồi trên xe. | Phương tiện và người điều khiển di chuyển như một thực thể động học thống nhất. Box gộp giúp bao trọn toàn bộ khối không gian chướng ngại vật trên đường. |
| **2. Người đi xe đạp** | Gộp chung 1 box `bicycle` gồm cả người và xe đạp. | Không gán phương tiện, chỉ gán box `person`. | **GỘP CHUNG 1 BOX `bicycle`** bao gồm cả xe đạp và người đạp xe. | Tách riêng loại hình phương tiện thô sơ có người đạp để mô hình phân biệt rõ tốc độ di chuyển chậm so với xe cơ giới. |
| **3. Vẽ box khi bị che khuất** | Vẽ box ước lượng cả phần bị che khuất. | Chỉ vẽ phần nhìn thấy được (Visible). | **CHỈ VẼ PHẦN NHÌN THẤY ĐƯỢC** (Visible area). | Chuẩn gán nhãn 2D COCO dựa trên đặc trưng pixel nhìn thấy được. Vẽ ước lượng mang tính phỏng đoán chủ quan, làm lệch chỉ số IoU giữa các người gán và gây nhiễu cho detector. |
| **4. Danh mục lớp (Taxonomy)** | Bổ sung `bicycle`, `bus`, `other_vehicle`. | Đề xuất giữ 6 lớp cũ. | **BỔ SUNG `bicycle`, `bus`, `other_vehicle`, `van`** vào Taxonomy v1.0. | Giúp phân loại chính xác các phương tiện đặc thù trong bộ ảnh thực tế: xe buýt lớn (`bus`), xe đạp (`bicycle`), xe tuk-tuk / ba gác (`other_vehicle`), xe thùng kín (`van`). |

---

## 3. Danh mục Lớp và Thuộc tính thống nhất (Taxonomy v1.0 / v1.1)

### 3.1. Danh mục 9 Lớp

| Lớp | Định nghĩa chuẩn | Ví dụ thực tế | Phản ví dụ / Không gán |
|---|---|---|---|
| `car` | Ô tô con, xe gia đình 4–7 chỗ, SUV, sedan, hatchback, coupe, taxi. | Taxi trong `img_040.jpg`, ô tô con trên sàn `img_002.jpg`. | Xe thùng kín (`van`), xe tải thùng hở (`truck`), xe buýt (`bus`). |
| `truck` | Xe tải chở hàng, xe chuyên dụng có cabin và thùng/sàn chở hàng tách rời. | Xe tải thùng `img_001.jpg`, xe cõng xe `img_002.jpg`, xe bán tải có thùng chở hàng hở phía sau. | Xe van nguyên khối (`van`), xe buýt chở khách (`bus`). |
| `van` | Xe tải nhẹ hoặc xe khách thương mại có thân hình hộp nguyên khối, khoang hàng/người liền mạch với cabin. | Xe minivan chở hàng, xe cứu thương, xe chuyển phát nhanh thùng liền kín. | Xe SUV 7 chỗ (`car`), xe tải thùng rời (`truck`). |
| `bus` | Xe buýt, xe khách chở người, phương tiện giao thông công cộng cỡ lớn. | Xe buýt nội đô, xe khách liên tỉnh, xe du lịch 29-45 chỗ. | Xe van 16 chỗ nhỏ (`van`), xe tải (`truck`). |
| `motorcycle` | Phương tiện cơ giới 2 hoặc 3 bánh vận hành bằng động cơ. Box vẽ **bao gồm cả người điều khiển và người ngồi sau**. | Xe số, xe ga trong `img_005.jpg`, `img_026.jpg`, xe phân khối lớn `img_011.jpg`. | Xe đạp (`bicycle`), xe ba gác máy/tuk-tuk (`other_vehicle`). |
| `bicycle` | Xe đạp hai bánh vận hành bằng sức người. Box vẽ **bao gồm cả xe và người đạp xe**. | Xe đạp trên đường phố, xe đạp chở hàng rong (`img_003.jpg`). | Xe máy có động cơ (`motorcycle`). |
| `other_vehicle` | Các phương tiện đặc thù khác không thuộc các lớp trên: xe tuk-tuk, xe ba gác, xích lô máy/xe lam. | Xe tuk-tuk trong `img_004.jpg`, xe auto-rickshaw trong `img_035.jpg`. | Ô tô con thông thường (`car`), xe tải (`truck`). |
| `person` | Con người còn sống thực tế đang đi bộ, đứng, hoặc ngồi (KHÔNG nằm trên xe máy hoặc xe đạp). | Người đi bộ qua đường `img_014.jpg`, người đứng nói chuyện `img_001.jpg`, xích lô đạp (`img_006.jpg`). | Tượng đài (`img_017.jpg`), áp phích (`img_018.jpg`), ma-nơ-canh (`img_033.jpg`), người trên xe máy (đã gộp vào `motorcycle`). |
| `ignore_region` | Vùng ảnh chứa đám đông hoặc cụm xe cộ dày đặc ở hậu cảnh quá xa, mờ nhòe. | Cụm người/xe li ti ở cuối đường hoặc góc tối không thể phân biệt box riêng rẽ. | Không dùng để thay thế cho vật thể nhìn rõ. Bỏ sót trong vùng này không tính là lỗi. |

### 3.2. Thuộc tính chuẩn

- **`occluded_level`** (Mức độ che khuất):
  - `0`: Rõ ràng, thấy $\ge 80\%$ diện tích thực tế của vật thể.
  - `1`: Che nhẹ, thấy từ $50\%$ đến dưới $80\%$.
  - `2`: Che nặng, thấy từ $20\%$ đến dưới $50\%$ (vẫn nhận dạng được hình dáng).
  - `3`: Che cực nặng, thấy $< 20\%$ (chỉ nhận ra qua một chi tiết nhỏ hoặc ngữ cảnh).
- **`truncated`** (Cắt mép ảnh):
  - `no`: Vật thể nằm trọn vẹn trong khung hình.
  - `yes`: Bất kỳ phần nào của thân xe hoặc cơ thể người chạm hoặc bị cắt đứt bởi mép ảnh.
- **`can_review`**:
  - `false`: Mặc định, box rõ ràng theo quy tắc.
  - `true`: Đánh dấu tick khi gặp trường hợp mơ hồ, chưa chắc chắn để reviewer kiểm lại.

---

## 4. Quy tắc kỹ thuật vẽ Box (Quy chuẩn mã hóa Rule ID)

- **R-01 (Box ôm sát phần nhìn thấy):** Bounding box phải chạm sát các điểm cực trị ngoài cùng của phần nhìn thấy (pixel-level). Không vẽ phần ước lượng bị che khuất.
- **R-02 (Gộp người và xe hai bánh):** 
  - Vẽ **1 box `motorcycle` duy nhất** bao trọn cả thân xe máy và toàn bộ người ngồi trên xe (lái xe + người ngồi sau).
  - Vẽ **1 box `bicycle` duy nhất** bao trọn cả xe đạp và người đạp xe.
- **R-03 (Người đi bộ - Lớp `person`):** Chỉ gán lớp `person` cho người đi bộ, người đứng/ngồi độc lập không điều khiển hay ngồi trên xe máy/xe đạp.
- **R-04 (Xe chở trên phương tiện khác - Car carrier):** Với xe tải cõng xe (`img_002.jpg`), vẽ 1 box `truck` cho xe tải; vẽ từng box `car` riêng cho các ô tô con nằm trên sàn xe nếu thấy rõ đặc trưng xe.
- **R-05 (Xe Tuk-tuk và xe đặc thù):** Xe tuk-tuk (`img_004.jpg`) và auto-rickshaw (`img_035.jpg`) được xếp vào lớp `other_vehicle`.
- **R-06 (Hình ảnh nhân tạo / Phi thực thể sống - KHÔNG GÁN):**
  - Tượng đài đá/đồng (`img_017.jpg`, `img_027.jpg`): KHÔNG gán nhãn `person` hay phương tiện.
  - Biển quảng cáo, áp phích in 2D (`img_018.jpg`): KHÔNG gán nhãn.
  - Ma-nơ-canh cửa hàng thời trang (`img_033.jpg`): KHÔNG gán nhãn `person`.
  - Mô hình / đồ chơi trong tủ kính (`img_038.jpg`): KHÔNG gán nhãn.
  - Ảnh phản chiếu qua gương chiếu hậu (`img_020.jpg`): KHÔNG gán nhãn đối tượng trong mặt gương (là ảnh ảo).
- **R-07 (Người hóa trang / Trang phục đặc biệt):** Người thật mặc trang phục hóa trang (Predator trong `img_011.jpg`) vẫn gán `person` vì là con người di chuyển thực tế.
- **R-08 (Hàng hóa cồng kềnh):** Hàng hóa chở trên xe máy (`img_029.jpg`) hoặc xe tải: Box xe ôm sát thân xe và người, không mở rộng quá mức bao trọn kiện hàng cồng kềnh bất thường nếu kiện hàng làm méo mó hình dáng phương tiện.
- **R-09 (Ngưỡng kích thước tối thiểu và ignore_region):** Vật thể có chiều cao $< 20\text{px}$ nếu đứng riêng lẻ và mờ thì KHÔNG gán; nếu nằm trong cụm đông đúc thì bao lại bằng `ignore_region`.
- **R-10 (Làm rõ Van vs Car):** Xe có khoang chở khách/hàng nguyên khối dạng hình hộp (minivan, xe giao hàng kiểu Ford Transit / Toyota Hiace) xếp vào `van`. Xe SUV, Crossover, Sedan cá nhân thông thường dù 7 chỗ vẫn xếp vào `car`.
- **R-11 (Chuẩn hóa Truncated):** Thuộc tính `truncated=yes` chỉ áp dụng khi box tiếp xúc trực tiếp mép ảnh (x=0, y=0, x2=width, y2=height) và bị cắt cụt. Nếu vật trọn vẹn trong ảnh mà bị che bởi vật khác thì gán `occluded_level`, không gán `truncated=yes`.
- **R-12 (Ưu tiên Box cá nhân so với Ignore):** Khi một annotator vẽ box cá thể rõ ràng và annotator kia phủ vùng `ignore_region`, nếu vật thể có chiều cao $\ge 20\text{px}$ và nhận diện rõ thì ưu tiên giữ box cá thể và thu hẹp vùng ignore.

---

## 5. Bảng xử lý Ca khó tiêu biểu từ 40 ảnh

| Mã ảnh | Tình huống | Mã Rule | Quyết định chốt của nhóm | Lý do kỹ thuật |
|---|---|---|---|---|
| `img_001.jpg` | Xe bán tải đỗ cạnh quán ăn tối. | `R-01` | Gán `truck` (có thùng hở riêng), `occluded_level = 0`. | Phân loại rõ bán tải thùng hở thuộc `truck`. |
| `img_002.jpg` | Xe cõng xe (Renault 380 car carrier). | `R-04` | 1 box `truck` cho xe tải; từng box `car` cho các xe con trên sàn. | Camera xe tự lái cần phát hiện các ô tô con dù đang nằm trên sàn xe khác. |
| `img_003.jpg` | Người bán hàng rong bằng xe đạp tại Hà Nội. | `R-02` | 1 box `bicycle` ôm cả xe đạp và người bán hàng. | Áp dụng quy tắc gộp người và xe đạp theo thống nhất của nhóm. |
| `img_004.jpg` | Hai xe tuk-tuk màu xanh đỗ trên vỉa hè. | `R-05` | Gán `other_vehicle`, tick `can_review = false`. | Xếp vào lớp phương tiện đặc thù theo taxonomy mới. |
| `img_005.jpg` | Đám đông xe máy chen chúc ở TP.HCM. | `R-02` | Mỗi xe máy có người vẽ đúng 1 box `motorcycle`. | Gộp cả người và xe giúp đơn giản hóa số lượng box và bao trọn khối chướng ngại vật. |
| `img_006.jpg` | Xích lô du lịch tại Huế chở khách. | `R-05` | Khung xích lô gán `other_vehicle` (hoặc chỉ gán `person` cho người ngồi nếu phân vân). | Xích lô chở khách là phương tiện đặc thù đường phố. |
| `img_011.jpg` | Người hóa trang quái vật Predator. | `R-07` | Gán `person`, `occluded_level = 0`. | Con người thật có khả năng cử động trên đường phố. |
| `img_017.jpg` | Tượng Duke of Wellington cưỡi ngựa. | `R-06` | KHÔNG gán `person`. | Tượng điêu khắc cố định, tránh lỗi phanh khẩn cấp vô cớ. |
| `img_018.jpg` | Biển quảng cáo tranh cử in chân dung. | `R-06` | KHÔNG gán `person`. | Hình ảnh in 2D phẳng, không có thể tích vật lý 3D. |
| `img_020.jpg` | Hình ảnh cầu vồng và xe trong gương chiếu hậu. | `R-06` | KHÔNG gán nhãn vật trong gương. | Ảnh ảo quang học, không tồn tại ở vị trí thực địa của gương. |
| `img_022.jpg` | Xe máy có thùng bên (sidecar). | `R-02` | 1 box `motorcycle` ôm trọn cả thân xe, thùng bên và người ngồi. | Khối phương tiện di chuyển thống nhất. |
| `img_026.jpg` | Hai người cùng ngồi trên 1 xe máy trong hầm. | `R-02` | Vẽ đúng 1 box `motorcycle` bao trọn cả xe máy và cả 2 người. | Quy tắc gộp người vào xe máy theo thống nhất nhóm. |
| `img_027.jpg` | Tượng đài xe bán tải bằng đá ở nghĩa trang. | `R-06` | KHÔNG gán `truck`. | Tác phẩm điêu khắc bằng đá cố định, không có động cơ. |
| `img_033.jpg` | Ma-nơ-canh trưng bày quần áo sau cửa kính. | `R-06` | KHÔNG gán `person`. | Hình nộm tĩnh trong nhà, không tham gia giao thông. |
| `img_035.jpg` | Xe Auto-rickshaw chở quá tải tại Ấn Độ. | `R-05` | Gán `other_vehicle`, bao trọn xe và người. | Xe 3 bánh cơ giới đặc thù. |
| `img_038.jpg` | Đồ chơi mô hình ô tô trong tủ kính Munich. | `R-06` | KHÔNG gán `car`. | Mô hình tĩnh trong phòng trưng bày. |

---

## 6. Lịch sử Phiên bản

| Phiên bản | Ngày | Người duyệt | Nội dung thay đổi chính |
|---|---|---|---|
| `v1.0` | 2026-11-16 | Cả nhóm (038, 039) | Thống nhất ý kiến của Team: Chốt gộp người vào xe máy (`motorcycle`) và xe đạp (`bicycle`); chỉ vẽ box phần nhìn thấy (visible); bổ sung các lớp `bicycle`, `bus`, `other_vehicle`, `van` vào Taxonomy v1.0; thiết lập 9 quy tắc Rule ID và giải quyết 16 ca khó. Khóa v1.0 để bước vào Pha 3 gán nhãn độc lập. |
| `v1.1` | 2026-11-16 | Cả nhóm (038, 039) | Hiệu chỉnh sau Calibration: Làm rõ quy tắc gộp người vào `motorcycle`/`bicycle` (R-02) giải quyết 19 ca lệch lớp; bổ sung quy tắc phân biệt `van` vs `car` (R-10); chuẩn hóa thuộc tính `truncated` (R-11); và quy chuẩn ưu tiên box cá thể so với `ignore_region` (R-12). |
