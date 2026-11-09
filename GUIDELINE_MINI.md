# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Các dòng xe chở người 4–7 chỗ gồm sedan, hatchback, SUV, crossover, taxi và xe bán tải (pickup). | Xe con cá nhân, xe taxi; xe bán tải nhỏ có cabin xếp vào `car` (không xếp vào `truck`). |
| `truck` | Các loại xe tải chở hàng hạng trung và nặng, xe tải thùng kín/hở, xe đầu kéo container, xe bồn. | Thùng chở hàng tách rời cabin lái, kích thước to lớn chuyên chở hàng hóa nặng. |
| `bus` | Xe buýt nội đô công cộng, xe khách liên tỉnh, minibus chở khách lớn trên 16 chỗ. | Thân xe dài hình khối chữ nhật lớn, nhiều cửa sổ kính hai bên sườn, sàn xe cao. |
| `van` | Xe van, minivan chở hàng hoặc chở người 9–16 chỗ (Ford Transit, Toyota Hiace, Hyundai Starex). | Thân hình hộp chữ nhật liền khối; ngắn và thấp hơn `bus`, không có thùng hàng rời như `truck`. |

## Quy tắc chung

- **Box bám sát vật thể nghĩa là:** Bốn cạnh của bounding box phải tiếp xúc sát nhất với các điểm cực biên ngoài cùng của xe (mũi xe, cản sau, mép gương chiếu hậu, điểm đáy lốp xe tiếp xúc mặt đường). Không để thừa nền quá 2–3 pixel và tuyệt đối không cắt lẹm vào thân xe.
- **Vật thể bị che khuất:** Vẽ box bao trọn phần nhìn thấy được (visible part) của vật thể nếu phần nhìn thấy vẫn đủ để nhận diện phân lớp chắc chắn; không tự vẽ ước lượng phần bị che khuất.
- **Vật thể bị cắt bởi rìa ảnh:** Cạnh của box phải chạm khít đúng mép ảnh; không vẽ tràn ra ngoài tọa độ ảnh và không đoán phần thân xe nằm ngoài khung hình.
- **Vật thể rất nhỏ/mơ hồ:** Bỏ qua các đốm ảnh quá nhỏ (kích thước nhỏ hơn 10x10 pixel) hoặc mờ nhòe hoàn toàn không thể phân biệt loại phương tiện để tránh tạo ra false positives.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_001.jpg` | Hai xe đi nối đuôi sát nhau ở làn giữa (vùng x≈240, y≈210), xe sau che khuất một phần đầu xe trước. | Vẽ hai bounding box độc lập ôm phần nhìn thấy của từng xe. | Schema yêu cầu mỗi xe hợp lệ là một box riêng; không được gộp hai xe thành một box duy nhất. |
| `drive_001.jpg` | Phương tiện ở rất xa cuối làn đường (vùng x≈618, y≈127) kích thước chỉ khoảng 13x14 pixel. | Vẫn gán `car` vì vẫn nhìn rõ hình bóng xe sedan và đèn hậu. | Vật thể ở xa nhưng vẫn nhận diện được rõ ràng loại phương tiện thì phải gán để tránh lỗi bỏ sót (False Negative). |
| `drive_005.jpg` | Xe bán tải (pickup truck) có thùng chở hàng phía sau gây phân vân giữa `car` và `truck`. | Gán nhãn là `car` (Class ID: 0). | Schema quy định rõ ràng: "xe bán tải/cabin nhỏ thuộc car, không gán vào truck". |
