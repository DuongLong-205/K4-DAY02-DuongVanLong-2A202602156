# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** Dương Văn Long<br>
**MSSV:** 2A202602156<br>
**Hình thức:** cá nhân<br>
**Mã cặp:** SOLO

## 1. Bài độc lập và nguồn dữ liệu

- Mã SHA-256 của ZIP ảnh được cấp: `f7d99888f21440fb0374d84962b93213bd8c14e665d093cc8d37f4c61b71ed33`
- Bốn mã ảnh: `drive_022`, `drive_033`, `drive_038`, `drive_008`
- Số vật thể thực tế: 103 (ngoài mục tiêu 40–60; ghi số thật, không vẽ ẩu để đủ số)
- Mã SHA-256 của gói YOLO của bạn: `332225c985ac59d9ec68b56a2475c4dc17c4053239ed613c4f03bf97eb9afb0a`
- Mã SHA-256 của gói CVAT gốc của bạn: `345ba9b313a2d2576cc06670966ebbc840934dc0b519e4be201944c8af195d41`
- Nguồn đối chiếu: bộ tham chiếu do người hướng dẫn thực hành (Lab Coach) cung cấp
- Mã SHA-256 của gói đối chiếu: `c8bbc767d8bb9a29f4ca5abf0c3516e5c2af94c58143a980b0148cfe0b500d2b`
- Nếu làm cá nhân, ghi mã lần phát và thời điểm nhận bộ tham chiếu: 2026-09-14 09:57 (thời điểm xuất gói từ CVAT)

Giải thích vì sao bài của bạn vẫn độc lập trước khi đối chiếu:

Tôi đã hoàn thành toàn bộ bốn ảnh, tự kiểm tra phạm vi, lớp, hình học và thuộc tính, sau đó xuất cả hai gói (YOLO và CVAT gốc) và ghi mã SHA-256 của gói YOLO trước khi nhận bộ tham chiếu. Trong suốt quá trình gán nhãn, tôi không xem bài của bất kỳ ai khác và không sử dụng kết quả dự đoán tự động làm đáp án. Bản xuất đã được khóa bằng mã SHA-256 nên không thể sửa lại mà không thay đổi mã kiểm.

## 2. Quyết định phân lớp

| Ảnh/vật thể | Lớp | Dấu hiệu nhìn thấy | Quy tắc áp dụng |
| --- | --- | --- | --- |
| drive_022 — xe lớn vàng-trắng gi  ữa đường | `bus` | Thân dài, nhiều cửa sổ hàng ghế, hai cửa lên xuống, sơn vàng-trắng đặc trưng xe buýt thành phố | Thân xe khách dài, nhiều cửa sổ hoặc hàng ghế → `bus` (mã 2) |
| drive_008 — xe đỏ chở đất giữa ngã tư | `truck` | Thùng ben mở chứa vật liệu xây dựng, cabin tách biệt rõ ràng, kích thước lớn hơn xe con nhiều lần | Có thùng, ben, sàn hàng hoặc thiết bị công vụ rõ ràng → `truck` (mã 1) |
| drive_038 — xe trắng có chữ "公安" phía dưới ảnh | `truck` | Thân xe có thiết bị cẩu/công vụ gắn phía sau, cabin rõ ràng, có ghi chữ "公安" (công an) | Có thiết bị công vụ rõ ràng → `truck` (mã 1); không phải van vì khoang hàng tách biệt |
| drive_033 — xe hộp trắng nhỏ giữa làn | `van` | Thân hộp kín một khối, không có thùng hàng tách biệt, kích thước nhỏ hơn xe buýt, không thấy nhiều cửa sổ hàng ghế | Thân hộp nhỏ, kín → `van` (mã 3); không phải truck vì không có khoang hàng tách biệt |
| drive_008 — xe xám nhỏ ở góc dưới phải | `car` | Dáng sedan, thấp, bốn cửa, không có thùng hay thiết bị đặc biệt | Sedan → `car` (mã 0) |

