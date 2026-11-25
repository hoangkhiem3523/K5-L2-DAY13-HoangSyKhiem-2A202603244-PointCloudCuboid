# Báo cáo Day 13 — Hoàng Sỹ Khiêm, 2A202603244

## 1. Kết luận trước
- **Thông tin chung**: MSSV `2A202603244` · Task `D13-2A202603244` · 10 frame được giao: `s01f25`, `s01f49`, `s02f31`, `s02f53`, `s03f32`, `s03f54`, `s04f00`, `s04f47`, `s05f06`, `s05f40` · Hệ điều hành: Windows 11 · Trình duyệt: Google Chrome · CVAT local phiên bản `v2.82.1` (dùng tiếp bản từ Ngày 12 theo hướng dẫn) · MSSV bạn ngồi cạnh đổi kiểm: `2A202603270`.
- **Tổng quan kết quả**: Đã hoàn thành **91 cuboid** (100% dạng Shape) trên cả 10 frame; các frame có số lượng ít nhất là `s02f31`, `s02f53`, `s03f32`, `s04f47`, `s05f40` (đều **6 cuboid**/frame) do mật độ phương tiện trong vùng `≤ 30 m` thưa hơn và chủ động bỏ qua các xe quá xa không đủ cơ sở. Phiếu tự kiểm `selfcheck` cờ **4/91 cuboid** (3 cờ điểm sát mép do vật đứng sát cột/xe kế cận và 1 cờ chiều cao của xe van mái cao). Phần tin cậy nhất là đáy chạm đường (`z_bottom ≈ −1,7 m`), phân loại các phương tiện gần và hướng `yaw` của xe đi trên làn chính; phần còn yếu hơn là ước lượng chiều sâu mặt bị khuất của xe ở khoảng `25–35 m` và hướng của một số xe hai bánh/người đi bộ bị che một phần.
- **Cách dùng CVAT 3D**:
  1. Đổi góc nhìn: kéo chuột trái để xoay khung 3D (xoay thẳng từ trên xuống và thu nhỏ bằng con lăn để quan sát 360°), kéo chuột phải để dời khung nhìn, nháy đúp chỗ trống để về góc mặc định, bấm biểu tượng bánh răng trên từng khung camera để đổi qua đủ `cam01`–`cam09`.
  2. Vẽ và chỉnh cuboid: chọn công cụ Cuboid → chọn đúng label → bấm **Shape** (không bấm Track) → nháy đúp vào cụm điểm để đặt; chọn hộp để chỉnh tâm, dài/rộng và xoay `yaw` ở khung **Top**, chỉnh độ cao và đáy chạm đường ở khung **Side** và **Front** (không xoay ở Side/Front).
  3. Kiểm tra hướng và đối chiếu camera: bật **Appearance → Cuboid orientation** để căn vạch đỏ ngắn (`+X`) chỉ về đầu xe, bấm **Save (`Ctrl+S`)**, rồi chạy `python scripts/lab.py sync` để xem ảnh chiếu cuboid trên đủ 7 camera trong `work/xem/`.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ. Ảnh: `t1-s01f25-3d.png`, `t1-s01f25-top.png`, `t1-s01f49-3d.png`, `t1-s01f49-top.png` (ảnh nhìn từ trên xuống có đánh dấu phía trước `−y`, `+x`, `+y`).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f25` | Phía trước (`−y`, trùng `cam01`) ở **cạnh trên**; `+x` sang **trái**; `+y` hướng **xuống dưới** | Phẳng (trước `−1,75 m`, sau `−1,69 m`), thấp hơn LiDAR **~1,71 m** | ~50 m (xe ở bên kia ngã tư còn cụm điểm rõ; >60 m rất thưa) | Xe con (`Car`), xe tải nhỏ/van (`Truck`), xe buýt phía sau (`Bus`), người đi bộ (`Pedestrian`), người đi xe hai bánh (`Cyclist`) | SUV trắng bên phải (`cam07`, `cam08`), sedan trắng ngay phía sau-phải (`cam08`, `cam09`), xe buýt và nhiều xe con phía sau (`cam05`, `cam09`), xe con trắng cắt ngang bên trái (`cam05`, `cam06`), nhóm người đi bộ/xe hai bánh gần vạch qua đường bên phải-sau (`cam08`, `cam09`) | Cụm điểm dính sát nhau dọc vỉa hè bên phải (hàng xe đạp dựng san sát lẫn gốc cây, cột) và cụm điểm thưa ở xa `>55 m` bên kia ngã tư |
| `s01f49` | Phía trước (`−y`, trùng `cam01`) ở **cạnh trên**; `+x` sang **trái**; `+y` hướng **xuống dưới** | Phẳng (trước `−1,77 m`, sau `−1,78 m`), thấp hơn LiDAR **~1,77 m** | ~45–50 m (dòng xe phía trước và xe buýt xa gần giao lộ) | Xe con (`Car`), xe van/tải nhỏ (`Truck`), xe buýt ở xa phía trước (`Bus`), người đi xe đạp (`Cyclist`) ở làn xe hai bánh bên phải | Sedan trắng bám ngay phía sau cùng làn (`cam09`), sedan tối màu phía sau-trái (`cam05`, `cam09`), các người đi xe đạp trên làn riêng bên phải sau dải cây (`cam07`, `cam08`, `cam09`) | Các cụm điểm nhỏ, rời rạc ẩn sau hàng rào cây xanh ở dải phân cách bên phải (nếu chỉ nhìn point cloud khó tách người đi xe đạp với tán cây và xe đạp dựng) |

Đối chiếu từ `s01f25` và `s01f49`:
- **3 điều chỉ camera cho biết**: (1) Màu sắc xe và trạng thái đèn giao thông (đỏ/xanh) tại ngã tư ở `s01f25`; (2) Vạch kẻ làn, vạch qua đường, mũi tên chỉ hướng trên mặt đường; (3) Phân biệt xe hai bánh có người đang lái (`Cyclist`) với hàng xe đạp dựng không người dọc vỉa hè phải.
- **3 điều chỉ LiDAR cho biết**: (1) Khoảng cách chính xác theo mét từ tâm LiDAR đến từng vật (xác định rõ vật trong ngưỡng `30 m` / `50 m`); (2) Độ cao mặt đường (`z ≈ −1,7 m`) và kích thước thực 3D (dài, rộng, cao theo mét) của vật; (3) Cấu trúc hình học 360° liên tục quanh xe, bao gồm cả các vị trí nằm ở vùng giáp ranh/góc mù giữa các camera.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f31` | Cạnh trên (`−y`) | Hàng rào phân cách sắt sát bên trái tạo đường điểm dài liên tục; nhiều tán cây hai bên đường |
| `s02f53` | Cạnh trên (`−y`) | Xe con đỏ đi ngược chiều khá gần bên kia hàng rào phân cách bên trái; có cụm điểm lạ dưới mặt đường |
| `s03f32` | Cạnh trên (`−y`) | SUV trắng rất gần bên trước-phải; bên kia dải phân cách cây xanh bên trái có xe buýt đỏ đi ngược chiều |
| `s03f54` | Cạnh trên (`−y`) | Đường nhiều làn có hàng rào sắt bên phải, người đi xe hai bánh chạy ngoài hàng rào bị che khuất một phần |
| `s04f00` | Cạnh trên (`−y`) | Xe MPV/SUV tối màu áp sát góc trước-phải gây vùng bóng che LiDAR lớn về phía bên phải |
| `s04f47` | Cạnh trên (`−y`) | Hai xe con (đỏ và trắng) đi rất sát nhau ở làn trước-phải; hàng rào phân cách màu vàng dọc bên trái |
| `s05f06` | Cạnh trên (`−y`) | Ngã tư rộng có nhiều xe ở nhiều hướng; góc trước-trái có xe tối màu sát đầu xe, bên phải có cọc tiêu/rào chắn |
| `s05f40` | Cạnh trên (`−y`) | Xe buýt lớn đi ngay sát làn trước-trái chắn gần hết tia LiDAR phía trước-trái (bóng che rất rộng) |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f25`, `#1` | `Car` | Đúng (`266,0°`, chỉ đầu xe tiến cùng chiều `−y`) | Chạm đường (`z_bottom = −1,711 m`) | Có (Top/Side/Front + `cam08`, `cam09`) | Trong khoảng (`4,500 × 1,880 × 1,494 m`, chuẩn Sedan `3,8–5,0 × 1,70–1,90 × 1,40–1,60 m`) |
| (b) xe buýt hoặc xe tải | `s01f25`, `#17` | `Bus` | Đúng (`270,4°`, chỉ đầu xe tiến về phía xe thu dữ liệu) | Chạm đường (`z_bottom = −1,533 m`) | Có (Top/Side/Front + `cam05`, `cam09`) | Chiều rộng `2,650 m` gần chuẩn (`2,50–2,60 m`), dài `14,028 m` hơi dài hơn bus thường (`10,0–12,5 m`) và cao `2,717 m` hơi thấp hơn `3,00 m` do xe ở xa `35,64 m` chỉ thấy rõ phần trước |
| (c) người đi bộ hoặc xe hai bánh | `s01f25`, `#4` | `Cyclist` | Đúng (`349,0°`, chỉ theo hướng đầu xe máy đang dừng chờ) | Chạm đường (`z_bottom = −1,675 m`) | Có (Top/Side/Front + `cam08`, `cam09`) | Trong khoảng (`1,850 × 0,669 × 1,629 m`, chuẩn xe hai bánh có người `1,6–2,3 × 0,50–0,95 × 1,40–1,90 m`) |

