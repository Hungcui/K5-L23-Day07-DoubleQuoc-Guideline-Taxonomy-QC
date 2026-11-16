# Guideline gán nhãn — v0.1 — Mã 038

- Phiên bản: v0.1 (Bản cá nhân khởi tạo độc lập)
- Ngày: 2026-11-16
- Mã thành viên lab: 038
- Họ và tên: Nguyễn Quốc Hưng
- Mã học viên: 2A202604038
- Nhóm: DoubleQuoc
- Công cụ áp dụng: CVAT, chuẩn xuất COCO 1.0

---

## 1. Mục tiêu

- **Mục đích sử dụng:** Bộ dữ liệu nhãn bounding box 2D dùng để huấn luyện mô hình thị giác máy tính nhận diện người và phương tiện giao thông trên đường phố (phục vụ xe tự lái / trợ lái ADAS và hệ thống camera giám sát giao thông thông minh ITS).
- **Hệ quả của lỗi gán nhãn:**
  - *Bỏ sót (False Negative):* Bỏ sót người đi bộ hoặc xe máy là lỗi chí mạng, có thể dẫn đến tai nạn đâm va trực tiếp gây thương tích hoặc tử vong.
  - *Gán thừa / Nhầm lẫn (False Positive):* Gán nhầm tượng đài, ma-nơ-canh, hình in 2D trên biển quảng cáo hoặc bóng phản chiếu qua gương thành người thật sẽ khiến xe tự lái phanh khẩn cấp đột ngột (phantom braking), gây nguy hiểm cho phương tiện chạy phía sau và ùn tắc giao thông.
  - *Nhầm lẫn giữa các lớp phương tiện:* Nhầm `car` và `truck` gây sai lệch trong việc phân bổ luồng đường, ước tính tải trọng và tính toán khoảng cách an toàn.

---

## 2. Lớp và thuộc tính

### 2.1. Phân loại lớp (Classes)

Mỗi đối tượng hợp lệ chỉ thuộc đúng một lớp duy nhất:

| Lớp | Định nghĩa (một câu) | Ví dụ thực tế | Phản ví dụ / Dễ nhầm với |
|---|---|---|---|
| `car` | Phương tiện cơ giới chở người từ 4 bánh trở lên cỡ nhỏ và vừa (sedan, hatchback, SUV, coupe, taxi). | Ô tô 4 chỗ, SUV gia đình, xe taxi trong `img_040.jpg`. | Dễ nhầm với `van` (xe thùng kín chở hàng/chở khách đông) hoặc `truck` (xe bán tải thùng hở riêng). |
| `truck` | Xe cơ giới chuyên dụng chở hàng có kết cấu cabin và thùng/sàn hàng hóa tách biệt rõ rệt. | Xe tải thùng `img_001.jpg`, xe cõng xe `img_002.jpg`, xe bán tải (pickup truck). | Dễ nhầm với `van` (thân xe nguyên khối liền cabin) hoặc `car` (xe bán tải đóng nắp thùng kín). |
| `van` | Xe tải nhẹ hoặc xe khách thương mại có thân xe hình hộp nguyên khối, khoang chở hàng/người liền mạch với cabin. | Xe minivan, xe cứu thương, xe chuyển phát nhanh thùng liền, xe khách nhỏ. | Dễ nhầm với xe SUV gia đình (`car`) hoặc xe tải nhỏ có thùng rời (`truck`). |
| `motorcycle` | Phương tiện cơ giới 2 hoặc 3 bánh vận hành bằng động cơ (xăng hoặc điện) dùng để chở người/hàng. | Xe máy số, xe tay ga `img_005.jpg`, `img_026.jpg`, xe phân khối lớn `img_011.jpg`. | Dễ nhầm với xe đạp / xích lô thô sơ (không có động cơ -> không gán). |
| `person` | Con người còn sống thực tế đang đi bộ, đứng, ngồi, hoặc điều khiển/ngồi trên phương tiện. | Người đi bộ qua đường `img_014.jpg`, người lái/ngồi sau xe máy `img_005.jpg`, `img_026.jpg`. | Tuyệt đối KHÔNG gán cho tượng đài (`img_017.jpg`), áp phích (`img_018.jpg`), ma-nơ-canh (`img_033.jpg`). |
| `ignore_region` | Vùng ảnh chứa đám đông hoặc cụm xe cộ dày đặc, ở xa hoặc bị mờ nhòe không thể tách rời từng vật thể đơn lẻ. | Vùng xe cộ/người đi bộ li ti ở cuối con đường hoặc trong bóng tối mờ mịt. | Không dùng để thay thế cho vật thể nhìn rõ nhưng người gán nhãn lười vẽ. |

