# Báo cáo Day 13 — Khưu Minh Phú, 2A202603337

## 1. Kết luận trước
- **Thông tin chung:** Mã số học viên `2A202603337`, task `D13-2A202603337`. Bộ 10 frame được phân: `s01f13`, `s01f58`, `s02f07`, `s02f27`, `s03f19`, `s03f40`, `s04f20`, `s04f43`, `s05f19`, `s05f37`. Hệ điều hành Windows 11, trình duyệt Google Chrome, phiên bản CVAT 3D v2.83.0. MSSV bạn ngồi cạnh (đổi kiểm chéo): `2A202603338`.
- **Tổng kết kết quả gán nhãn:** Tổng cộng đã vẽ **82 cuboid** dạng Shape trên 10 frame (`s01f13=8`, `s01f58=8`, `s02f07=8`, `s02f27=8`, `s03f19=11`, `s03f40=5`, `s04f20=10`, `s04f43=8`, `s05f19=8`, `s05f37=8`). Frame có ít cuboid nhất là `s03f40` (5 cuboid) do cảnh đường thoáng hơn ở vùng gần ≤ 30 m và các vật ở xa quá thưa điểm không đủ cơ sở 15–20% thể tích để đoán. Công cụ `selfcheck` chỉ cờ **2/82 cuboid** (cả hai đều là cảnh báo nhiều điểm sát ngoài mép ở class `Pedestrian` đứng cạnh vật khác). Tôi tự tin nhất ở phần xác định **hướng yaw**, **phân loại class** và **căn đáy chạm mặt đường** cho các phương tiện trong tầm 30 m; phần còn cần lưu ý nhất là ước lượng chiều dài và độ cao đáy của các **vật ở xa trên 35–50 m** khi điểm LiDAR thưa và mặt đường có độ dốc nhẹ.
- **Cách dùng CVAT 3D:**
  1. **Đổi góc nhìn:** Kéo chuột trái trên khung 3D Perspective để xoay (kéo cho tới khi nhìn thẳng từ trên xuống để quan sát toàn cảnh 360°), kéo chuột phải để dời tâm nhìn (pan), lăn con lăn chuột để phóng to/thu nhỏ và nháy đúp chuột vào khoảng trống để quay về góc nhìn mặc định.
  2. **Vẽ và chỉnh một cuboid:** Chọn công cụ Cuboid → chọn Class → bấm **Shape** (không dùng Track), đưa con trỏ tới cụm điểm của vật rồi nháy đúp để đặt hộp; sau đó dùng khung **Top** để dời tâm, chỉnh chiều dài/rộng và xoay góc `yaw`, dùng khung **Side** và **Front** để hạ đáy chạm sát mặt đường và chỉnh chiều cao (tuyệt đối không xoay nghiêng ở Side/Front).
  3. **Bật mũi tên hướng và kiểm trên ảnh camera:** Vào mục **Appearance → Cuboid orientation** để hiển thị vạch đỏ ngắn (`+X` của cuboid) chỉ về đầu xe, đối chiếu với 7 góc camera trên CVAT và chạy `python scripts/lab.py preview` để kiểm tra hình chiếu 3D cuboid lên 7 ảnh camera trong `work/xem/`.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: `s01f13` và `s01f58`. Ảnh: `t1-s01f13-3d.png`, `t1-s01f13-top.png`, `t1-s01f58-3d.png`, `t1-s01f58-top.png` (ảnh nhìn từ trên xuống có đánh dấu phía trước `−y`, trục `+x`, trục `+y`).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f13` | Phía trước xe (`−y`, hướng `cam01`) hướng lên cạnh trên của ảnh top-view; trục `+x` chỉ sang cạnh trái (bên trái xe), trục `+y` chỉ xuống cạnh dưới (phía sau xe). | Mặt đường tương đối phẳng, thấp hơn tâm LiDAR khoảng `1.68 m – 1.72 m` (`z ≈ -1.70 m`). | Khoảng `36 m – 45 m` (xe con ở `36.06 m` phía sau bên trái và cấu trúc ven đường). | `Car`, `Bus`, `Pedestrian`. | Phía trước-trái có xe con gần (`x=3.04 m, y=-4.09 m`); phía sau-phải có người đi bộ (`x=-7.26 m, y=6.14 m`, thấy qua `cam08`); phía sau có các xe con (`y=20.68 m` tới `32.12 m`, thấy qua `cam09`/`cam05`). | Cụm điểm thưa ở khoảng cách `> 45 m` lẫn vào tán cây và dải phân cách bên lề không rõ ranh giới phương tiện. |
| `s01f58` | Phía trước xe (`−y`, hướng `cam01`) hướng lên cạnh trên của ảnh top-view; trục `+x` chỉ sang cạnh trái, trục `+y` chỉ xuống cạnh dưới (phía sau xe). | Mặt đường phẳng ở gần, hơi thoải nhẹ ở xa; thấp hơn tâm LiDAR khoảng `1.71 m – 1.75 m` (`z ≈ -1.72 m`). | Khoảng `35 m – 40 m` dọc theo làn đường trước và sau. | Chủ yếu là `Car` trên các làn cùng chiều. | Ngay sát bên trái-sau nhẹ có xe con ở `x=2.89 m, y=0.66 m` (`dist_m=2.96 m`, góc `cam05`/`cam06`); phía sau có xe con ở `y=6.75 m` và `y=28.65 m` (`cam09`). | Cụm điểm lẻ tẻ dưới 10 điểm ở xa trên `40 m` bị xe phía trước che khuất một phần nên không xác định chắc chắn loại xe. |

**Đối chiếu giữa Camera và LiDAR từ hai frame `s01f13` và `s01f58`:**
- **3 điều chỉ camera cho biết:**
  1. Màu sơn xe, kiểu dáng chi tiết thân xe (phân biệt rõ đầu xe và đuôi xe khi cụm điểm đối xứng, hoặc phân biệt sedan/SUV/bán tải).
  2. Trạng thái đèn tín hiệu giao thông, đèn phanh/đèn xi-nhan của xe phía trước và vạch kẻ phân làn trên mặt đường.
  3. Bối cảnh thị giác của người đi bộ hoặc người đi xe hai bánh (tư thế mặt quay về đâu, đang dắt xe hay đang ngồi lái xe).
- **3 điều chỉ LiDAR cho biết:**
  1. Tọa độ 3D và khoảng cách hình học chính xác theo đơn vị mét (`x, y, z`, `dist_m`) từ xe thu dữ liệu tới từng vật thể trong không gian 360°.
  2. Kích thước vật lý thực tế (chiều dài `length`, chiều rộng `width`, chiều cao `height` tính bằng mét) không bị biến dạng phối cảnh xa-gần như ảnh 2D.
  3. Độ cao mặt đường (`z_bottom ≈ -1.7 m`) và vị trí của các vật nằm ở vùng giao thoa hoặc góc mù giữa các khung hình camera.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f07` | Cạnh trên (`−y` hướng lên trên, `+x` sang trái, `+y` xuống dưới) | Có 2 xe buýt lớn (`Bus`) ở bên trái-trước (`dist_m = 14.31 m` và `30.90 m`) tạo vùng bóng che rộng phía sau thân xe. |