- **Vòng hiệu chuẩn (`D13-HC485087`)**: Khi đối chiếu cách hiểu guideline ở đầu Bước 3, điểm dễ lệch nhất là hướng `yaw` của xe ngược chiều/xe cắt ngang (dễ bị ngược `180°` nếu không nhìn đèn pha/đèn hậu trên ảnh camera) và chiều dài mặt khuất phía sau tia LiDAR. Cách hiểu đã chốt theo guideline: luôn đối chiếu camera xác định đầu xe để căn vạch đỏ `+X`, giữ mặt gần bám sát cụm điểm, nới mặt khuất theo kích thước tham chiếu Bảng 4.3 và không xoay hộp ở khung Side/Front.
- **Các vật bỏ qua vì không đủ cơ sở**:
  - `s01f25` (phía trước xa `>40 m` bên kia ngã tư): vài xe con chỉ còn dưới chục điểm rời rạc, khó xác định đủ thể tích (`LOW_VISIBILITY`).
  - `s01f25` & `s01f49` (dọc vỉa hè bên phải): hàng xe đạp dựng sát nhau không có người lái (theo guideline không gán nhãn xe hai bánh không có người).
  - `s03f32` (phía sau bên phải sau hàng cây xanh, `>20 m`): cụm điểm bị tán cây che lẫn xe đỗ phía trong, không tách được hình dạng xe (`LOW_VISIBILITY` / `UNCERTAIN_SCOPE`).

