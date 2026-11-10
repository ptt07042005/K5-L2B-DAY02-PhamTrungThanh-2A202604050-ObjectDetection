# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Các dòng xe ô tô cá nhân chở người cỡ nhỏ và vừa (4–7 chỗ), bao gồm cả xe đang lưu thông hoặc đỗ bên đường. | Sedan, hatchback, SUV, crossover, taxi, xe bán tải cabin nhỏ (phân biệt với `van` có thân hình hộp cao và `truck` có thùng chở hàng lớn). |
| `truck` | Xe tải chuyên dụng dùng để chở hàng hóa tải trọng vừa và lớn, có phần cabin lái tách biệt rõ với thùng hàng phía sau. | Xe tải thùng kín, xe tải thùng bạt/thùng lửng, xe đầu kéo (không gán cho xe bán tải nhỏ thuộc `car` hoặc xe tải van nhỏ thuộc `van`). |
| `bus` | Phương tiện giao thông công cộng hoặc xe khách cỡ lớn/trung bình chuyên chở nhiều hành khách. | Xe buýt nội đô, xe khách đường dài, minibus chở khách có thân dài và dãy cửa kính liên tiếp dọc hai bên sườn. |
| `van` | Xe thân liền khối dáng hình hộp chữ nhật cao, dùng để chở người (MPV cỡ lớn/minivan) hoặc chở hàng nhẹ trong đô thị. | Minivan, xe 16 chỗ dáng hộp (như Ford Transit), xe tải van kín nhỏ (cao và vuông hơn `car`, nhưng ngắn hơn `bus` và không có thùng rời như `truck`). |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: 4 cạnh của khung chữ nhật chạm sát vào các điểm cực trị nhìn thấy được của thân xe (nóc xe, bánh xe/gầm xe, mũi xe, đuôi xe và gương chiếu hậu), không kéo dư khoảng trống mặt đường hoặc bao gồm vệt bóng đổ, vệt nhòe chuyển động (motion blur).
- Vật thể bị che khuất: Nếu phương tiện bị cột đèn, biển báo, bụi cây hoặc xe khác che một phần nhưng phần lộ ra vẫn đủ nhận diện rõ loại xe thì vẽ 1 khung duy nhất bao quanh phần nhìn thấy/hình khối của xe đó (không chia một xe thành 2 khung rời rạc; chấp nhận các khung gối lên nhau khi các xe đỗ nối đuôi).
- Vật thể bị cắt bởi rìa ảnh: Chỉ vẽ khung bao trọn phần diện tích xe còn nằm trong phạm vi bức ảnh (cạnh khung chạm sát mép ảnh) nếu phần hiện diện còn đủ đặc trưng để xác định đúng lớp; bỏ qua nếu chỉ lộ một mẩu quá nhỏ ở mép ảnh không rõ loại xe.
- Vật thể rất nhỏ/mơ hồ: Phóng to tối đa để quan sát các xe ở góc xa phía chân trời; vẫn gán nhãn nếu mắt thường còn phân biệt được hình khối phương tiện (`car`, `bus`, `truck`, `van`), không gán đoán mò cho các cụm điểm ảnh quá mờ nhòe hòa lẫn vào nền đường.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_001.jpg` | Chiếc xe màu trắng đang chạy ở làn bên phải bị thân cột đèn chiếu sáng ở tiền cảnh che cắt ngang dọc giữa thân xe. | Vẽ 1 khung `car` duy nhất bao trọn từ phần đầu xe bên trái cột đèn sang hết phần đuôi xe bên phải cột đèn. | Cả hai nửa thuộc cùng một phương tiện thực tế và phần thân/đuôi xe lộ ra đủ rõ để xác định chắc chắn là SUV/crossover thuộc lớp `car`, không tách thành 2 đối tượng giả. |
| `drive_001.jpg` | Hàng xe đỗ sát nhau dọc làn phụ bên trái (cạnh bồn cây và trước dãy nhà mái đỏ), xe đứng trước và cột biển báo che khuất một phần mũi/đuôi xe đứng sau. | Vẽ khung `car` tách biệt cho từng chiếc xe (cho phép cạnh khung của hai xe liền kề chồng lấn nhẹ lên nhau), kể cả xe đang đỗ tĩnh. | Schema quy định phân loại theo bản chất vật thể chứ không theo trạng thái di chuyển (xe đang đỗ vẫn là `car`), và phần thân nhìn thấy của từng xe vẫn tách bạch rõ số lượng cũng như loại xe. |
| `drive_001.jpg` | Chiếc xe màu bạc ở gần camera nhất (góc dưới ảnh) bị nhòe chuyển động (motion blur) quanh viền và các xe ở rất xa phía trên cùng góc phải ảnh. | Với xe bạc gần camera: thu hẹp khung bám sát khung thân xe thực tế, bỏ phần bóng nhòe dư ra mặt đường; với các xe ở xa: phóng to để gán `car` cho xe đỏ và `bus` cho xe buýt ở cuối đường khi còn rõ khối. | Đảm bảo độ khít (tightness/IoU cao) cho vật thể lớn ở gần, đồng thời giữ Recall tốt cho các phương tiện ở xa còn đủ dấu hiệu nhận biết lớp theo đúng định nghĩa schema. |