| `s02f27` | Cạnh trên (`−y` hướng lên trên, `+x` sang trái, `+y` xuống dưới) | Các xe ở xa `45.51 m – 45.89 m` phía trước (`y ≈ -45 m`) có `z_bottom` thấp hơn bình thường (`~ -1.95 m`) do mặt đường hơi thoải xuống. |
| `s03f19` | Cạnh trên (`−y` hướng lên trên, `+x` sang trái, `+y` xuống dưới) | Giao lộ đông đúc với nhiều loại đối tượng: nhóm 3 `Pedestrian` đi gần nhau bên phải (`x ≈ -12 m` đến `-14 m`) và 3 `Cyclist` chuyển hướng. |
| `s03f40` | Cạnh trên (`−y` hướng lên trên, `+x` sang trái, `+y` xuống dưới) | Mật độ phương tiện quanh xe thưa hơn các frame khác; có `Cyclist` ở `25.87 m` phía sau bên phải chỉ còn 52 điểm. |
| `s04f20` | Cạnh trên (`−y` hướng lên trên, `+x` sang trái, `+y` xuống dưới) | Mật độ xe con (`Car`) dày đặc ở cự ly gần 3–18 m bao quanh cả hai bên trái (`x > 0`), phải (`x < 0`) và phía sau (`y > 0`). |
| `s04f43` | Cạnh trên (`−y` hướng lên trên, `+x` sang trái, `+y` xuống dưới) | Nhiều xe xếp hàng dọc theo làn đường từ gần (`6.30 m`) tới rất xa (`47.91 m`), các xe ở xa bị che khuất một phần (`occluded=1`). |
| `s05f19` | Cạnh trên (`−y` hướng lên trên, `+x` sang trái, `+y` xuống dưới) | Có xe tải lớn (`Truck`, `length = 11.608 m`) ở khoảng cách `47.06 m` phía sau vẫn giữ được cụm điểm thùng xe rõ (218 điểm). |
| `s05f37` | Cạnh trên (`−y` hướng lên trên, `+x` sang trái, `+y` xuống dưới) | Có cả `Truck` ở gần (`16.66 m`) và một `Bus` ở rất xa phía trước (`dist_m = 59.93 m`, vượt ngưỡng 50 m, mặt đường dốc làm `z_bottom = -2.415 m`). |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f13`, `idx=1` | `Car` | Đúng (`yaw_deg = 266.2°`, hướng dọc thân xe về phía trước `−y`) | Chạm đường (`z_bottom = -1.684 m`, `ground_gap_m = 0.07 m`) | Đã kiểm đủ 3 chiếu Top/Side/Front và đối chiếu ảnh `cam01`/`cam06` | Khớp bảng 4.3 Sedan/Hatchback (`l = 4.365 m`, `w = 1.900 m`, `h = 1.440 m`) |
| (b) xe buýt hoặc xe tải | `s01f13`, `idx=3` | `Bus` | Đúng (`yaw_deg = 284.8°`, chỉ đúng về đầu xe buýt phía trước) | Chạm đường (`z_bottom = -1.628 m`, `ground_gap_m = 0.12 m`) | Đã kiểm đủ Top/Side/Front và đối chiếu ảnh `cam01`/`cam03` | Sát ngưỡng bảng 4.3 Xe buýt thường (`l = 12.695 m`, `w = 2.740 m`, `h = 3.023 m`, tính cả gương chiếu hậu) |
| (c) người đi bộ hoặc xe hai bánh | `s01f13`, `idx=6` | `Pedestrian` | Đúng (`yaw_deg = 0.0°`, theo hướng đứng/di chuyển của người) | Chạm mặt nền vỉa hè (`z_bottom = -1.432 m`, cao hơn lòng đường ~0.25 m) | Đã kiểm đủ Top/Side/Front và đối chiếu ảnh `cam08` | Khớp bảng 4.3 Người lớn (`l = 0.435 m`, `w = 0.795 m`, `h = 1.729 m`) |

- **Vòng hiệu chuẩn và thống nhất guideline:** Trong vòng hiệu chuẩn trên task chung, sự khác biệt thường gặp nhất giữa các thành viên là độ dài cuboid của xe bị che khuất một phần (có bạn chỉ vẽ bao cụm điểm nhìn thấy khiến hộp bị ngắn, có bạn ước lượng đủ thân xe), cách tính cả gương chiếu hậu vào chiều rộng `width`, và việc quên hạ đáy cuboid ở khung Side/Front. Cả lớp đã chốt theo đúng guideline: cuboid phải bao trọn cả phần gương/thùng chìa ra và phần thân bị che khuất có thể suy luận được theo kích thước chuẩn Bảng 4.3, bật cờ `occluded`, căn mũi tên đỏ `+X` về đúng đầu xe và kiểm tra `z_bottom` chạm mặt đất ở cả Side lẫn Front.
- **Các vật bỏ qua vì không đủ cơ sở:**
  - `s01f13` (phía trước khoảng `45 m – 50 m`, gần lề trái): Cụm dưới 10 điểm rất thưa nằm sau xe buýt `s01f13#3`, không thấy tới 15% thể tích và bị che trên camera → bỏ qua (`LOW_VISIBILITY` / `UNCERTAIN_SCOPE`).
  - `s03f40` (phía trước bên phải khoảng `42 m`): Cụm điểm nhỏ lẻ cạnh lề đường không phân biệt được là xe hai bánh dựng bên đường hay người đứng cạnh vật cản → bỏ qua (`UNCERTAIN_CLASS` / `LOW_VISIBILITY`).

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**. Các ảnh chụp được lưu trong `report/individual/images/`.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s01f13` (hoặc `s02f07`) / `t4-1-bong-che.png` / task cá nhân `D13-2A202603337` | Tia laser từ LiDAR truyền theo đường thẳng bị thân xe lớn (như xe buýt `s01f13#3`, `s02f07#1` hoặc xe phía trước) chắn lại, không xuyên qua được vật cản. | Phía sau thân xe chắn xuất hiện một vùng hình quạt trống hoàn toàn không có điểm LiDAR mặt đường và làm khuyết mất một phần điểm của các xe nằm sau (như `s01f13#7`, `s01f13#8`). | Vùng trống sau vật chắn là vùng chưa quan sát được chứ không phải đường trống; với vật bị che một phần, phải bật thuộc tính `occluded` và vẽ cuboid bao trọn cả phần thể tích suy luận được. |
| 2. Thưa ở xa | `s05f37` (vật `idx=6`, `dist_m = 48.39 m`) / `t4-2-thua-o-xa.png` / task cá nhân `D13-2A202603337` | Các chùm tia LiDAR 40 tia phân kỳ theo góc chiếu, nên khi khoảng cách tăng lên (30–50 m), khoảng cách giữa các vòng tia quét và các điểm đo giãn rộng ra. | Chiếc xe con `s05f37#6` ở khoảng cách `48.39 m` chỉ còn đúng `32 điểm` LiDAR rời rạc trên bề mặt hướng về phía LiDAR, mất hoàn toàn các mặt khuất. | Nếu còn thấy được khoảng 15–20% thể tích hoặc ảnh camera hỗ trợ nhận diện rõ thì vẫn vẽ cuboid theo kích thước hợp lý của Bảng 4.3; nếu quá thưa không đủ cơ sở suy luận thì bỏ qua và ghi `LOW_VISIBILITY`. |
| 3. Phản xạ thấp / ít điểm bất thường | `s01f58` (vật `idx=8`, `dist_m = 20.27 m`) / `t4-3-phan-xa-thap.png` / task cá nhân `D13-2A202603337` | Giả thuyết: bề mặt vật liệu tối màu hấp thụ tia hồng ngoại của LiDAR, hoặc bề mặt kính chắn gió/cửa sổ nghiêng gây phản xạ gương làm tia laser bật đi hướng khác không quay về đầu thu. | Xe con `s01f58#8` ở cự ly trung bình `20.27 m` nhưng chỉ thu được `59 điểm` (khuyết hoàn toàn vùng kính và nóc), ít hơn hẳn xe `s01f13#4` ở cùng tầm `20.69 m` (`289 điểm`). | Không được co nhỏ hộp theo đám điểm bị khuyết; phải đối chiếu ảnh camera và Bảng 4.3 để dựng cuboid bao đủ kích thước thực của toàn bộ thân xe, đồng thời chỉ ghi nguyên nhân là giả thuyết. |
| 4. Điểm lạ | `s02f35` (hoặc điểm nhiễu trên frame) / `t4-4-diem-la.png` / task chung `D13-HC485087` (và đối chiếu task cá nhân) | Giả thuyết: hiện tượng đa phản xạ (multipath reflection) khi tia laser phản xạ qua mặt kính/bề mặt bóng hoặc nhiễu phần cứng cảm biến LiDAR. | Xuất hiện cụm điểm nằm lệch hẳn xuống dưới mặt đường (ở `z ≈ -13.5 m`, sâu hơn mặt đường quanh đó khoảng `11 m`) hoặc lơ lửng giữa không khí mà không có vật thể nào trên ảnh camera. | Nhận diện đây là nhiễu hình học (outlier/ghost points), tuyệt đối không gán nhãn thành vật thể. |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603337-bangA.csv`; số điểm lấy từ cột `points_in` của `submission/D13-2A202603337-tukiem.csv`. Mặt đường khoảng `−1.7 m`.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f13`, `idx=1`, `Car` | `-0.964 m` | `1.440 m` | `-1.684 m` | `5.10 m` | `5.19 m` | `266.2°` | Chạm đường chuẩn (lệch `0.016 m` so với `-1.70 m`, `ground_gap_m = 0.07 m`). |
| `s01f58`, `idx=1`, `Car` | `-0.980 m` | `1.468 m` | `-1.714 m` | `2.96 m` | `3.12 m` | `269.3°` | Chạm đường chuẩn (lệch `0.014 m` so với `-1.70 m`, `ground_gap_m = 0.09 m`). |
| `s03f19`, `idx=4`, `Pedestrian` | `-0.816 m` | `1.791 m` | `-1.712 m` | `19.02 m` | `19.04 m` | `263.4°` | Chạm đường chuẩn (`z_bottom = -1.712 m`, `ground_gap_m = 0.07 m`). |