## 4. Bốn hiện tượng chất lượng (Bước 4)
Mọi nguyên nhân bên dưới đều ghi nhận dưới dạng **giả thuyết**. Cả 4 ảnh đều lấy trực tiếp từ 10 frame trong task cá nhân `D13-2A202603244`.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s05f40` / `t4-1-bong-che.png` / frame cá nhân (`D13-2A202603244`) | Giả thuyết: thân xe buýt lớn `#1 Bus` (`9,93 m`) ở sát làn trước-trái chắn hoàn toàn các tia LiDAR phát ra theo góc phần tư trước-trái | Sau thân xe buýt xuất hiện một vùng hình quạt rộng gần như trống trơn không có điểm; xe `#3 Car` (`19,81 m`) và `#7 Car` (`28,92 m`) phía trước bị che khuất phần lớn thân xe | Bật cờ `occluded` cho `#3`, `#7` và vẽ cuboid bao trọn phần thân suy luận được; vùng trống phía sau xe buýt coi là vùng chưa quan sát được, không kết luận là đường trống |
| 2. Thưa ở xa | `s01f25` / `t4-2-thua-o-xa.png` / frame cá nhân (`D13-2A202603244`) | Giả thuyết: khoảng cách giữa các tia LiDAR 40 kênh mở rộng tuyến tính theo khoảng cách nên số tia trúng cùng một thân xe giảm rất nhanh khi ra xa | Xe `#18 Car` ở `35,33 m` chỉ còn `101 điểm` (mỏng hơn rất nhiều so với xe `#1 Car` ở `6,88 m` có `1888 điểm`), các xe xa hơn `45 m` bên kia ngã tư chỉ còn vài điểm lẻ | Với `#18 Car` còn thấy trên `15–20%` thể tích và rõ trên `cam09` thì vẽ kết hợp kích thước bảng 4.3; các xe xa hơn nữa quá ít điểm thì bỏ qua và ghi `LOW_VISIBILITY` |
| 3. Phản xạ thấp / ít điểm bất thường | `s01f49` / `t4-3-phan-xa-thap.png` / frame cá nhân (`D13-2A202603244`) | Giả thuyết: bề mặt sơn xe tối màu và kính cửa hấp thụ hoặc phản xạ gương tia laser hồng ngoại yếu hơn xe sơn sáng màu ở cùng tầm | Xe tối màu `#8 Car` ở `16,32 m` chỉ có `417 điểm` (thưa hơn rõ rệt so với xe ở khoảng cách tương đương như `s01f49 #7` ở `15,55 m` có `649 điểm` hoặc `s04f00 #5` ở `16,02 m` có `704 điểm`), vùng kính xe hầu như trống điểm | Chỉ ghi nguyên nhân là giả thuyết (không khẳng định chắc chắn); đối chiếu ảnh `cam01`/`cam03` để bao đủ chiều dài thân xe thay vì cắt cụt cuboid theo cụm điểm ít |
| 4. Điểm lạ | `s02f53` / `t4-4-diem-la.png` / frame cá nhân (`D13-2A202603244`) | Giả thuyết: tia LiDAR bị phản xạ đa đường (multipath) từ mặt đường/bề mặt phản xạ hoặc nhiễu cảm biến tạo tọa độ giả dưới mặt đất | Khi xoay nghiêng khung 3D thấy một cụm `87 điểm` nằm sâu dưới mặt đường quanh `(x ≈ 3,7 m; y ≈ 28,6 m; z ≈ −13,4 m)`, thấp hơn mặt đường (`z ≈ −1,7 m`) hơn `11 m` | Tuyệt đối không gán nhãn cụm điểm lạ này thành bất kỳ vật thể nào theo guideline |

