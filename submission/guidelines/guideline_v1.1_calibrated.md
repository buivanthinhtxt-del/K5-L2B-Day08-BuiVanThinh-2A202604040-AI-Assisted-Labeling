# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202604040 - Bùi Văn Thịnh

> **Chỉ viết phần thay đổi so với v1.0**. Đúc kết các quyết định giải quyết câu hỏi mở ở mục 6 của v1.0 và các quy tắc mới rút ra từ lỗi của máy, bẫy lọt và quan sát thực tế trong quá trình review 24 ảnh batch.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | Gán là `motorbike` nếu có động cơ điện và tốc độ di chuyển tương đương xe máy; chỉ gán `bicycle` khi thuần đạp chân hoặc xe trợ lực có bàn đạp rõ ràng. | `img_003.jpg` | CAL-02 |
| 2 | Khối xe máy đỗ dày đặc ở xa: từng xe hay `ignore_region`? | Bắt buộc vẽ 1 box `ignore_region` bao quanh cụm nếu cụm có $\ge 5$ xe và chiều cao mỗi xe $< 20\text{px}$. Không vẽ lẻ từng xe để tránh tranh cãi sai số. | `img_018.jpg`, `img_024.jpg` | CAL-02 |
| 3 | Hàng nhô ra khỏi xe ba gác / xe đạp: box xe có gồm hàng không? | Nếu hàng gắn cố định trong thùng xe: box xe ôm cả hàng. Nếu hàng nhô dài ra ngoài xe $> 20\%$ chiều dài thân xe (ví dụ thanh tre, sắt): chỉ vẽ box xe ôm thân xe, không kéo dài theo phần hàng nhô. | `img_004.jpg`, `img_020.jpg` | CAL-01 |
| 4 | Người ngồi trong xe ba bánh có mui che: gán `person` khi nào? | Gán `person` khi nhìn thấy $\ge 20\%$ phần thân trên (đầu, vai, ngực). Nếu chỉ thấy thoáng một phần cánh tay hoặc bóng mờ qua khe kính: không gán `person`. | `img_013.jpg` | CAL-01 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_013.jpg`, `img_004.jpg` | Xe tuk-tuk và xe ba gác chở hàng trên phố | YOLO26 gọi nhầm thành `car` hoặc `truck` | Đổi toàn bộ thành `three_wheeler`; tách box `person` riêng cho tài xế và khách | **Mục 2 & 5:** Mọi phương tiện 3 bánh đặc thù địa phương (tuk-tuk, cyclo, xe ba gác) đều thuộc `three_wheeler`. |
| `img_025.jpg` | Xe ba gác máy chở hàng bị che một phần | Bẫy đổi nhãn thành `truck` | Đổi lại `three_wheeler` dựa vào quy tắc đếm bánh xe | **Mục 4.3:** Quan sát số bánh xe nhìn thấy: 3 bánh $\rightarrow$ `three_wheeler`; $\ge 4$ bánh $\rightarrow$ `truck`. |
| `img_028.jpg`, `img_029.jpg` | Bẫy box ảo (`extra`) trên lề đường và box lỏng (`loose`) | Máy sinh box ma không có vật thật; box bao quá rộng | Xóa box ma; co 4 cạnh ôm khít đối tượng $\le 2\text{px}$ | **Quy trình review:** Bắt buộc ẩn nhãn máy trước để quét ảnh bằng mắt trần trước khi bật nhãn máy lên kiểm tra. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên viết sẵn | — |
| v1.1 | Bổ sung quy tắc phân loại xe ba bánh, quy định chuẩn cho cụm xe máy nhỏ bằng `ignore_region`, chốt quy tắc hàng chở nhô ra và nhận diện người trong xe mui kín; đưa ra checklist chống thiên kiến tự động hóa. | Khắc phục lỗi nhận nhầm của YOLO26, chốt các câu hỏi mở mục 6 của v1.0 và sửa các bẫy lọt (`CAL-01`, `CAL-02`, `CAL-03`). |