- **(a) Cuboid gần nhất và xa nhất:**
  - **Cuboid gần nhất:** `s01f58`, `idx=1` (`Car`, tọa độ `x = 2.890 m, y = 0.655 m, z = -0.980 m`) có khoảng cách ngang `dist_m = 2.96 m` và khoảng cách 3D `r = √(2.890² + 0.655² + (-0.980)²) = 3.12 m`.
  - **Cuboid xa nhất trong phạm vi chuẩn ≤ 50 m:** `s05f37`, `idx=6` (`Car`, tọa độ `x = -4.518 m, y = 48.184 m, z = -1.012 m`) có khoảng cách ngang `dist_m = 48.39 m` và khoảng cách 3D `r = √((-4.518)² + 48.184² + (-1.012)²) = 48.41 m`. *(Ngoài ra, nếu tính cả hộp ngoài ranh 50 m thì trong bảng A có `s05f37`, `idx=8`, `Bus` ở `x = 4.468 m, y = -59.763 m, z = -0.722 m` với `dist_m = 59.93 m` và `r = 59.93 m`)*.
- **(b) So sánh kích thước `l, w, h` với Bảng 4.3:**
  - **Người đi bộ (`s01f13`, `idx=6`, `Pedestrian`):** `length = 0.435 m`, `width = 0.795 m`, `height = 1.729 m`. Đối chiếu với Bảng 4.3 cho Người lớn (`Dài 0.4–0.9 m`, `Rộng 0.5–0.9 m`, `Cao 1.50–1.95 m`), cả 3 chiều đều nằm hoàn toàn trong khoảng chuẩn. *(Tương tự với xe hai bánh `s02f07`, `idx=4`, `Cyclist`: `l = 1.763 m, w = 0.677 m, h = 1.525 m`, khớp chuẩn Bảng 4.3 nhưng `width` của xe đạp/xe máy hơi gọn hơn 0.70 m một chút khoảng 3 cm do người lái ngồi gọn theo thân xe)*.
  - **Xe con (`s01f13`, `idx=1`, `Car`):** `length = 4.365 m`, `width = 1.900 m`, `height = 1.440 m`. Đối chiếu với Bảng 4.3 cho Sedan/Hatchback (`Dài 3.8–5.0 m`, `Rộng 1.70–1.90 m`, `Cao 1.40–1.60 m`), cả 3 kích thước đều khớp hoàn toàn trong khoảng tham chiếu. Với một số xe con ở xa như `s05f37#6` (`dist_m = 48.39 m`, `l = 3.427 m, w = 1.524 m, h = 1.224 m`), chiều dài và rộng nhỏ hơn khoảng chuẩn Bảng 4.3 vì ở cự ly gần 50 m chỉ còn 32 điểm ở phần đuôi/hông xe, thiếu điểm ở đầu xe.