## 5. Số từ cuboid của tôi (Bước 5)
Số liệu lấy trực tiếp từ `submission/D13-2A202603244-bangA.csv` và cột `points_in` của `submission/D13-2A202603244-tukiem.csv`.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f25`, `#1`, `Car` | `−0,964 m` | `1,494 m` | `−1,711 m` | `6,88 m` | `6,95 m` | `266,0°` | Chạm đường (lệch `0,01 m` so với mức chuẩn `−1,7 m`) |
| `s01f49`, `#4`, `Car` | `−1,055 m` | `1,494 m` | `−1,802 m` | `10,23 m` | `10,28 m` | `272,0°` | Chạm đường (lệch `0,10 m`, trong ngưỡng `±0,3 m`) |
| `s01f25`, `#17`, `Bus` | `−0,175 m` | `2,717 m` | `−1,533 m` | `35,64 m` | `35,64 m` | `270,4°` | Chạm đường (lệch `0,17 m`, trong ngưỡng `±0,3 m`) |

- **(a) Cuboid gần nhất và xa nhất**:
  - **Gần nhất**: `s05f06`, `#1` (`Car`) có `dist_m = 3,84 m` và khoảng cách 3D `r = 3,93 m` (`x = −3,458 m`, `y = 1,659 m`, `z = −0,865 m`, `5406 điểm`).
  - **Xa nhất**: `s03f32`, `#6` (`Bus`) có `dist_m = 35,74 m` và khoảng cách 3D `r = 35,74 m` (`x = −7,071 m`, `y = 35,034 m`, `z = −0,252 m`, `480 điểm`).
