# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Phạm Trung Thành
- MSSV: 2A202604050
- Email: 26ai.thanhpt@vinuni.edu.vn
- Lớp: K5-L2B
- Link repo bài làm: https://github.com/ptt07042005/K5-L2B-DAY02-PhamTrungThanh-2A202604050-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/1u_Vp3iElQtTENz75cWtlejsCfiQjfqVT?usp=sharing

## 2. Schema và mini guideline

Chi tiết quy tắc gán nhãn cá nhân xem tại [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).

- **Cách áp dụng schema:** Phân loại 4 lớp phương tiện (`0: car`, `1: truck`, `2: bus`, `3: van`) dựa trên hình thái cấu trúc xe thực tế. Khung bao (bounding box) được kéo bám sát 4 điểm cực trị nhìn thấy được của thân xe, loại bỏ bóng đổ mặt đường và vệt nhòe chuyển động. Chỉ gán nhãn cho các phương tiện đang lưu thông trong khu vực lòng đường chính; bỏ qua các phương tiện đỗ tĩnh ở bãi đỗ/vỉa hè hoặc các xe quá nhỏ ở sát đường chân trời (vùng Ignore của bộ dữ liệu gốc UA-DETRAC).
- **Tóm tắt 3 ca mơ hồ đã quyết định:**
  - `drive_001.jpg` (Xe bị che khuất bởi cột đèn): Chiếc SUV màu trắng ở làn phải bị thân cột đèn cắt dọc giữa thân xe. Quyết định vẽ 1 box `car` duy nhất bao trọn cả phần đầu và đuôi xe thay vì tách làm 2 box rời, vì hai phần cùng thuộc một thực thể và vẫn nhận diện rõ loại xe.
  - `drive_002.jpg` (Phân biệt `car` và `van` cho xe dáng hộp cỡ nhỏ): Chiếc xe màu trắng có giá nóc đi bên trái xe buýt trắng lớn có kích thước tương đương xe con nhưng thân liền khối dáng hộp cao. Quyết định gán nhãn `van` thay vì `car` để bám sát đặc trưng hình học của lớp minivan/xe tải van nhỏ.
  - `drive_003.jpg` (Phương tiện bị cắt bởi rìa ảnh): Cụm xe buýt lớn và xe ở sát mép phải màn hình bị cắt khuất một phần thân. Quyết định vẫn giữ và vẽ box chạm sát mép phải ảnh vì phần diện tích hiện diện còn đủ lớn để phân biệt rõ đặc trưng của `bus` và `car`.
- **Phần cần giảng viên làm rõ thêm:** Ranh giới chính xác của vùng loại trừ (Ignore region) theo khoảng cách xa gần tại các nút giao thông lớn (như dưới gầm cầu vượt hoặc phía bên kia vạch kẻ sang đường ở `drive_002`, `drive_003`), vì bằng mắt thường người gán vẫn nhìn thấy xe nhưng trong ground truth gốc các vùng xa này không được tính nhãn, dễ phát sinh False Positive nếu không biết trước vùng ROI của từng góc camera.

## 3. Annotation và self-QC

