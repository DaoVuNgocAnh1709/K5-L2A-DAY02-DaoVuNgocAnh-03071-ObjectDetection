# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Phương tiện ô tô con cá nhân từ 4–7 chỗ ngồi phục vụ di chuyển thông thường: sedan, hatchback, SUV, crossover, coupe, xe taxi và xe bán tải (pickup) cabin nhỏ. | Có nắp capo trước (mũi xe) và cốp sau rõ ràng (sedan) hoặc dáng 2 khoang gầm cao (SUV/crossover như Honda CR-V ở `drive_002.jpg`, Kia Soul ở `drive_005.jpg`); xe taxi thường sơn 2 màu và có mào trên nóc. Không bao gồm xe van hình hộp hoặc xe tải chở hàng. |
| `truck` | Xe tải chuyên dụng chở hàng hóa, xe đầu kéo, xe ben, xe cứu hộ giao thông hoặc xe công trình có kết cấu thùng/sàn chở hàng tách biệt với cabin buồng lái. | Nhìn rõ phần đầu cabin phía trước tách rời với thùng hàng phía sau (thùng kín, thùng bạt, thùng ben chở đất đá hoặc sàn xe cứu hộ như chiếc xe trắng ở `drive_005.jpg`, `drive_006.jpg`). Không gán cho xe bán tải nhỏ thuộc lớp `car`. |
| `bus` | Xe buýt công cộng, xe buýt khớp nối (2 khoang), xe khách đường dài hoặc minibus cỡ lớn chuyên chở nhiều hành khách. | Thân xe hình hộp chữ nhật rất dài và cao, dọc hai bên sườn có dãy nhiều ô cửa kính hành khách liên tiếp và cửa lên xuống lớn (như xe buýt nội đô ở `drive_002.jpg`, xe buýt khớp nối ở `drive_011.jpg`, `drive_015.jpg`). |
| `van` | Xe van thương mại, microvan hoặc minivan (MPV thân hộp dài) dùng để chở hàng nhẹ hoặc chở người. | Thân xe liền một khối hình hộp từ trước ra sau, phần đuôi cắt thẳng đứng; bao gồm cả dòng xe van đầu tù (Wuling, Jinbei), microvan có giá chở hàng trên nóc (`drive_002.jpg`, `drive_006.jpg`) và minivan chở người thân dài (`drive_006.jpg`). |

## Quy tắc chung

- **Box bám sát vật thể nghĩa là:** Bốn cạnh của bounding box phải tiếp xúc sát với các điểm ngoài cùng của phần thân xe thực tế nhìn thấy được (bao gồm bánh xe, cản trước/sau, gương chiếu hậu và giá nóc nếu có). Tuyệt đối không lấy dư nền mặt đường, khoảng trống phía trên nóc xe, vệt nước bắn hoặc bóng đổ/bóng phản chiếu của xe dưới mặt đường ướt (ví dụ ở `drive_006.jpg`).
- **Vật thể bị che khuất:**
  - Nếu xe bị che khuất một phần bởi xe khác, bụi cây dải phân cách hoặc mái hiên nhưng phần lộ ra vẫn đủ đặc trưng để nhận diện chắc chắn lớp (`car`/`truck`/`bus`/`van`), vẽ box ôm sát đúng **phần diện tích thực tế nhìn thấy được** của xe đó.
  - Nếu hai xe chồng/che lấp một phần lên nhau (như xe `van` trắng đỗ sau chiếc SUV `car` trắng ở `drive_001.jpg`), tách thành **2 box riêng biệt** cho từng xe, không gộp chung thành 1 box.
  - Nếu một xe bị vật cản hẹp thẳng đứng ở tiền cảnh (như cột đèn, cột biển báo ở `drive_001.jpg`) chắn ngang giữa thân xe, vẫn vẽ **1 box duy nhất** bao trọn từ đầu đến đuôi của chiếc xe đó.