- **(c) Góc `yaw_deg` của xe cùng chiều và ngược chiều:**
  - **Xe cùng chiều phía trước:** `s01f58`, `idx=2` (`Car`, `y = -8.841 m`) có `yaw_deg = 270.1°` (và `s01f13`, `idx=2` có `yaw_deg = 271.3°`), rất sát giá trị lý thuyết **`270°`** (hướng `−y`).
  - **Xe đi ngược chiều:** `s05f37`, `idx=7` (`Car`) có `yaw_deg = 90.1°` (và `s04f20`, `idx=5` có `yaw_deg = 90.6°`; `s02f27`, `idx=1` có `yaw_deg = 89.6°`), rất sát giá trị lý thuyết **`90°`** (hướng `+y`).

**Mật độ theo khoảng cách (4 cuboid `Car` ở các khoảng cách `dist_m` khác nhau):**

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s01f58`, `idx=1`, `Car` | `2.96 m` | `3122` |
| `s05f19`, `idx=1`, `Car` | `10.58 m` | `1384` |
| `s02f07`, `idx=5`, `Car` | `21.39 m` | `234` |
| `s05f19`, `idx=5`, `Car` | `36.97 m` | `81` |

- **Quy luật mật độ và so sánh với slide bài giảng:** Khi khoảng cách `dist_m` từ LiDAR tới vật tăng gấp khoảng `2 lần`, số điểm LiDAR nằm trong cuboid (`points_in`) giảm mạnh còn khoảng `1/4 đến 1/3` (giảm xấp xỉ theo hàm mũ / nghịch đảo bình phương khoảng cách). So với đường cong trung vị của xe con trên slide bài giảng (`0–10 m: ~1968 điểm`, `10–20 m: ~718 điểm`, `20–30 m: ~214 điểm`, `30–40 m: ~72 điểm`), số liệu thực tế trên cuboid của tôi hoàn toàn tương đồng với xu hướng lý thuyết (`2.96 m` có `3122 điểm`; `10.58 m` có `1384 điểm`; `21.39 m` có `234 điểm` rất sát mốc `214 điểm`; `36.97 m` có `81 điểm` rất sát mốc `72 điểm`).
- **Tự kiểm và chỉnh sửa sau `preview` / `selfcheck`:**
  - **Hai cuboid đã sửa trên CVAT trước khi lưu bản cuối:**
    1. `s04f43`, `idx=8` (`Car`, `dist_m = 47.91 m`): Ban đầu hộp có chiều rộng `width = 2.51 m` vượt ngoài khoảng cho phép `1.4–2.3 m` của class `Car` (bị bao dư điểm lề đường bên cạnh). Tôi đã mở lại khung Top trên CVAT, thu hẹp cạnh ngang của cuboid về `width = 2.300 m` sát đúng cụm điểm thân xe rồi Save và chạy lại `sync`.
    2. `s05f19`, `idx=6` (`Truck`, `dist_m = 47.06 m`): Ban đầu hộp có chiều rộng `width = 3.36 m` vượt ngoài khoảng `1.6–3.2 m` của class `Truck`. Tôi đã kiểm tra lại ở khung Top và Front, chỉnh lại chiều rộng hộp về `width = 3.200 m` (`length = 11.608 m, height = 3.317 m`) vừa khít cụm điểm thùng xe tải rồi Save và chạy lại `sync` (cờ cảnh báo kích thước đã hết).
  - **Cảnh báo giữ nguyên và lý do:** Còn **2 cảnh báo** ở `s01f13`, `idx=6` (`Pedestrian`, `"383 điểm sát ngay ngoài mép"`) và `s03f19`, `idx=4` (`Pedestrian`, `"47 điểm sát ngay ngoài mép"`). Khi kiểm tra lại trên point cloud và ảnh camera, đây là những người đi bộ đứng ngay sát người đi bộ khác (`s03f19#5`) hoặc đứng sát vật cản/bậc thềm trên vỉa hè (`s01f13#6`); nếu nới rộng hộp để nuốt các điểm sát mép đó thì cuboid sẽ bao lấn sang vật thể bên cạnh và vượt kích thước người chuẩn ở Bảng 4.3, nên tôi quyết định giữ nguyên kích thước chuẩn của người đi bộ.