- **(b) So sánh kích thước với bảng 4.3**:
  - Người đi bộ `s01f25`, `#5` (`Pedestrian`): `l = 0,539 m`, `w = 0,650 m`, `h = 1,795 m` — cả 3 chiều đều nằm trong khoảng Người lớn (`0,4–0,9 m` × `0,5–0,9 m` × `1,50–1,95 m`).
  - Xe con `s01f25`, `#1` (`Car`): `l = 4,500 m`, `w = 1,880 m`, `h = 1,494 m` — cả 3 chiều đều nằm gọn trong khoảng Sedan (`3,8–5,0 m` × `1,70–1,90 m` × `1,40–1,60 m`). (Riêng ca đặc biệt `s05f40 #4 Car` có `l = 5,500 m`, `h = 2,450 m` nằm ngoài khoảng SUV/minivan `5,3 m` và `2,10 m` vì đây là xe van mái cao thân dài).
- **(c) Góc `yaw_deg` theo chiều đi**:
  - Xe cùng chiều phía trước `s01f49`, `#4` (`Car`): `yaw_deg = 272,0°` (rất gần `270°`, hướng `−y`).
  - Xe đi ngược chiều `s02f31`, `#2` (`Car`): `yaw_deg = 95,0°` (rất gần `90°`, hướng `+y`).

