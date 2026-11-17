# Báo cáo Day 08 — 2A202604040 - Bùi Văn Thịnh

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết trung thực theo kết quả thực nghiệm.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26-s | 0.3 (mặc định) | Nhận diện nhanh (0.96s/ảnh), độ phủ recall tốt với người và xe máy trên đường phố đông đúc (`r01`, `r02`). bicycle/bus/truck có < 5 vật trên gold nên giữ mặc định theo khuyến nghị của `threshold_sweep`. |
| three_wheeler | YOLOE-s, prompt: `tuk-tuk, cyclo, cargo tricycle` | 0.15 | YOLO thuần không nhận diện được xe ba bánh; YOLOE bắt trúng 8 xe ba bánh ở tập pilot (`r03`). Ngưỡng 0.15 đạt recall 100% (7/7 vật gold) và precision 78% theo `threshold_sweep.md`. |

- Lệnh `merge_prelabels.py` đã dùng:
  ```bash
  python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
  ```
- **Phiên bản SAM bạn chọn:** SAM 2.1 (`sam2.1_b.pt`). Lý do: Cho đường biên polygon cực kỳ mịn và ôm sát vật thể phức tạp (`r04`), thời gian chạy trên CPU chấp nhận được (~13s/ảnh), ổn định hơn SAM 1 và không đòi hỏi quyền truy cập gated model trên HuggingFace như SAM 3.
- Lỗi nào **không sửa được bằng đổi ngưỡng**: Xe tuk-tuk và xe ba gác bị YOLO gọi là `car` hoặc `truck`. Nguyên nhân do kiến trúc bộ nhãn huấn luyện COCO không có lớp `three_wheeler`. Cách sửa: Dùng model open-vocabulary (YOLOE) với prompt chữ chuyên biệt và gộp nhãn bằng IoU matching.
- Box **điểm cao mà sai**: `img_002.jpg` có xe tuk-tuk bị YOLO26 gán nhầm thành `car` với điểm tin cậy rất cao (conf 0.82). Box **điểm thấp mà đúng**: `img_032.jpg` xe xích lô được YOLOE phát hiện ở ngưỡng conf 0.18 nhưng vị trí box ôm sát xe thật.
- Giới hạn: Quét ngưỡng tối ưu trên chính 4 ảnh gold (in-sample) nên các chỉ số precision/recall của máy trên gold mang tính lạc quan; mẫu gold nhỏ (4 ảnh) chỉ là ước lượng thô ban đầu.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 489 | 489/553 = 88.4% | 553 box máy |
| edited (chỉnh sửa) | 8 | 8/553 = 1.4% | 553 box máy |
| deleted (xóa bỏ) | 56 | 56/553 = 10.1% | 553 box máy |
| added (vẽ thêm) | 39 | 39/536 = 7.3% | 536 đối tượng sau duyệt |

- Lớp máy tệ nhất: `three_wheeler` bị deleted 38 box do YOLOE ở ngưỡng thấp 0.15 sinh ra box trùng lặp hoặc box quá lỏng ở các góc che khuất; lớp `person` máy bỏ sót 15 người ở xa và bị che khuất một phần (ví dụ `img_018.jpg`, `img_024.jpg`).
- So với gold (4 ảnh gold, 63 đối tượng gold): Precision của máy đạt **77.0%** (47/61, KTC 95%: 65–86%), Recall của máy đạt **74.6%** (47/63, KTC 95%: 63–84%).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **7/12 = 58.3%** (KTC 95%: 32–81%).
  - wrong_class: bắt được 2/3 (lọt `img_025.jpg`).
  - extra: bắt được 2/3 (lọt `img_028.jpg`).
  - loose: bắt được 1/3 (lọt `img_018.jpg`, `img_029.jpg`).
  - missing: bắt được 2/3 (lọt `img_026.jpg`).
- **Phân tích nguyên nhân:** Tỉ lệ accepted cao (88.4%) nhưng bắt bẫy chỉ đạt 58.3% là minh chứng rõ rệt của **Thiên kiến tự động hóa (Automation Bias)**. Người duyệt có tâm lý chủ quan, thấy khung box AI vẽ sẵn trông khá ngay ngắn thì bấm lướt qua mà không soi kỹ đường biên (khiến lọt 2 bẫy `loose`). Bẫy `missing` bị lọt ở `img_026.jpg` do người duyệt bỏ qua bước 0 của checklist (quét mắt trần trước khi bật nhãn máy).

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): Gán tay độc lập 4 ảnh gold mất 10 phút (**2.5 phút/ảnh**); Duyệt 24 ảnh có pre-label máy mất 24 phút (**1.0 phút/ảnh**). Tốc độ tăng hơn **2.5 lần**.
- Kết luận: **RẤT ĐÁNG DÙNG** cho các lớp đối tượng phổ biến (`person`, `motorbike`, `car`) vì tiết kiệm hơn 60% thời gian vẽ tay. Tuy nhiên, với các đối tượng đặc thù địa phương (`three_wheeler`), cần luôn bổ sung model chuyên biệt và quy trình review nghiêm ngặt.
- **Ba quy tắc giảm thiên kiến:**
  1. *Quy tắc 5 giây mắt trần (Bắt buộc ẩn nhãn máy):* Khi chuyển sang ảnh mới, luôn ẩn toàn bộ box máy trong 5 giây đầu để tự đếm số người và xe chính.
  2. *Quy tắc rà soát biên phóng đại 200%:* Phóng to các xe ở gần để kiểm tra 4 cạnh box có chạm mép ngoài cùng của vật trong khoảng $\le 2\text{px}$ hay không.
  3. *Quy tắc đếm bánh xe ba bánh:* Khi gặp xe có mui hoặc xe chở hàng, kiểm tra số bánh xe nhìn thấy (3 bánh $\rightarrow$ `three_wheeler`).

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click / Box | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 2 pos (`400,480`, `600,110`), 1 neg (`720,300`) | Ôm trọn thân xe tuk-tuk và khung mui; không dính nền đường phía sau. | Phần kính chắn gió phía trước bị thủng nhẹ do phản chiếu ánh sáng. | `tuktuk_img013.jpg` |
| `img_004.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 1 box prompt (`474,229,834,596`) | Ôm trọn xe tuk-tuk đỗ bên lề đường; ôm sát phần bánh xe và mui bạt. | Vùng bóng đổ dưới gầm xe bị ăn nhẹ vào mask khoảng 3px. | `tuktuk_img004.jpg` |
| `img_002.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 1 box (`371,414,736,658`) + 1 pos (`550,530`) | Ôm trọn thân xe ba bánh chở hàng; tách biệt rõ với người đi xe máy bên cạnh. | Tay lái xe có chi tiết mảnh bị bo tròn nhẹ. | `tuktuk_img002.jpg` |