- **Vật thể bị cắt bởi rìa ảnh:** Với các xe đi vào/ra ở mép khung hình, vẽ box dừng chạm đúng biên ảnh (`x = 0`, `x = 1`, `y = 0` hoặc `y = 1`), không kéo vượt ra ngoài khung ảnh và không tự ước lượng phần xe nằm ngoài ảnh. Chỉ gán khi phần thân xe nằm trong ảnh còn đủ rõ để xác định đúng lớp.
- **Vật thể rất nhỏ/mơ hồ:**
  - Những xe ở quá xa phía chân trời/gầm cầu vượt (ví dụ đám xe tít phía xa gần cột đèn tín hiệu trên cùng ở `drive_002.jpg`, `drive_003.jpg`) bị nhòe mờ, dính liền vào nhau không tách bạch được ranh giới từng xe, hoặc chỉ lộ một mảnh nóc cực nhỏ ở mép ảnh không thể xác định chắc chắn lớp thì **không gán nhãn**.
  - Các phương tiện đang đỗ ven đường hoặc đường gom (như ở `drive_001.jpg`) vẫn gán nhãn bình thường theo loại xe vì schema phân loại theo loại vật thể chứ không theo trạng thái di chuyển.
  - Không gán người đi bộ, xe đạp, xe máy/mô tô, xe ba gác (như ở `drive_005.jpg`, `drive_011.jpg`), biển báo hay hình phản chiếu.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_001.jpg` | Tại đường gom bên trái (ngay chân cột biển báo lớn), một chiếc xe van màu trắng đỗ phía sau bị chiếc SUV màu trắng (`car`) và cột biển báo che khuất phần thân dưới; đồng thời trên làn chính bên phải có chiếc SUV trắng bị cột đèn cắt ngang giữa thân. | Vẽ **2 box riêng** ở chỗ cột biển báo: 1 box `van` (`3`) ôm phần thân trên và bánh trước nhìn thấy của xe van trắng + 1 box `car` (`0`) ôm chiếc SUV trắng phía trước. Với chiếc SUV trắng trên làn chính bị cột đèn chắn ngang, vẽ **1 box `car` (`0`)** bao trọn thân xe. | Theo quy tắc trong `GUIDE.md`: xe đang đỗ vẫn gán theo loại xe; hai xe chồng lên nhau phải vẽ hai box riêng trên phần nhìn thấy của từng xe; một xe bị cột mảnh che ngang vẫn là một thực thể duy nhất nên gán 1 box. |
| `drive_002.jpg` | Chiếc xe màu trắng cỡ nhỏ ở làn bên trái có thân hình hộp cao, mũi ngắn và lắp khung giá chở hàng trên nóc (dòng microvan/Changhe Beidouxing), gây phân vân giữa xe hatchback cỡ nhỏ (`car`) và xe van nhỏ (`van`). Ngoài ra ở phía xa trên cùng có đoàn xe nối đuôi nhau dưới đèn tín hiệu. | Gán chiếc xe trắng có giá nóc là **`van` (`3`)** (áp dụng nhất quán cho mẫu xe tương tự màu xanh ngọc ở `drive_006.jpg`). Với đoàn xe ở xa, chỉ gán các xe từ ngang chiếc `bus` thứ 2 trở xuống còn nhìn rõ hình khối, bỏ qua các chấm xe quá nhỏ/nhòe dính nhau ở sát chân trời. | Schema định nghĩa `van` bao gồm cả *"xe van, minivan chở hàng hoặc chở người"*; mẫu xe này có khoang hình hộp đứng và giá chở hàng đặc trưng của microvan. Các chấm xe ở quá xa không đủ độ phân giải để xác định biên và lớp chính xác. |
| `drive_005.jpg` | Ở làn giữa bên phải (sát hàng rào trắng) có chiếc xe chuyên dụng/cứu hộ giao thông màu trắng có sàn chở phía sau; ở làn ngoài cùng bên trái có người điều khiển xe máy/xe đạp. | Gán chiếc xe cứu hộ màu trắng là **`truck` (`1`)**; **bỏ qua không gán** người đi xe máy/xe đạp ở làn trái. | Xe cứu hộ có cấu trúc cabin buồng lái tách biệt với phần sàn/cẩu chở hàng phía sau nên thuộc lớp `truck` (`1`). Xe 2 bánh và người đi bộ nằm ngoài 4 lớp của schema. |
| `drive_006.jpg` | Ở góc trên bên phải (phía trên 2 chiếc taxi vàng) có chiếc xe màu xanh đen thân dài có mũi capo (dòng Buick GL8), dễ gây lưỡng lự giữa SUV (`car`) và minivan chở người (`van`). | Gán là **`van` (`3`)**, đồng thời cắt cạnh dưới của box sát mép lốp xe thật (không lấy bóng bánh xe phản chiếu dưới mặt đường mưa ướt). | Tra cứu kiểu dáng thực tế (Buick GL8) thuộc phân khúc **minivan chở người**; theo đúng bảng schema trong `README.md`, lớp `van` (`3`) bao gồm *"xe van, minivan chở hàng hoặc chở người"*. |
