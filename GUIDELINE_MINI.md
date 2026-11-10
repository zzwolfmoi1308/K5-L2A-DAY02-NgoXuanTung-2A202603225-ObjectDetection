# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe con, sedan, SUV, hatchback, crossover, taxi, xe bán tải (pickup truck). | Xe nhỏ 4-7 chỗ hoặc bán tải cabin nhỏ. Phân biệt với van (thân hộp) và truck (xe tải lớn/đầu kéo). |
| `truck` | Xe tải chở hàng, xe ben, xe bồn, xe đầu kéo container. | Thùng hàng lớn phía sau hoặc cabin tách rời rơ-moóc. Không gán xe bán tải cabin nhỏ vào đây. |
| `bus` | Xe buýt nội đô, xe khách liên tỉnh, minibus chở khách (> 16 chỗ). | Thân dài, nhiều hàng ghế/cửa sổ bên hông chở nhiều khách. Phân biệt với van chở người (9-16 chỗ). |
| `van` | Xe minivan, xe van thương mại thân hộp đóng kín chở hàng nhẹ hoặc chở người (9-16 chỗ). | Thân xe dạng khối hộp liền mạch (Ford Transit, Solati, minivan...). Không gán xe buýt lớn hoặc SUV/sedan. |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: Ôm khít đường viền thân xe nhìn thấy được (tightness); không bao gồm bóng đổ xe trên mặt đường, mặt đường bên dưới, bầu trời hay lấn sang xe bên cạnh.
- Vật thể bị che khuất: Chỉ vẽ bounding box bọc phần thân xe nhìn thấy được; tuyệt đối không ước lượng/vẽ đoán phần thân xe bị che khuất. Bỏ qua nếu phần thấy được quá nhỏ (vài pixel, chỉ có mẩu gương/bánh xe) không thể nhận diện được lớp.
- Vật thể bị cắt bởi rìa ảnh: Vẽ sát đường biên xe và kéo chạm sát đúng mép rìa ảnh; không vẽ ước đoán ra ngoài khung hình.
- Vật thể rất nhỏ/mơ hồ: Không gán xe ở quá xa, mờ nhòe pixel không rõ đặc trưng nhận dạng để tránh noise và false positive. Không gán xe máy, xe đạp, người đi bộ, hình ảnh xe trên biển quảng cáo hoặc hình ảnh phản chiếu qua gương/kính.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_005.jpg` | Xe cứu hộ giao thông (đầu cabin giống van nhưng đuôi có cần cẩu/càng kéo lộ thiên) | Gán là `truck` (ID: 1) | Phía sau có kết cấu cơ khí/cần cẩu chuyên dụng chở và kéo phương tiện, không có thân vỏ hộp kín liền khối chở người/hàng nhẹ như `van`, thuộc nhóm phương tiện vận tải chuyên dụng nên xếp vào `truck` (ID 1). |
| `drive_012.jpg` | Xe dáng cao cỡ nhỏ, phân vân giữa xe con (hatchback/compact) và minivan | Gán là `car` (ID: 0) | Kích thước xe nhỏ, chiều dài cơ sở ngắn, kiểu dáng hatchback đô thị 4-5 chỗ phục vụ cá nhân/gia đình (không phải dạng thân hộp lớn chở nhiều khách hay chở hàng thương mại của `van`), nên xếp vào `car` (ID 0). |
| `drive_019.jpg` | Phân vân giữa xe con và minivan | Gán là `van` (ID: 3) | Tuy kích thước không quá lớn như `bus`, nhưng xe có thiết kế thân hộp vuông vức và nhìn rõ rãnh ray cửa trượt (sliding door) ở sườn xe, đặc trưng của dòng minivan thương mại chở hàng/khách nhẹ (khác với xe con hatchback ở `drive_012`), nên xếp vào `van` (ID 3). |