- **Đổi kiểm chéo (Peer-check):** Đã đổi kiểm với bạn có MSSV **`2A202603338`**. Hai con số được đối chiếu trực tiếp với `submission/D13-2A202603337-bangA.csv` là: (1) độ cao đáy `z_bottom = -1.684 m` và khoảng cách `dist_m = 5.10 m` của cuboid `s01f13#1` (`Car`), và (2) khoảng cách `dist_m = 48.39 m` cùng số điểm `points_in = 32` của cuboid `s05f37#6` (`Car`). Kết quả: cả hai con số đều **khớp 100%** với Bảng A và Phiếu tự kiểm.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
- **(a) Ý nghĩa của nhãn đối với việc huấn luyện model 3D:**
  1. Nhãn 3D phải đo bằng đơn vị **mét** trong không gian thực (thay vì pixel trên ảnh 2D) vì hệ thống tự hành cần học và dự đoán trực tiếp vị trí, kích thước vật lý thực và khoảng cách an toàn trong không gian 3D quanh xe để lập kế hoạch quỹ đạo tránh va chạm, không bị phụ thuộc vào tiêu cự ống kính hay biến dạng phối cảnh xa-gần của camera.
  2. Vật thưa điểm cần có quy tắc ngưỡng riêng (chỉ vẽ khi thấy ≥ 15–20% thể tích hoặc đủ cơ sở suy luận, nếu ít hơn phải bỏ qua thay vì đoán mò) vì nếu gán nhãn bừa các cụm vài điểm không rõ hình dạng làm ground-truth, model sẽ bị ép học các đặc trưng nhiễu (false positives) hoặc học sai kích thước/hướng hộp, làm giảm độ tin cậy của bộ phát hiện đối tượng.