Nêu một ví dụ cho thấy lớp và thuộc tính là hai loại thông tin khác nhau:

Trong ảnh `drive_008`, xe buýt vàng-trắng-xanh được gán lớp `bus` (mã 2) vì có thân dài và nhiều cửa sổ hàng ghế. Đồng thời, thuộc tính `visibility` được gán `clear` (nhìn rõ toàn bộ thân xe), `boundary` là `inside` (không bị mép ảnh cắt), và `review_state` là `confident`. Lớp cho biết **loại phương tiện** (bus), còn thuộc tính cho biết **điều kiện quan sát** (nhìn rõ, nằm trong ảnh). Một xe bus bị cắt bởi mép ảnh vẫn là `bus` nhưng sẽ có `boundary=truncated` — lớp không thay đổi, chỉ thuộc tính thay đổi.

## 3. Tự kiểm tra và sửa nhãn

| Trước khi sửa | Loại lỗi | Cách phát hiện | Sau khi sửa và quy tắc |
| --- | --- | --- | --- |
| drive_038 — xe "公安" gán là `van` | lớp | Phóng ảnh 100%, thấy rõ thiết bị cẩu/công vụ phía sau cabin, khoang hàng tách biệt | Đổi thành `truck`; quy tắc: có thiết bị công vụ rõ ràng → `truck` |
| drive_033 — xe đỏ góc dưới trái hộp quá rộng | hình học | Rà lại hộp, thấy hộp bao gồm cả phần lan can đường và xe bên cạnh | Thu hộp sát phần xe nhìn thấy; quy tắc: vẽ sát phần vật thể, không đoán phần bị che |
| drive_022 — xe trắng bên phải thiếu thuộc tính `boundary` | thuộc tính | Lọc theo thuộc tính, thấy hộp chưa được gán `boundary` | Gán `boundary=truncated` vì xe bị mép phải ảnh cắt; quy tắc: vật thể chạm mép ảnh → `truncated` |

- Số hộp `needs_review` trước và sau khi kiểm: trước: 5 hộp; sau: 1 hộp (xe nhỏ xa trong ảnh `drive_033` không đủ chi tiết phân biệt car/van)
- Một quyết định chưa đủ bằng chứng và cách bạn xin hỗ trợ: Trong ảnh `drive_033`, có một xe nhỏ ở xa phía trên bên phải, hình dạng mờ không rõ là `car` hay `van`. Tôi đặt `review_state=needs_review` và `visibility=unclear`, sau đó hỏi Lab Coach xem có nên bỏ qua vật thể quá nhỏ/mờ không thể phân lớp có căn cứ hay vẫn gán với nhãn phỏng đoán.

## 4. Một dòng nhãn YOLO

- Dòng `class x_center y_center width height`: `2 0.392984 0.72625 0.434219 0.34`
- Tên lớp và tọa độ điểm ảnh `xyxy`: Lớp `bus` (mã 2). Tọa độ chuẩn hóa → điểm ảnh (ảnh 640×640): x_center = 0.392984 × 640 ≈ 251.5, y_center = 0.72625 × 640 ≈ 464.8, width = 0.434219 × 640 ≈ 277.9, height = 0.34 × 640 = 217.6. Tọa độ `xyxy`: x1 ≈ 251.5 − 277.9/2 ≈ 112.6, y1 ≈ 464.8 − 217.6/2 ≈ 356.0, x2 ≈ 251.5 + 277.9/2 ≈ 390.5, y2 ≈ 464.8 + 217.6/2 ≈ 573.6. Vậy hộp có tọa độ điểm ảnh **(112.6, 356.0, 390.5, 573.6)**.
- Vì sao dòng đúng định dạng vẫn có thể sai lớp, phạm vi hoặc hình học?

