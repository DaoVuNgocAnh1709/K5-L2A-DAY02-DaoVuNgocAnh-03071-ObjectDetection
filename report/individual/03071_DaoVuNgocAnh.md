# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Đào Vũ Ngọc Ánh
- MSSV: 2A202603071 (03071)
- Email: ngocanhdaovu1709@gmail.com
- Lớp: K5-L2A
- Link repo bài làm: https://github.com/DaoVuNgocAnh1709/K5-L2A-DAY02-DaoVuNgocAnh-03071
- Link Colab đã chạy: *(Cập nhật sau khi chạy xong notebook trên Colab ở CP6)*

## 2. Schema và mini guideline

Chi tiết định nghĩa 4 lớp và quy tắc gán nhãn đã được trình bày tại [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).

- **Cách áp dụng schema (`0: car`, `1: truck`, `2: bus`, `3: van`):**
  - Phân lớp dựa trên **loại phương tiện**, không phụ thuộc vào trạng thái đang di chuyển hay đang đỗ (xe đỗ ven đường gom ở `drive_001.jpg`, `drive_028.jpg` vẫn gán nhãn theo đúng lớp).
  - Các phương tiện 2–3 bánh (xe máy, xe đạp, xe ba gác chở khách ở `drive_005.jpg`, `drive_011.jpg`, `drive_024.jpg`) và người đi bộ được loại trừ hoàn toàn khỏi bộ nhãn.
- **Tóm tắt các ca mơ hồ đã quyết định:**
  1. **`drive_001.jpg` (Hai xe chồng lấp & vật cản tiền cảnh):** Tại đường gom bên trái ngay chân cột biển báo lớn, chiếc xe `van` trắng đỗ phía sau bị chiếc SUV trắng (`car`) và cột biển báo che khuất phần dưới $\rightarrow$ Tách thành **2 box riêng biệt** (`van` và `car`) ôm phần diện tích thực tế nhìn thấy của từng xe. Với chiếc SUV trắng trên làn chính bị cột đèn kim loại chắn ngang giữa thân $\rightarrow$ Vẽ **1 box `car` duy nhất** bao trọn thân xe.
  2. **`drive_002.jpg` (Ranh giới `van` vs `car` & xe ở xa):** Chiếc xe trắng cỡ nhỏ ở làn bên trái có thân hình hộp cao, mũi ngắn và lắp giá chở hàng trên nóc (dòng microvan Changhe Beidouxing) $\rightarrow$ Thống nhất gán **`van` (`3`)** (áp dụng nhất quán cho mẫu xe tương tự màu xanh ngọc ở `drive_006.jpg`). Với đoàn xe nối đuôi ở xa trên cùng, chỉ gán các xe từ ngang chiếc `bus` thứ 2 trở xuống còn nhìn rõ hình khối, bỏ qua các chấm xe quá nhỏ/dính liền sát chân trời.
  3. **`drive_005.jpg` (Xe chuyên dụng & phương tiện ngoài schema):** Chiếc xe cứu hộ giao thông màu trắng ở làn giữa bên phải có cabin buồng lái tách biệt với sàn/cẩu chở hàng phía sau $\rightarrow$ Gán **`truck` (`1`)**; bỏ qua người đi xe máy/xe đạp ở làn ngoài cùng bên trái.
  4. **`drive_006.jpg` (Minivan chở người thân dài & bóng phản chiếu):** Chiếc xe màu xanh đen ở góc trên bên phải (dòng Buick GL8) có mũi capo dễ nhầm với SUV (`car`), nhưng có khoang hành khách hình hộp kéo dài của dòng minivan chở người $\rightarrow$ Gán **`van` (`3`)** theo đúng định nghĩa *"xe van, minivan chở hàng hoặc chở người"* của schema; đồng thời cắt cạnh dưới của box sát mép lốp thật, không lấy bóng bánh xe phản chiếu dưới mặt đường mưa ướt.

## 3. Annotation và self-QC

- **Số file nhãn train:** 50 file (`drive_001.txt` đến `drive_050.txt` trong `dataset/labels/train/`), tổng cộng **805 bounding boxes** (`car`: 637, `truck`: 23, `bus`: 88, `van`: 57).
- **Kết quả checker và lỗi đã xử lý:**
  - Chạy lệnh `python -X utf8 tools/check_yolo_labels.py` đạt kết quả:
    `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ. Chờ ground truth để chấm chất lượng.`
  - Toàn bộ 50 file đều đúng basename, chuẩn 5 cột normalized `[0, 1]`, `class_id` thuộc `{0, 1, 2, 3}` và không có box vượt biên.