**Quy tắc đối với vật thể không thuộc phạm vi các lớp trên:**
- *Phương tiện thô sơ không động cơ (Xe đạp, xích lô, xe kéo tay, xe bán hàng rong):* KHÔNG gán nhãn cho phương tiện thô sơ; chỉ gán box `person` cho người điều khiển / hành khách nếu nhận diện rõ.
- *Động vật (chó, mèo, gia súc, chim bồ câu):* KHÔNG gán nhãn.
- *Hàng hóa rời, đồ vật tĩnh trên vỉa hè, biển báo giao thông:* KHÔNG gán nhãn.

---

### 2.2. Thuộc tính (Attributes)

| Tên thuộc tính | Kiểu dữ liệu | Giá trị cố định | Định nghĩa & Ngưỡng định lượng cụ thể |
|---|---|---|---|
| `occluded_level` | select | `0` | Thấy rõ toàn bộ: Phần nhìn thấy được $\ge 80\%$ diện tích bề mặt thực tế của vật thể. |
| `occluded_level` | select | `1` | Bị che khuất nhẹ: Phần nhìn thấy được từ $50\%$ đến dưới $80\%$. |
| `occluded_level` | select | `2` | Bị che khuất nặng: Phần nhìn thấy được từ $20\%$ đến dưới $50\%$ (vẫn nhận dạng được hình dáng đặc trưng). |
| `occluded_level` | select | `3` | Bị che khuất cực nặng: Phần nhìn thấy được $< 20\%$ (chỉ nhận ra nhờ một chi tiết nhỏ hoặc ngữ cảnh lân cận). |
| `truncated` | select | `no` | Vật thể nằm trọn vẹn trong khung hình, không chạm hoặc bị cắt bởi bất kỳ mép ảnh nào. |
| `truncated` | select | `yes` | Bất kỳ phần nào của thân xe hoặc cơ thể người bị cắt cụt bởi 1 trong 4 biên ảnh. |
| `can_review` | checkbox | `true` / `false` | Đánh dấu tick (`true`) khi gặp trường hợp mơ hồ (edge case) cần đưa ra thảo luận nhóm. Mặc định là `false`. |

---

## 3. Quy tắc vẽ Bounding Box

1. **Box ôm sát (Tightness):** Bounding box 2D hình chữ nhật phải tiếp xúc chính xác với các điểm cực trị (biên ngoài cùng) của phần nhìn thấy được. Không để lọt khoảng trống nền đường quá mức, không cắt lẹm vào chi tiết của đối tượng.
2. **Xử lý che khuất (Occlusion Handling):** Box chỉ bao quanh **phần thực sự nhìn thấy (Visible area)** của vật thể, tuyệt đối không suy đoán hay vẽ mở rộng xuyên qua vật che chắn phía trước.
3. **Xử lý cắt mép ảnh (Truncation Handling):** Kéo box sát kịch mép ảnh tại vị trí vật bị cắt đứt. Đặt `truncated = yes`.
4. **Tách biệt Người và Phương tiện (Person on Vehicle):**
   - Không bao giờ gộp chung người và xe vào 1 box.
   - Luôn tách biệt: 1 box `person` ôm trọn người (từ mũ bảo hiểm/đầu đến mông/chân người) và 1 box `motorcycle` ôm trọn thân xe và bánh xe.
   - Nếu xe máy chở 2 người (`img_026.jpg`): Vẽ 2 box `person` riêng biệt và 1 box `motorcycle`.
