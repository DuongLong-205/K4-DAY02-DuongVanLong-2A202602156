# Phiếu quy tắc gán nhãn — Ngày 2

**Họ và tên:** Dương Văn Long<br>
**MSSV:** 2A202602156<br>
**Hình thức:** Cá nhân<br>
**Mã cặp:** Solo

## 1. Phạm vi

- Chỉ gán phương tiện thuộc bốn lớp bên dưới.
- Mỗi phương tiện là một hộp; không gộp nhiều xe.
- Không gán người, xe máy, xe đạp, biển báo hoặc phần phản chiếu.
- Vật thể quá nhỏ hoặc mờ đến mức không thể phân lớp có căn cứ: không đoán; ghi lý do vào nhật ký quyết định.

## 2. Bốn lớp cố định

| Mã | Lớp | Gán khi nhìn thấy | Không gán vào lớp này |
| ---: | --- | --- | --- |
| 0 | `car` (ô tô con) | sedan, hatchback, SUV, taxi, xe bán tải dùng như xe con | xe có thùng/ben rõ; thân xe buýt; xe van thân hộp |
| 1 | `truck` (xe tải) | thùng, ben, sàn hàng hoặc thiết bị công vụ rõ ràng | ô tô con; thân xe buýt; xe van kín một khối |
| 2 | `bus` (xe buýt) | thân xe khách dài, nhiều cửa sổ hoặc hàng ghế | xe van nhỏ; xe tải; ô tô con |
| 3 | `van` (xe van) | thân hộp nhỏ, kín, dùng chở người hoặc hàng | thân xe buýt; khoang hàng tách biệt như xe tải |

Thứ tự lớp là cố định: `0 car, 1 truck, 2 bus, 3 van`.

## 3. Hộp giới hạn

- Vẽ sát phần vật thể nhìn thấy.
- Không ước lượng phần bị xe khác che.
- Vật thể chạm mép ảnh vẫn được gán nếu đủ bằng chứng phân lớp.
- Không để hộp chứa nhiều nền hoặc nhiều phương tiện.

## 4. Ba thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
| --- | --- | --- |
| `visibility` (mức nhìn thấy) | `clear` (rõ), `occluded` (bị che), `unclear` (không rõ) | mức bằng chứng nhìn thấy |
| `boundary` (quan hệ mép ảnh) | `inside` (trong ảnh), `truncated` (bị cắt) | vật thể có bị mép ảnh cắt hay không |
| `review_state` (trạng thái xem lại) | `confident` (tự tin), `needs_review` (cần xem lại) | đánh dấu quyết định cần quay lại |

YOLO không lưu ba thuộc tính này. Vì vậy phải xuất thêm `CVAT for images 1.1` từ cùng công việc.

## 5. Ba tình huống mơ hồ

Hoàn thành trước khi xem bài của người khác hoặc bộ nhãn tham chiếu.

### Tình huống A — xe buýt hay xe van?

- Ảnh và mã vật thể: `drive_008` — xe vàng-trắng-xanh giữa ngã tư
- Dấu hiệu nhìn thấy: Thân dài, nhiều cửa sổ hàng ghế dọc thân xe, có hai cửa lên xuống, sơn vàng-trắng-xanh đặc trưng xe buýt thành phố, kích thước lớn hơn van rõ rệt
- Quy tắc áp dụng: Thân xe khách dài, nhiều cửa sổ hoặc hàng ghế → `bus` (mã 2); van có thân hộp nhỏ kín, không có nhiều cửa sổ hàng ghế
- Quyết định: Gán `bus`
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Phóng ảnh 100% kiểm đếm số cửa sổ hàng ghế; nếu vẫn không rõ thì đặt `review_state=needs_review` và hỏi Lab Coach về tiêu chí kích thước tối thiểu để phân biệt bus/van

### Tình huống B — xe tải hay xe van/ô tô con?

- Ảnh và mã vật thể: `drive_038` — xe trắng có chữ "公安" phía dưới ảnh
- Dấu hiệu nhìn thấy: Thân xe có thiết bị cẩu/công vụ gắn phía sau, cabin tách biệt rõ ràng, khoang hàng/thiết bị tách biệt khỏi phần cabin
- Quy tắc áp dụng: Có thùng, ben, sàn hàng hoặc thiết bị công vụ rõ ràng → `truck` (mã 1); van là thân hộp kín một khối không có khoang hàng tách biệt
- Quyết định: Gán `truck` — ban đầu gán nhầm `van`, sau đó phát hiện và sửa
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Phóng ảnh kiểm tra xem có thiết bị gắn nổi phía sau cabin không; nếu thân hoàn toàn kín một khối và không thấy khoang hàng tách biệt → giữ `van`; nếu không chắc → `needs_review`

### Tình huống C — bị che, bị mép ảnh cắt hay không đủ bằng chứng?

- Ảnh và mã vật thể: `drive_033` — xe nhỏ ở xa phía trên bên phải ảnh
- Dấu hiệu nhìn thấy khi phóng 100%: Vật thể rất nhỏ, hình dạng mờ, chỉ thấy phần mái và một phần thân xe, bị các xe khác che một phần
- Giá trị `visibility`: `unclear` — vật thể quá nhỏ/mờ để nhận diện rõ
- Giá trị `boundary`: `inside` — không bị mép ảnh cắt
- Trạng thái `review_state`: `needs_review` — không đủ bằng chứng phân biệt `car` hay `van`
- Lý do: Vật thể chiếm diện tích quá nhỏ, không nhìn rõ đặc điểm thân xe (đầu xe, cửa, thân hộp hay sedan). Sau khi đối chiếu với bộ tham chiếu (55 hộp của tôi không ghép được), có thể những vật thể loại này nên được bỏ qua thay vì gán với nhãn phỏng đoán.

## 6. Xác nhận tự kiểm tra

- [x] Đã rà đủ bốn ảnh.
- [x] Đã kiểm vật thể thiếu và trùng.
- [x] Đã kiểm lớp và hình học từng hộp.
- [x] Mỗi hộp có đủ ba thuộc tính.
- [x] Đã xử lý mọi hộp `needs_review`.
- [x] Đã hoàn thành ba tình huống trước khi xem nguồn đối chiếu.
- [ ] Nếu làm theo cặp, hai người đã xuất bài độc lập trước khi trao đổi.
- [x] Nếu làm cá nhân, bài riêng đã được kiểm trước khi nhận bộ tham chiếu.
- [x] Số vật thể thực tế: 103 — 40–60 là mục tiêu khối lượng, không phải điểm cắt. Số thực tế vượt mục tiêu vì gán cả các vật thể nhỏ/xa.