- Số file nhãn train: 50/50 file `.txt` trong `dataset/labels/train/`.
- Kết quả checker và lỗi đã xử lý: Chạy `python tools/check_yolo_labels.py` đạt kết quả `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ`, không phát sinh lỗi sai định dạng YOLO, không có tọa độ vượt ngoài khoảng `[0, 1]` hay box rỗng diện tích bằng 0.
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_001.jpg`, `drive_002.jpg`, `drive_003.jpg`, `drive_006.jpg`, `drive_007.jpg`, `drive_008.jpg`, `drive_009.jpg`, `drive_028.jpg`, `drive_031.jpg`, `drive_038.jpg`, `drive_043.jpg`, `drive_049.jpg`, `drive_050.jpg`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:
  - Phát hiện gán dư nhiều xe đỗ tĩnh trên vỉa hè/làn phụ và xe quá nhỏ ở xa (`drive_001`, `drive_002`, `drive_003`, `drive_049`) -> Xóa các box nằm ngoài luồng giao thông chính.
  - Phát hiện nhầm lẫn lớp giữa `car` với `van` và `truck` (ví dụ xe minivan bạc ở giữa ngã tư trong `drive_008`, xe tải thùng kín màu xanh lá trong `drive_009` ban đầu gán nhầm là `car`) -> Đổi lại đúng class ID trên CVAT.
  - Phát hiện bỏ sót một số xe bị kẹp giữa các xe lớn hoặc sát rìa phải (`drive_003`, `drive_009`) -> Bổ sung box cho các xe còn nhìn rõ hình khối trên làn chính.
- Commit khóa nhãn ban đầu: `339dfbb` (`339dfbbd1d64b471c1bf7239c50118590d5e1b24` — thực hiện rà lỗi và chỉnh nhãn cục bộ trên máy trước khi push lên repo cá nhân; toàn bộ số liệu đối chiếu trước và sau khi sửa được ghi nhận tại Mục 4).

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 480 | 432 |
| FP | 673 | 270 |
| FN | 173 | 221 |
| Precision | 0.4163 | 0.6154 |
| Recall | 0.7351 | 0.6616 |
| F1 | 0.5316 | 0.6376 |
| mean_iou_of_matches | 0.8389 | 0.8453 |
| passed | false | false |

Kết quả đánh giá chi tiết được lưu tại [outputs/annotation_evaluation.json](../../outputs/annotation_evaluation.json).

**Nhật ký kiểm tra và quyết định Rework trên các ảnh tiêu biểu:**
- `drive_001`: Ban đầu vẽ 16 box (dư 11 box xe đỗ dọc lề đường phụ bên trái và các chấm xe ở chân trời). Quyết định xóa các box ngoài vùng giao thông chính, giữ nguyên 5 box `car` đang di chuyển trên trục đường chính -> Khớp tuyệt đối 5/5 với Gold.
- `drive_002`: Ban đầu vẽ 47 box bao gồm cả đám đông xe chờ đèn đỏ phía xa bên kia ngã tư. Quyết định xóa các box nhỏ phía trên vạch kẻ sang đường, giữ lại 13 phương tiện rõ nét ở nửa dưới ngã tư và rà lại nhãn lớp cho xe dáng hộp cạnh xe buýt (đạt 10/13 matches).
- `drive_003`: Ban đầu vẽ 45 box tràn lên khu vực kẹt xe dưới gầm cầu vượt đi bộ. Quyết định xóa cụm xe nhỏ dưới chân cầu vượt, giữ lại đoàn xe đang rẽ ở trung tâm nút giao và bổ sung lại cụm phương tiện bị cắt một phần ở sát mép phải màn hình (đạt 16/18 matches).
- `drive_006`, `drive_008` & `drive_009`: Lọc bớt các box ở vùng xa sát mép trên (như xe đỗ sau hàng rào góc trên `drive_009`) và kiểm tra lại nhãn lớp cho nhóm xe thương mại (`van`, `truck`).
- Một số ảnh mật độ dày như `drive_024`, `drive_027` trong lúc lọc bỏ vùng xa đã xóa hơi mạnh tay vào các xe tầm trung khiến FN tăng từ 173 lên 221; tuy nhiên tổng lượng FP toàn tập đã giảm sâu từ 673 xuống 270 (giảm 403 box thừa, giúp Precision tăng từ 0.4163 lên 0.6154, F1 tăng từ 0.5316 lên 0.6376) và độ khít khung `mean_iou_of_matches` duy trì ở mức cao (0.8453).

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab GPU Tesla T4 (CUDA:0, 14913MiB), Python `3.13.16`, Ultralytics `8.4.208`, PyTorch `2.14.1+cu130`.
- Các tham số thay đổi so với notebook nếu có và lý do: Giữ nguyên cấu hình mặc định của notebook (mô hình `yolo26n.pt`, số epochs, kích thước ảnh và cấu hình `data.yaml`) để đánh giá khách quan chất lượng bộ nhãn 50 ảnh train.
- Đường dẫn best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.3156 |
| Recall | 0.3066 |
| mAP50 | 0.2740 |
| mAP50–95 | 0.2006 |

Thư mục ảnh dự đoán và log huấn luyện đính kèm tại: [assets/eval_predictions/](./assets/eval_predictions/) và [assets/yolo26_runs/](./assets/yolo26_runs/).
- **Nhận xét trên ảnh dự đoán cụ thể:** Quan sát trực tiếp trong `./assets/eval_predictions/`:
  - Lớp đa số `car` (chiếm 94/113 thực thể trong tập eval) đạt kết quả khá tốt (`Precision = 0.629`, `Recall = 0.777`, `mAP50 = 0.738`), thể hiện rõ trên ảnh `drive_058.jpg` và `drive_059.jpg` khi hầu hết các xe con ở làn gần và trung bình đều được khoanh đúng với độ tự tin cao.
  - Tuy nhiên ở các lớp ít mẫu hơn, trên ảnh `drive_056.jpg`, hai chiếc xe tải thùng ở tiền cảnh (xe thùng xanh ngọc ở làn trái và xe thùng đỏ ở làn phải) hoặc bị bỏ sót, hoặc bị nhận diện lệch lớp; trên ảnh `drive_058.jpg`, chiếc xe buýt màu vàng ở làn ngoài cùng bên trái có độ tự tin rất thấp nên không vượt qua ngưỡng đánh giá chuẩn của lớp `bus`.

**Trả lời bốn câu hỏi cuối notebook:**

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Dựa trên bảng kết quả chi tiết từng lớp, lớp **`bus`** có Recall thấp nhất (`R = 0.000`, kế tiếp là `truck` với `R = 0.200` và `van` với `R = 0.250`), trong khi lớp `car` đạt `R = 0.777`. Nguyên nhân chính là sự mất cân bằng dữ liệu mạnh trong tập nhỏ 50 ảnh (lớp `bus` trong tập eval chỉ có đúng 2 thực thể trên 2 ảnh, `truck` có 5 thực thể, so với 94 thực thể của `car`). Ảnh minh họa rõ là `drive_058.jpg` trong `./assets/eval_predictions/` (chiếc xe buýt lớn ở làn trái góc trên chỉ đạt confidence rất thấp hoặc bị lẫn sang lớp khác nên không ghi nhận được True Positive ở ngưỡng đánh giá chuẩn) và `drive_056.jpg` (bỏ sót xe tải thùng màu xanh ở làn trái).
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Cần kiểm tra lại **độ bám sát (tightness) và tính nhất quán của 4 cạnh bounding box** trong dữ liệu gán nhãn (ví dụ ngay ở lớp `car`, `mAP50` đạt `0.738` nhưng `mAP50–95` giảm xuống `0.521`): ở ngưỡng IoU 0.50, box hơi lỏng hoặc hơi hụt vẫn được tính là đúng, nhưng khi xét trung bình qua các ngưỡng khắt khe từ 0.55 đến 0.95, các box kéo dư mặt đường, dính bóng đổ, hoặc ước lượng lệch phần thân xe bị che khuất sẽ làm IoU tụt dưới ngưỡng và kéo mAP50–95 giảm mạnh.
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi nhầm lẫn giữa **`car`, `van` và `truck`** có liên quan trực tiếp tới phần định nghĩa ranh giới lớp trong guideline. Dữ kiện hỗ trợ: trên tập eval, `van` chỉ đạt `mAP50 = 0.115` (`P = 0.243`, `R = 0.250`) và `truck` đạt `mAP50 = 0.235`, kết hợp với quan sát từ bước Gold QC (`drive_008`, `drive_009`, `drive_026` có tổng số box khớp nhưng lệch tên lớp) và confusion matrix trong `./assets/yolo26_runs/`. Các dòng xe MPV 7 chỗ, minivan và xe tải van kín có hình dáng giao thoa nhau khi nhìn từ camera lắp cao; nếu guideline không tách biệt thật chi tiết dấu hiệu hình học thì nhãn huấn luyện thiếu nhất quán, khiến mô hình có xu hướng dự đoán lệch về lớp chiếm đa số là `car`.
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Không dùng 10 ảnh `eval` để train nhằm tránh **rò rỉ dữ liệu (data leakage)** và hiện tượng học thuộc lòng (overfitting): nếu mô hình được cập nhật trọng số trực tiếp trên tập ảnh kiểm tra thì điểm mAP thu được sẽ cao giả tạo, mất khả năng đánh giá độ tổng quát hóa trên bối cảnh giao thông mới. Trong notebook, 10 ảnh `eval` đóng vai trò là tập chuẩn đối chiếu cố định (validation benchmark) để theo dõi quá trình hội tụ sau mỗi epoch, lưu lại checkpoint tốt nhất (`best.pt`) và đo lường công bằng chất lượng bộ nhãn train do học viên tự gán.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Khi bắt đầu vẽ box cho 50 ảnh trên CVAT | Ảnh trích từ camera giao thông có góc nhìn từ trên cao xuống, nhiều xe chạy nhanh bị nhòe viền (`drive_001`) hoặc nối đuôi che khuất nhau ở ngã tư (`drive_002`, `drive_003`), rất dễ kéo box bị rộng ra mặt đường hoặc lưỡng lự việc tách 1 xe bị cột đèn che thành 2 box | Phóng to ảnh để căn sát mép thân xe thật (bỏ qua phần bóng đổ và vệt nhòe mặt đường); thống nhất trong guideline chỉ vẽ 1 box duy nhất bao quanh xe bị cột đèn cắt ngang | Độ khít của các khung vẽ khớp đạt mức cao ngay từ lần đầu (`mean_iou_of_matches = 0.8389`) và giữ ổn định ở mức `0.8453` sau khi chỉnh sửa. |
| Đối chiếu Gold QC lần đầu (dư quá nhiều box ngoài vùng chấm) | Lần đầu em gán nhãn tất cả các xe nhìn thấy trong ảnh kể cả xe đỗ tĩnh trên vỉa hè, bãi xe bên lề và các xe nhỏ li ti ở cuối đường nên bị báo dư tới `673` FP (`Precision = 0.4163`, `F1 = 0.5316`) | Mở lại từng ảnh có độ lệch lớn (`drive_001`, `drive_002`, `drive_003`, `drive_049`...) để lọc xóa các cụm xe đỗ bên lề và xe nằm quá xa phía bên kia ngã tư / dưới gầm cầu vượt | Giảm được `403` box thừa (FP giảm mạnh từ `673` xuống `270`), kéo Precision tăng từ `0.4163` lên `0.6154` và F1 tăng từ `0.5316` lên `0.6376` (một số ảnh như `drive_001`, `drive_013` đã khớp tuyệt đối 100%). |
| Cân đối việc xóa box vùng xa ở các ngã tư đông xe | Khi xóa bớt các xe ở vùng xa để giảm FP thì ở một số ảnh mật độ dày như `drive_024` (còn 17/30 xe), `drive_027` (còn 19/31 xe) hay `drive_015`, `drive_016` em lại xóa hơi quá tay vào các xe tầm trung mà bộ Gold vẫn tính nhãn | Rà lại theo từng cụm ảnh để giữ lại các xe đang lưu thông rõ nét ở giữa nút giao, tránh xóa lan xuống nửa dưới khung hình | Do ranh giới vùng không tính nhãn (Ignore region) ở tầm trung của bộ UA-DETRAC không có đường kẻ cố định trên ảnh nên FN tăng từ `173` lên `221` (Recall giảm nhẹ từ `0.7351` xuống `0.6616`), làm F1 dừng ở `0.6376` và chưa chạm mốc `0.80`. |
| Phân biệt nhãn lớp ở các ảnh có số lượng xe khớp nhưng điểm `matches` thấp | Ở các ảnh như `drive_009` (vẽ 21/21 xe nhưng khớp 8), `drive_014` (14/14 khớp 9) hay `drive_026` (39/39 khớp 29), tổng số box vẽ ra bằng đúng đáp án nhưng vẫn bị tính đồng thời lỗi FP và FN do lệch tên lớp giữa `car`, `van` và `truck` | Kiểm tra lại các xe dáng hộp (MPV/minivan) và xe tải thùng nhỏ trên CVAT để đổi nhãn từ `car` sang `van` hoặc `truck` theo đúng đặc điểm hình dáng | Cải thiện được một phần các xe thương mại nhìn rõ ở gần, tuy nhiên với các xe chụp từ góc lưng hoặc ở xa hơi mờ thì việc phân biệt rạch ròi giữa `car` cỡ lớn (SUV/7 chỗ) và `van` vẫn còn sai số nhất định. |

## 7. Tự đánh giá theo rubric

Đối chiếu với [RUBRIC.md](../../RUBRIC.md):
- **Đã hoàn thành và có minh chứng rõ ràng:**
  - Thiết lập đúng cấu trúc repo cá nhân `K5-L2B-DAY02-PhamTrungThanh-2A202604050-ObjectDetection`, phân quyền công khai và không đẩy nhầm dữ liệu `gold_train_labels` hay trọng số `.pt` nặng lên GitHub.
  - Xây dựng đầy đủ [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md) gồm định nghĩa 4 lớp, quy tắc vẽ box và phân tích 3 tình huống mơ hồ thực tế.
  - Hoàn thành gán nhãn đủ 50/50 ảnh train trên CVAT, vượt qua kiểm tra định dạng `check_yolo_labels.py` (50/50 hợp lệ) và thực hiện đối chiếu Gold QC + Rework có lưu vết số liệu trước/sau rõ ràng (`mean_iou_of_matches` đạt `0.8453` > `0.70`).
  - Chạy hoàn tất pipeline huấn luyện và đánh giá mô hình `YOLO26n` trên Google Colab GPU T4, trích xuất đầy đủ `assets/eval_predictions/` và `assets/yolo26_runs/` kèm phân tích cụ thể kết quả theo từng lớp.
- **Phần còn hạn chế / cần cải thiện thêm:**
  - Sau quá trình rework, dù đã cắt giảm `403` lỗi FP để nâng F1 từ `0.5316` lên `0.6376` và giữ độ khít box cao (`0.8453`), bộ nhãn vẫn chưa đạt ngưỡng `F1 >= 0.80` (`passed: false`) do khác biệt về vùng loại trừ tầm trung (`drive_024`, `drive_027`) và độ nhiễu nhãn giữa `car`/`van`. Điều này cũng phản ánh trực tiếp sang kết quả mô hình YOLO khi lớp `car` học tốt (`mAP50 = 0.738`) nhưng các lớp ít mẫu như `bus`, `van`, `truck` còn đạt Recall thấp.

## 8. Công cụ AI và tài liệu tham khảo

- **Công cụ AI đã sử dụng:** Sử dụng Gemini hỗ trợ ở các bước: tra cứu cú pháp lệnh Terminal trên Windows PowerShell, gợi ý cách đọc bảng thống kê `per_image` từ script đánh giá để khoanh vùng nguyên nhân gây tăng False Positive, và hỗ trợ rà soát bố cục trình bày Markdown cho guideline/báo cáo.
- **Phần đã tự thực hiện và tự kiểm tra:** Trực tiếp khởi tạo task trên CVAT, tự tay vẽ và chỉnh sửa bounding box cho 50 ảnh giao thông, tự chạy các script kiểm tra định dạng/chấm điểm trên máy cá nhân, tự cấu hình chạy phiên huấn luyện trên Google Colab và kiểm chứng toàn bộ số liệu, hình ảnh đưa vào báo cáo.
- **Tài liệu tham khảo ngoài repo:**
  - Tài liệu hướng dẫn sử dụng công cụ gán nhãn CVAT (CVAT Documentation - Shape mode & YOLO 1.1 export format).
  - Tài liệu Ultralytics YOLO Documentation về cấu trúc dữ liệu `data.yaml` và các chỉ số đánh giá Object Detection (`Precision`, `Recall`, `mAP50`, `mAP50-95`).
  - Thông tin đặc tả bộ dữ liệu giao thông UA-DETRAC (quy ước các lớp phương tiện và vùng Ignore regions).