5. **Xe chở trên phương tiện khác (Car carrier):**
   - Phương tiện vận tải (xe tải cõng xe `img_002.jpg`): Vẽ 1 box `truck` bao toàn bộ thân xe tải.
   - Các xe ô tô con được chở trên sàn: Vẽ riêng từng box `car` cho từng chiếc xe nếu nhìn rõ hình dáng phương tiện.
6. **Hình ảnh giả mạo / Không phải thực thể sống:**
   - Tượng đài (`img_017.jpg`, `img_027.jpg`): KHÔNG gán nhãn `person` hay phương tiện.
   - Tranh ảnh, áp phích, biển quảng cáo in 2D (`img_018.jpg`): KHÔNG gán nhãn.
   - Ma-nơ-canh cửa hàng thời trang (`img_033.jpg`): KHÔNG gán nhãn `person`.
   - Đồ chơi, mô hình thu nhỏ trong tủ kính trưng bày (`img_038.jpg`): KHÔNG gán nhãn.
   - Hình ảnh phản chiếu qua gương chiếu hậu (`img_020.jpg`): KHÔNG gán nhãn đối tượng trong mặt gương (là ảnh ảo).
7. **Ngưỡng kích thước tối thiểu:**
   - Chiều cao box $< 20\text{px}$: Nếu đứng đơn lẻ và không nhận diện được rõ ràng -> BỎ QUA không gán. Nếu nằm trong cụm đông đúc -> Gom vào `ignore_region`.

---

## 4. Danh mục xử lý Ca khó (Edge Cases từ 40 ảnh thực tế)