- **Danh sách ít nhất 10 ảnh đã rà bằng mắt (Self-QC):**
  Đã kiểm tra trực tiếp 16 ảnh đại diện cho nhiều góc máy, điều kiện ánh sáng (ngày/đêm/mưa) và mật độ giao thông khác nhau:
  `drive_001.jpg`, `drive_002.jpg`, `drive_003.jpg`, `drive_004.jpg`, `drive_005.jpg`, `drive_006.jpg`, `drive_008.jpg`, `drive_009.jpg`, `drive_011.jpg`, `drive_015.jpg`, `drive_024.jpg`, `drive_026.jpg`, `drive_031.jpg`, `drive_035.jpg`, `drive_042.jpg`, `drive_050.jpg`.
- **Lỗi bỏ sót / box / class phát hiện khi rà và cách sửa:**
  1. **Lỗi sót xe và lệch box ở dãy xe đỗ (`drive_001.jpg`):** Ở dãy xe đỗ trước cửa hàng biển đỏ, phát hiện chiếc sedan trắng thứ 4 bị bỏ sót do box của xe đen thứ 3 bị lệch sang phải; đồng thời cạnh trên của box chiếc SUV trắng trên làn chính hơi dư phần mặt đường $\rightarrow$ Đã chỉnh khít lại box xe đen thứ 3, bổ sung box `car` cho xe trắng thứ 4 và kéo sát cạnh trên của SUV trắng.
  2. **Lỗi box trùm sang xe bên cạnh & sót hàng xe trung cảnh (`drive_002.jpg`):** Box của chiếc `bus` thứ 2 ở phía trên bị rộng sang phải, trùm lên chiếc `car` màu xám đi sát sườn phải; ngoài ra một số xe `car` và `van` ở trung cảnh (ngang chiếc `bus` thứ 2 trở xuống) ban đầu chưa được gán $\rightarrow$ Đã thu hẹp cạnh phải của box `bus` về sát thân xe buýt và bổ sung đầy đủ các box `car`/`van` ở trung cảnh.
  3. **Lỗi lấy dư bóng mặt đường (`drive_004.jpg`, `drive_006.jpg`):** Dưới ánh đèn đường ban đêm (`drive_004.jpg`) và mặt đường mưa ướt (`drive_006.jpg`), cạnh dưới của một số box bị kéo trùm xuống phần bóng đen/bóng phản chiếu bánh xe và cạnh trên box xe sedan đỏ ở `drive_006.jpg` bị cao hơn nóc xe $\rightarrow$ Đã kéo sát 4 cạnh box chạm đúng thân xe và đáy lốp xe thật.
  4. **Lỗi phân lớp `van` / `truck` / `bus` và vật thể ngoài scope (`drive_008.jpg`, `drive_009.jpg`, `drive_015.jpg`, `drive_024.jpg`, `drive_042.jpg`):**
     - Chuyển chiếc xe tải thùng kín màu xanh (China Post) ở `drive_009.jpg` từ `bus` về đúng `truck` (`1`).
     - Chuyển các xe minivan màu bạc/trắng ở `drive_008.jpg`, `drive_009.jpg`, `drive_024.jpg`, `drive_050.jpg` từ `car` về đúng `van` (`3`).
     - Xóa bỏ mảnh nóc xe quá nhỏ (< 7px) vừa nhô lên ở sát mép đáy ảnh `drive_015.jpg` và xóa xe ba gác chở người ở `drive_024.jpg`.
- **Commit khóa nhãn ban đầu:** *(Cập nhật mã commit ngay sau khi chạy `git commit` ở CP4)*

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | | |
| FP | | |
| FN | | |
| Precision | | |
| Recall | | |
| F1 | | |
| mean_iou_of_matches | | |
| passed | | |

Kết quả chi tiết lưu tại [`outputs/annotation_evaluation.json`](../../outputs/annotation_evaluation.json). *(Cập nhật nhận xét sau khi nhận `gold_train_labels/` và chạy `tools/evaluate_annotations.py` ở CP5).*

## 5. Kết quả YOLO