Một dòng nhãn YOLO chỉ chứa năm số — mã lớp và bốn tọa độ chuẩn hóa — nên chỉ cần đúng cú pháp là hợp lệ về mặt định dạng. Tuy nhiên, dòng đó vẫn có thể sai về: (1) **lớp** — người gán chọn nhầm `van` thay vì `truck` dù vật thể có khoang hàng tách biệt; (2) **phạm vi** — vẽ hộp cho xe máy vốn không thuộc bốn lớp cần gán, hoặc bỏ sót vật thể hợp lệ; (3) **hình học** — hộp quá rộng bao gồm cả nền hoặc xe khác, hoặc hộp đoán phần bị che khuất. Định dạng đúng không đảm bảo nội dung đúng.

## 5. Huấn luyện và dự đoán thử

- Ba mã ảnh huấn luyện: `drive_022`, `drive_033`, `drive_038`
- Mã ảnh thẩm định: `drive_008`
- Mô tả một dự đoán trong `detect_result.jpg`: Mô hình dự đoán rất kém trên ảnh thẩm định `drive_008` với mAP50 chỉ đạt 0.00829, Precision 0.00246 và Recall 0.167. Mô hình gần như không phát hiện được vật thể nào chính xác trên 32 instances của ảnh val. EarlyStopping kích hoạt sau epoch 4 (patience=3) vì không cải thiện so với epoch 1. Điều này cho thấy 3 ảnh huấn luyện là quá ít để mô hình học được đặc trưng phân biệt bốn lớp.
- Dự đoán đó gợi ý cần kiểm lại quy tắc hoặc dữ liệu nào? Kết quả cực thấp không phản ánh chất lượng nhãn mà phản ánh giới hạn của tập huấn luyện quá nhỏ (3 ảnh). Cần kiểm tra: (1) phân bố lớp trong tập huấn luyện — drive_022 chỉ có 5 objects, drive_033 có 27, drive_038 có 39 — mất cân bằng rõ; (2) lớp `van` chỉ xuất hiện ở drive_038 (3 mẫu) và drive_008 (3 mẫu) nên mô hình không có đủ mẫu van trong tập train.
- Minh chứng nào có thể bác bỏ nhận định của bạn? Nếu tôi cho rằng mAP thấp do thiếu dữ liệu huấn luyện, minh chứng bác bỏ là: huấn luyện trên tập lớn hơn (ví dụ 100+ ảnh) với cùng nhãn và thấy mAP vẫn thấp — khi đó nguyên nhân có thể là nhãn sai hoặc quy tắc phân lớp không nhất quán. Ngoài ra, nếu freeze 10 layers đầu gây ra bottleneck, thì unfreeze thêm layers có thể cải thiện — khi đó nguyên nhân là cấu hình huấn luyện, không phải dữ liệu.
- Vì sao kết quả trên bốn ảnh không phải phép đánh giá mô hình dùng thực tế?

Bốn ảnh là bộ dữ liệu quá nhỏ và có thể liên quan về thời gian (cùng nguồn UA-DETRAC). Mô hình huấn luyện trên ba ảnh và dự đoán trên một ảnh không đại diện cho sự đa dạng của giao thông thực tế (thời tiết, góc camera, mật độ xe khác nhau). Kết quả mAP hoặc ảnh dự đoán chỉ kiểm tra đường ống dữ liệu có hoạt động hay không, không chứng minh mô hình có khả năng phát hiện vật thể chính xác ngoài thực tế. Dùng số đo này để đánh giá khả năng dùng thực tế hoặc để chấm điểm người gán nhãn là không hợp lệ.

## 6. Đối chiếu nhãn