Mật độ điểm theo khoảng cách (4 xe `Car` ở 4 khoảng `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s04f47`, `#1`, `Car` | `7,70 m` | `1718` |
| `s04f00`, `#5`, `Car` | `16,02 m` | `704` |
| `s05f06`, `#8`, `Car` | `24,87 m` | `151` |
| `s01f25`, `#18`, `Car` | `35,33 m` | `101` |

- **Quy luật mật độ**: Khi khoảng cách tăng gấp `~2,1` lần (từ `7,70 m` lên `16,02 m`) thì số điểm chỉ còn khoảng `41%` (`1718` → `704` điểm); khi xa gấp `~3,2` lần (`24,87 m`) số điểm giảm xuống chỉ còn khoảng `9%` (`151` điểm), và ở `35,33 m` còn `101` điểm (`~6%`). Xu hướng giảm phi tuyến này bám sát đường cong trung vị trên slide bài giảng (xe con: `~1968` điểm ở `0–10 m`, `~718` ở `10–20 m`, `~214` ở `20–30 m`, `~72` ở `30–40 m`).
- **Tự kiểm và chỉnh sửa sau `preview` / `selfcheck`**:
  - **Các cuboid đã sửa**: (1) `s01f25`, `#18` (`Car`): phát hiện cờ lơ lửng (`z_bottom = −0,991 m`, cao hơn mặt đường `0,67 m`) nên đã hạ đáy xuống chạm đường `−1,660 m` (số điểm trong hộp tăng từ `45` lên `101` và hết cờ); (2) `s01f25`, `#17` (`Bus`): thu gọn chiều rộng từ `3,536 m` về `2,650 m` sát cụm điểm xe buýt; (3) `s03f54`, `#1` (`Car`): xoay lại `yaw_deg` từ `265,0°` về `269,6°` cho thẳng làn; (4) `s01f49`, `#4` (`Car`) và `s03f54`, `#5` (`Bus`): tinh chỉnh lại vị trí tâm và kích thước cho ôm khít thân xe hơn.
  - **Các cảnh báo giữ nguyên (4/91 hộp)**: `s01f49 #6 Pedestrian` (`51` điểm sát mép thuộc hàng xe đạp dựng ngay bên cạnh), `s03f32 #3 Pedestrian` (`193` điểm sát mép là cột đèn/biển báo sát vỉa hè), `s04f00 #8 Car` (`37` điểm sát mép là xe đỗ sát làn kế bên), và `s05f40 #4 Car` (`height = 2,45 m` vì đây là xe van chở khách mái cao nên không cắt thấp hộp xuống làm mất điểm phần nóc xe).
- **Đổi kiểm**: Đã đổi kiểm trên màn hình với bạn MSSV `2A202603270`, đối chiếu 2 con số: `s01f25 #1 Car` (`z_bottom = −1,711 m`, `dist_m = 6,88 m`) và `s05f06 #1 Car` (`dist_m = 3,84 m`, `points_in = 5406`) — cả hai đều khớp hoàn toàn với `D13-2A202603244-bangA.csv` và `D13-2A202603244-tukiem.csv`, không có sai lệch cần sửa.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
- **(a) Ý nghĩa từ phía model 3D**:
  1. Nhãn 3D phải đo bằng **mét** trong hệ tọa độ thực của xe (thay vì pixel trên ảnh 2D) vì hệ thống lập kế hoạch chuyển động và tránh va chạm của xe tự hành cần biết chính xác khoảng cách vật lý, kích thước chiếm chỗ và hướng di chuyển thực tế trong không gian 3D (một vật kích thước pixel như nhau có thể là xe con ở gần hoặc xe tải ở xa).
  2. Vật thưa điểm cần **quy tắc riêng** (chỉ vẽ khi thấy khoảng `15–20%` thể tích hoặc suy luận chắc chắn, còn lại bỏ qua và ghi `LOW_VISIBILITY`) vì nếu người gán nhãn đoán bừa vị trí/kích thước/hướng cho vài điểm nhiễu rời rạc, model học trên nhãn sai sẽ sinh ra các dự đoán ảo (false positive) hoặc lệch hướng nguy hiểm khi vận hành thực tế.
- **(b) Cách dùng AI**:
  - **Công cụ**: Antigravity AI Assistant.
  - **Câu hỏi chính**: Hướng dẫn lệnh PowerShell/Git/Docker/Python `.venv`, giải thích hệ tọa độ VF (`−y` phía trước, góc `yaw`), hỗ trợ tự động hoá setup môi trường và chạy các lệnh kiểm tra `dry`, `task`, `sync`, `pack`, `check_submission.py`.
  - **Cách dùng kết quả**: Áp dụng các lệnh cài đặt, kiểm tra lỗi format và tự rà soát lại toàn bộ cuboid trên CVAT cùng ảnh `work/xem/` trước khi chốt bản cuối.
- **(c) Điều còn chưa chắc**:
  - Góc `yaw` của xe đang rẽ tại ngã tư như `s01f25 #2 Car` (`229,0°`) có thể còn sai lệch nhẹ vài độ so với trục thân xe thực tế vì xe đang bo cua.
  - Phân loại xe van mái cao `s05f40 #4` nằm ở ranh giới giữa van chở người (`Car`) và van chở hàng (`Truck`), cần tham khảo thêm quyết định chốt của coach nếu có bản quy định chi tiết hơn cho dòng xe van đa dụng.
  - Chiều dài mặt khuất của các xe ở xa `>30 m` (như `s01f25 #17 Bus` ở `35,64 m`) vẫn phải suy luận từ kích thước chuẩn Bảng 4.3 do LiDAR chỉ quét được mặt phía trước.