- **Runtime, GPU/CPU, phiên bản Ultralytics và Torch:** *(Cập nhật sau khi chạy Colab)*
- **Các tham số thay đổi so với notebook nếu có và lý do:** Không thay đổi (`epochs=50`, `imgsz=640`, `batch=-1`, `seed=20260909`).
- **Đường dẫn best checkpoint trong phiên chạy:** `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | |
| Recall | |
| mAP50 | |
| mAP50–95 | |

- **Thư mục kết quả từ Colab:**
  - Ảnh dự đoán trên 10 ảnh eval: [`assets/eval_predictions/visuals/`](assets/eval_predictions/visuals/)
  - Log huấn luyện và biểu đồ đánh giá: [`assets/yolo26_runs/vehicle_annotation/`](assets/yolo26_runs/vehicle_annotation/) và [`assets/yolo26_runs/vehicle_eval/`](assets/yolo26_runs/vehicle_eval/)

### Trả lời bốn câu hỏi cuối notebook:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   *(Cập nhật số liệu cụ thể theo confusion matrix / bảng kết quả per-class từ Colab, ví dụ lớp `truck` hoặc `van` do số lượng mẫu ít và sự khác biệt quy ước nhãn trên tập `eval` như `drive_052.jpg`, `drive_056.jpg`, `drive_058.jpg`).*

2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - `mAP50` chỉ yêu cầu độ chồng lấp `IoU >= 0.50` là được tính đúng (TP), trong khi `mAP50–95` lấy trung bình ở các ngưỡng khắt khe hơn từ `0.50` đến `0.95` (bước nhảy `0.05`).
   - Khi `mAP50` cao nhưng `mAP50–95` thấp, nghĩa là mô hình **phát hiện đúng vị trí và đúng lớp của xe**, nhưng **biên của bounding box chưa đủ khít (localization kém chính xác, IoU chỉ rơi vào khoảng 0.50–0.70)**.
   - Cần kiểm tra lại: (1) Box gán nhãn trong tập train có bị lỏng (lấy dư mặt đường, bóng đổ của xe) hay cắt lẹm mất phần đầu/đuôi xe không; (2) Cách vẽ box cho các xe bị che khuất một phần (bởi bụi cây, xe khác, cột đèn) giữa tập train và tập eval đã nhất quán chưa.

3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi nhầm lẫn giữa các lớp **`van`, `car` và `truck`/`bus`** có liên quan trực tiếp đến ranh giới định nghĩa trong guideline:
     - Giữa **`van` và `car`**: Các dòng minivan chở người có mũi capo (như Buick GL8 ở `drive_006.jpg`, `drive_030.jpg`) hoặc microvan/hatchback trần cao có giá nóc (như ở `drive_002.jpg`, `drive_006.jpg`) nằm ở ranh giới giữa SUV/MPV gia đình (`car`) và xe van (`van`).
     - Giữa **`van`, `truck` và `bus` trên tập `eval`**: Khi quan sát trực tiếp nhãn của tập `eval` (ví dụ `drive_051.txt`, `drive_052.txt`, `drive_053.txt`, `drive_056.txt`, `drive_058.txt`), có sự lệch quy ước lớp (ví dụ xe buýt lớn ở `drive_052.jpg`, `drive_058.jpg` mang nhãn `3`, xe van ở `drive_056.jpg`, `drive_059.jpg` mang nhãn `1`, xe tải ở `drive_051.jpg` mang nhãn `2`) cộng thêm hiện tượng mất cân bằng lớp nặng (`car` chiếm tới ~79% tổng số box trong tập train, trong khi `truck` chỉ chiếm ~2.8% và `van` ~7.1%).

4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - **Không dùng 10 ảnh `eval` để train** nhằm tránh **rò rỉ dữ liệu (data leakage)**: nếu đưa ảnh `eval` vào tập huấn luyện, mô hình sẽ học thuộc lòng (overfit) chính các ảnh dùng để chấm điểm, làm các chỉ số `Precision`, `Recall`, `mAP` tăng ảo và không còn phản ánh khả năng tổng quát hóa trên dữ liệu chưa từng thấy.
   - **Vai trò validation trong notebook:** Trong `data.yaml`, `val` trỏ tới `images/eval`. Trong suốt 50 epochs huấn luyện, tập 10 ảnh `eval` này được dùng làm **validation set** để theo dõi độ hội tụ sau mỗi epoch và **lựa chọn checkpoint tốt nhất (`best.pt`)** có fitness/mAP cao nhất. Chính vì đã tham gia vào việc chọn `best.pt`, tập `eval` đóng vai trò validation chứ không phải một tập test độc lập hoàn toàn.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| CP1–CP2 (Phân lớp phương tiện) | Một số dòng xe MPV/minivan có mũi capo (như Buick GL8 ở `drive_006.jpg`) và microvan lắp giá nóc (`drive_002.jpg`) khó phân biệt giữa `car` và `van`. | Tra cứu kiểu dáng thực tế của xe, đối chiếu định nghĩa *"minivan chở hàng hoặc chở người"* trong `README.md` và ghi quy tắc thống nhất vào `GUIDELINE_MINI.md`. | Đã phân lớp nhất quán trên toàn bộ 50 ảnh. |
| CP2 (Độ khít box và che khuất) | Xe đỗ sau cột biển báo (`drive_001.jpg`), xe chạy trên mặt đường mưa ướt có bóng phản chiếu (`drive_006.jpg`) hoặc đoàn xe xếp hàng ở xa (`drive_002.jpg`). | Áp dụng quy tắc tách 2 box riêng cho 2 xe chồng nhau, cắt đáy box tại mép lốp thật (bỏ bóng nước) và chỉ gán các xe ở trung cảnh còn rõ hình khối. | Các box bám sát thân xe, không vượt biên ảnh. |
| CP3 (Chạy `check_yolo_labels.py` trên Windows) | Lệnh `python tools/check_yolo_labels.py` báo lỗi `UnicodeEncodeError: 'charmap' codec` khi in tiếng Việt có dấu trên PowerShell. | Chạy với cờ UTF-8: `python -X utf8 tools/check_yolo_labels.py`. | Script chạy thành công, báo `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ`. |

## 7. Tự đánh giá theo rubric

- **Hoàn thành và định dạng YOLO (10/10):** Đủ 50 file `.txt` khớp tên ảnh trong `dataset/labels/train/`, vượt qua `check_yolo_labels.py`.
- **Đầy đủ vật thể (20/20):** Đã gán đầy đủ 805 phương tiện thuộc 4 lớp, bao gồm cả xe bị che khuất một phần và xe bị cắt ở rìa ảnh còn nhận diện được lớp.
- **Độ bám sát và hình học box (20/20):** Box ôm sát phần nhìn thấy của xe, đã loại bỏ phần bóng đổ/phản chiếu mặt đường.
- **Phân lớp nhất quán (15/15):** Nhất quán giữa `car`, `truck`, `bus`, `van`; không gán xe máy, xe đạp, xe ba gác hay người đi bộ.
- **Mini guideline và quyết định mơ hồ (10/10):** Đã điền đầy đủ định nghĩa 4 lớp, quy tắc vẽ box và 4 ca mơ hồ cụ thể trong [`GUIDELINE_MINI.md`](../../GUIDELINE_MINI.md).
- **Gold-set annotation QC (15đ):** *(Cập nhật sau khi chạy `evaluate_annotations.py` với `gold_train_labels`)*.
- **Pipeline YOLO và phân tích (10đ):** *(Cập nhật sau khi chạy xong notebook trên Colab và bổ sung `assets/`)*.

## 8. Công cụ AI và tài liệu tham khảo

- **Công cụ đã sử dụng:**
  - **CVAT (chạy local qua Docker tại `localhost:8080`):** Công cụ gán nhãn, duyệt trực quan từng ảnh và xuất bộ nhãn định dạng `YOLO 1.1`.
  - **Antigravity AI Assistant & YOLO26m ONNX pre-annotation:** Hỗ trợ giải thích quy tắc schema, tra cứu dòng xe khó phân biệt (`van` vs `car`), khởi tạo gợi ý bounding box trên CVAT và hỗ trợ rà soát lỗi (Self-QC) về độ khít box, xe bị bỏ sót ở trung cảnh và cách khắc phục lỗi `UnicodeEncodeError` khi chạy script kiểm tra trên Windows.
- **Phần học viên đã tự kiểm tra:**
  - Tự thiết lập task CVAT, trực tiếp gán và duyệt lại trên giao diện CVAT qua 50 ảnh (`drive_001.jpg` đến `drive_050.jpg`), kiểm tra lại các ca che khuất/chồng lấp (`drive_001.jpg`, `drive_002.jpg`, `drive_006.jpg`) và xuất bộ nhãn cuối cùng từ CVAT.
- **Tài liệu tham khảo:** Tài liệu hướng dẫn trong repository (`README.md`, `GUIDE.md`, `CHECKPOINTS.md`, `RUBRIC.md`, `SUBMISSION.md`) và tài liệu Ultralytics YOLO.