- Số hộp ghép được: 48
- IoU trung bình và trung vị: mean_iou = 0.735325, median_iou = 0.731171
- Mức đồng thuận lớp: 72.92% (class_agreement = 0.729167)
- Số hộp phía bạn không ghép được: 55 (unmatched_mine)
- Số hộp phía đối chiếu không ghép được: 2 (unmatched_comparison)
- Một điểm khác biệt cụ thể: Tôi gán 103 hộp trong khi bộ tham chiếu chỉ có khoảng 50 hộp (48 ghép được + 2 không ghép). 55 hộp của tôi không ghép được cho thấy tôi gán nhiều vật thể nhỏ/xa/mờ mà bộ tham chiếu không gán — có thể tôi đã gán quá nhiều vật thể không đủ bằng chứng phân lớp. Trong số 48 hộp ghép được, chỉ ~73% đồng thuận lớp, tức khoảng 13 hộp bị khác lớp — cần rà lại quyết định phân lớp giữa `car`/`van` và `truck`/`van`.
- Quy tắc hoặc hành động sửa phát sinh: (1) Bổ sung ngưỡng kích thước: vật thể chiếm dưới ~15×15 pixel (hoặc quá mờ để phân biệt dấu hiệu lớp) → không gán, ghi lý do vào nhật ký; (2) Rà lại tiêu chí phân biệt `car`/`van`: nếu không thấy rõ thân hộp kín đặc trưng → mặc định `car`; (3) IoU trung bình 0.735 cho thấy hộp giới hạn của tôi chưa sát vật thể — cần vẽ hộp chặt hơn, không bao gồm nền.
- Vì sao mức đồng thuận cao không chứng minh mọi nhãn đều đúng?

Mức đồng thuận cao giữa hai bộ nhãn chỉ cho thấy hai người áp dụng quy tắc tương tự nhau, tức là quy tắc có tính tái lập. Tuy nhiên, cả hai có thể cùng hiểu sai quy tắc hoặc cùng bỏ sót vật thể giống nhau, dẫn đến đồng thuận cao nhưng cả hai đều sai. Ví dụ: nếu cả hai đều không gán những xe nhỏ ở xa, IoU giữa các hộp ghép được sẽ cao nhưng tập nhãn vẫn thiếu vật thể. Đồng thuận đo mức nhất quán giữa người gán, không đo mức chính xác so với thực tế.

## 7. Kiểm tra kho GitHub cá nhân

- [x] Có phiếu quy tắc với ba tình huống mơ hồ.
- [x] Có kết quả kiểm hai gói xuất.
- [x] Có thông tin lần huấn luyện và ảnh dự đoán.
- [x] Có tóm tắt, bảng và ảnh phủ của bước đối chiếu.
- [x] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình.
- [x] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập.

Minh chứng mạnh nhất trong bài và câu hỏi còn lại cho Lab Coach:

Minh chứng mạnh nhất là bước đối chiếu nhãn (mục 6): với 55 hộp không ghép được (tôi gán 103, bộ tham chiếu ~50), tôi nhận ra mình đã gán quá nhiều vật thể nhỏ/xa/mờ không đủ bằng chứng phân lớp. Kết hợp với mức đồng thuận lớp chỉ 72.9% trên 48 hộp ghép được, tôi rút ra hai bài học: (1) cần ngưỡng kích thước tối thiểu để tránh gán phỏng đoán, và (2) cần phân biệt rõ hơn giữa `car`/`van`. Bước tự sửa nhãn (mục 3) — phát hiện và sửa lỗi phân lớp xe "公安" từ `van` sang `truck` — cũng cho thấy khả năng áp dụng quy tắc có hệ thống.

Câu hỏi cho Lab Coach: (1) Với những vật thể ở xa và mờ, tiêu chí nào để quyết định "đủ bằng chứng phân lớp" hay "quá mờ nên bỏ qua"? Có ngưỡng kích thước pixel tối thiểu nào nên áp dụng không? (2) IoU trung bình 0.735 cho thấy hộp của tôi chưa sát — ngoài việc vẽ chặt hơn, có kỹ thuật nào để tự kiểm tra hình học hộp trước khi đối chiếu không?