- **(b) Cách dùng AI trong bài lab:**
  - **Công cụ:** Antigravity AI.
  - **Câu hỏi chính:** Hướng dẫn quy trình từng bước làm bài.
  - **Cách dùng kết quả:** Tôi tự thực hiện toàn bộ việc đọc cảnh, xoay khung 3D, vẽ và tinh chỉnh 82 cuboid trên CVAT cũng như chụp màn hình; kết quả từ AI được dùng để kiểm tra tính nhất quán toán học từ chính dữ liệu CSV do tôi xuất ra và hệ thống hóa văn phong báo cáo.
- **(c) Điều còn chưa chắc:**
  - Ở các cự ly xa trên `40 m – 50 m` (như ở `s02f27`, `s04f43`, `s05f37`), mặt đường có độ dốc/thoải nhẹ khiến `z_bottom` của các xe ở xa xuống tới khoảng `-1.95 m` đến `-2.23 m`; dù đã căn sát các điểm mặt đường tại chỗ trên khung Side/Front, việc xác định chính xác ranh giới giữa bánh xe chạm đất và mặt đường khi mật độ tia LiDAR dưới chân xe quá thưa vẫn còn độ bất định nhất định.
  - Với các xe bị che khuất phần đầu hoặc phần đuôi ở xa (`occluded = 1`), việc ước lượng chiều dài `length` theo Bảng 4.3 vẫn có thể sai lệch vài chục centimet nếu không nhìn rõ được dòng xe cụ thể trên ảnh camera.
