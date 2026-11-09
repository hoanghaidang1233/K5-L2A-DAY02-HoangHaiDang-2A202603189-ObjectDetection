# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe ô tô con từ 4 đến 7 chỗ. | Dáng sedan, hatchback, SUV; chở người cỡ nhỏ. |
| `truck` | Xe tải chở hàng. | Có cabin riêng biệt phía trước và thùng chở hàng phía sau. |
| `bus` | Xe buýt, xe khách cỡ lớn. | Dáng dài, hình hộp lớn, nhiều hàng ghế/cửa sổ. |
| `van` | Xe thùng kín hoặc minivan dáng hộp liền khối. | Thân xe dạng hộp liền khối giữa khoang lái và khoang sau. |

## Quy tắc chung

- **Box bám sát vật thể nghĩa là:** Ôm khít viền ngoài cùng nhìn thấy của thân xe và bánh xe, không lấy thừa nền đường hay bầu trời.
- **Vật thể bị che khuất:** Chỉ vẽ bao quanh phần thân xe thực tế nhìn thấy được. Bỏ qua nếu bị che quá nhiều không rõ loại xe.
- **Vật thể bị cắt bởi rìa ảnh:** Kéo mép box chạm đúng mép ảnh, không tự vẽ đoán phần nằm ngoài ảnh.
- **Vật thể rất nhỏ/mơ hồ:** Bỏ qua xe ở quá xa mờ nhạt hoặc ảnh phản chiếu trên kính.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_015.jpg` | Nhiều xe đỗ sát nhau bên đường bị đuôi xe phía trước che khuất một phần đầu xe phía sau. | Vẽ từng box riêng ôm phần nhìn thấy của từng xe, không gộp hai xe thành một box. | Mỗi xe là một instance độc lập; xe đỗ vẫn phải gán nhãn đầy đủ. |
| `drive_027.jpg` | Xe dạng minivan di chuyển ở xa, khó phân biệt giữa `car` (SUV) và `van`. | Nếu thân xe dáng hộp vuông liền khối và nóc cao thì gán `van`, nếu dáng thuôn thấp 5-7 chỗ thông thường thì gán `car`. | Giữ nhất quán theo hình dáng khối hộp đặc trưng của van. |
| `drive_042.jpg` | Xe tải/xe con bị dải phân cách hoặc cột đèn che cắt ngang thân xe. | Vẽ box bao phần thân xe nhìn thấy, không kéo box bao trùm cả cột đèn; bỏ qua xe máy và người đi bộ bên cạnh. | Chỉ gán đúng 4 lớp trong schema; loại bỏ vật cản và đối tượng ngoài danh mục. |