- Đối chiếu guideline mục 4.2: Quy tắc không vẽ lỗ thủng nhỏ $< 5\%$ diện tích giúp mask của SAM không bị rách nát ở các thanh sắt giàn mui. Quy tắc bỏ bóng đổ cần chú ý thêm vì SAM có xu hướng ăn theo bóng xe dưới mặt đường nhựa.

## 6. Calibration (Part 6)

- Số quyết định: **3 quyết định** ghi nhận trong `submission/qc/calibration_decisions.csv` (`CAL-01`, `CAL-02`, `CAL-03`). Quyết định quan trọng nhất là `CAL-01`: Quy định chuẩn hóa mọi xe ba bánh địa phương vào `three_wheeler` và tách box `person` độc lập.
- Quy tắc mới trong `guideline_v1.1_calibrated.md`: Bổ sung định nghĩa xe máy điện/xe đạp điện, chuẩn hóa cụm xe máy nhỏ bằng `ignore_region`, quy tắc hàng chở nhô ra ngoài xe và nhận diện người trong xe mui kín.
- Nguyên nhân gốc: Phần lớn do **lỗi mô hình AI (YOLO COCO)** chưa được học lớp địa phương, kết hợp với một phần **guideline v1.0 để mở** chưa quy định cụ thể về kích thước tối thiểu của cụm xe.

## 7. QC và quyết định (Part 6)

- Mẫu kiểm tra: **5 ảnh ngẫu nhiên** (108 đối tượng, seed 42) gồm `img_003.jpg`, `img_006.jpg`, `img_014.jpg`, `img_031.jpg`, `img_035.jpg`. Người kiểm: Bùi Văn Thịnh.
- Lỗi theo **đối tượng**: **0/108 = 0.0%**; theo **ảnh**: **0/5 = 0.0%** (ngưỡng quy định Release $\le 2\%$).
- **Giới hạn mẫu nhỏ:** Với 0 ảnh lỗi trong mẫu 5 ảnh, cận trên khoảng tin cậy 95% Wilson của tỉ lệ lỗi thật lên tới **43%**. Cần mở rộng mẫu kiểm tra lên $\ge 20$ ảnh nếu muốn khẳng định độ tin cậy thống kê cao hơn.
- Script tự động kiểm tra được tính toàn vẹn file, định dạng COCO, số lượng box; nhưng **không thể kiểm tra** được box có thực sự ôm khít pixel đối tượng hay không nếu không có mắt người đối chứng.

**Quyết định: REWORK**, ngày 2026-11-17.
- **Lý do (theo cây quyết định Hold $\rightarrow$ Rework $\rightarrow$ Release):** Mặc dù mẫu kiểm tra ngẫu nhiên 5 ảnh đạt 0% lỗi, nhưng chỉ số bắt bẫy tổng thể chỉ đạt **58.3% (7/12 bẫy)**, chưa đạt ngưỡng yêu cầu chất lượng của taxonomy (`trap_catch_min = 0.8` tức $\ge 80\%$). Điều này cảnh báo còn tồn tại rủi ro thiên kiến tự động hóa lọt lỗi ở các ảnh phức tạp khác trong lô.
- **Kế hoạch Rework:** Rà soát lại 5 ảnh đã lọt bẫy (`img_018`, `img_025`, `img_026`, `img_028`, `img_029`), co lại box lỏng và bổ sung các đối tượng bị che khuất để đưa vào bản cập nhật tiếp theo.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** Dùng Antigravity hỗ trợ điều phối các script Python, tự động đo kiểm và phân tích bảng số liệu. Mọi nhãn manual và duyệt review trên CVAT đều do chính học viên thực hiện bằng mắt trần.
- Gold có bị nhiễm không: **Hoàn toàn không**. 4 ảnh gold được gán tay độc lập từ đầu trên CVAT trước khi chạy bất kỳ mô hình pre-label nào.
- Môi trường thực nghiệm: Windows 10/11, Python 3.12, CPU, Docker CVAT v2.81.0 chạy local.
- Dùng cache: **Không dùng cache có sẵn**, toàn bộ các lệnh `prelabel.py` YOLO26, YOLOE và SAM đều được tự chạy trực tiếp trên máy.
- Một điều làm khác lần sau: Sẽ siết chặt kỷ luật "bước 0" ẩn nhãn máy ngay từ ảnh đầu tiên để không bị lọt các bẫy box lỏng (`loose`).