| Ảnh | Tình huống cụ thể | Rule ID | Quyết định gán nhãn | Lập luận logic |
|---|---|---|---|---|
| `img_002.jpg` | Xe tải chuyên dụng chở nhiều ô tô con trên sàn (Car carrier). | `R-01` | Vẽ 1 box `truck` cho xe tải. Vẽ từng box `car` riêng cho các xe con trên sàn. | Camera xe tự lái cần nhận diện các ô tô con dù đang nằm trên sàn xe khác để tránh nhầm lẫn chướng ngại vật phía trước. |
| `img_004.jpg` | Xe Tuk-tuk 3 bánh chở khách đỗ trên vỉa hè. | `R-02` | Tạm gán `car`, tick `can_review = true`. | Tuk-tuk là phương tiện chở khách 3 bánh có cabin. Taxonomy v0 chưa có lớp riêng, tạm xếp vào `car` và cắm cờ để họp nhóm chốt ở Pha 2. |
| `img_005.jpg` | Dòng xe máy đông đúc ở TP.HCM, người và xe đan xen. | `R-03` | Tách riêng từng box `person` và `motorcycle`. | Tách biệt người và xe giúp mô hình dự đoán quỹ đạo người ngã hoặc bước xuống xe độc lập. |
| `img_006.jpg` | Xích lô du lịch tại Huế chở khách. | `R-04` | Gán `person` cho người lái và du khách. KHÔNG gán box phương tiện cho khung xích lô. | Xích lô là xe thô sơ đạp chân, không có động cơ nên không thuộc `motorcycle`, `car`, `truck` hay `van`. |
| `img_011.jpg` | Người mặc trang phục hóa trang quái vật Predator bên cạnh xe máy. | `R-05` | Gán `person` cho nhân vật Predator. | Dù mang đồ hóa trang, đây là con người thật có khả năng di chuyển và gây nguy cơ va chạm giao thông. |
| `img_017.jpg` | Tượng đài Duke of Wellington cưỡi ngựa đội nón chóp giao thông. | `R-06` | KHÔNG gán `person`. | Tượng điêu khắc bằng đá/kim loại cố định, không phải con người tham gia giao thông. Gán nhầm sẽ gây phanh khẩn cấp vô cớ. |
| `img_018.jpg` | Biển quảng cáo tranh cử ngoài trời in ảnh chân dung người khổ lớn. | `R-07` | KHÔNG gán `person`. | Ảnh in 2D phẳng trên tấm bảng, không có thể tích 3D và không cử động được. |
| `img_020.jpg` | Hình ảnh xe cộ và cầu vồng phản chiếu trong gương chiếu hậu ô tô. | `R-08` | KHÔNG gán nhãn các đối tượng trong mặt gương. | Đối tượng trong gương là ảnh ảo quang học, không phản ánh thực thể vật lý tại tọa độ đó. |
| `img_022.jpg` | Xe máy cổ điển có thùng bên chở người (Motorcycle with sidecar). | `R-09` | 1 box `motorcycle` ôm trọn cả thân xe và thùng bên; các box `person` riêng cho từng người. | Xe máy và thùng bên gắn liền thành một khối di chuyển cùng vận tốc và hướng lái. |
| `img_026.jpg` | Cặp đôi nam nữ cùng ngồi trên 1 chiếc xe máy trong hầm chui. | `R-10` | 2 box `person` riêng biệt cho 2 người; 1 box `motorcycle` cho chiếc xe máy. | Đảm bảo đếm chính xác số lượng người tham gia giao thông. |
| `img_027.jpg` | Tượng đài tạc hình xe bán tải (Pickup truck monument) bằng đá trong nghĩa trang. | `R-11` | KHÔNG gán `truck`. | Tác phẩm điêu khắc bằng đá cố định, không có động cơ và không tham gia giao thông. |
| `img_033.jpg` | Ma-nơ-canh trưng bày quần áo đứng trong cửa hàng thời trang. | `R-12` | KHÔNG gán `person`. | Ma-nơ-canh là tượng nhựa tĩnh trong nhà, gán nhầm sẽ gây nhiễu cho mô hình phát hiện người đi bộ. |
| `img_038.jpg` | Mô hình xe ô tô và búp bê đồ chơi trưng bày sau tủ kính. | `R-13` | KHÔNG gán nhãn `car` hay `person`. | Đồ chơi kích thước siêu nhỏ trong phòng trưng bày không thuộc phạm trù giám sát giao thông thực địa. |

---

## 5. Kiểm soát chất lượng (QC) và Phiên bản

### 5.1. Tiêu chí tự kiểm soát (Self-QC)
- Đảm bảo 100% box ôm sát đường viền nhìn thấy của vật thể, không có box lửng lơ hoặc vẽ lệch.
- Đạt ngưỡng IoU kiểm thử: $\ge 0.85$ cho vật thể lớn ($\ge 50\text{px}$) và $\ge 0.70$ cho vật thể nhỏ ($< 50\text{px}$).
- Tự kiểm tra không bỏ sót bất kỳ người đi bộ hoặc xe máy rõ ràng nào trong ảnh được phân công.

### 5.2. Giải quyết ca bất đồng & mơ hồ
- Khi gặp ca nghi ngờ không nằm trong 13 quy tắc trên: Bắt buộc đánh dấu `can_review = true`.
- Thu thập danh sách ca `can_review` để mang ra đối chiếu với thành viên 039 ở Pha 2.

### 5.3. Lịch sử thay đổi (Changelog)

| Phiên bản | Ngày | Người thực hiện | Tóm tắt thay đổi | Căn cứ ca cụ thể |
|---|---|---|---|---|
| `v0.1` | 2026-11-16 | Thành viên 038 | Khởi tạo Guideline cá nhân độc lập dựa trên khung v0 starter. Quy định chi tiết ranh giới lớp, ngưỡng che khuất 4 mức và 13 quy tắc ca khó có dẫn chứng ID ảnh thật. | Khảo sát toàn bộ 40 ảnh `data/images/img_001.jpg` đến `img_040.jpg`. |
