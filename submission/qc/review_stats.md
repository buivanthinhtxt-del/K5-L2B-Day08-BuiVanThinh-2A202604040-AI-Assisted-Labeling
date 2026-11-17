# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 553; đối tượng sau duyệt: 536; ngưỡng accepted: IoU >= 0.9
- Đã **loại 26 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 489 | 489/553 = 88.4% | 553 box máy |
| edited (sửa) | 8 | 8/553 = 1.4% | 553 box máy |
| deleted (xoá) | 56 | 56/553 = 10.1% | 553 box máy |
| added (vẽ thêm) | 39 | 39/536 = 7.3% | 536 đối tượng sau duyệt |

Trong số deleted + added, **11 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 28.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 276 | 0 | 6 | 15 |
| bicycle | 6 | 0 | 2 | 2 |
| motorbike | 142 | 2 | 3 | 9 |
| car | 49 | 0 | 4 | 3 |
| bus | 2 | 0 | 1 | 2 |
| truck | 3 | 0 | 2 | 3 |
| three_wheeler | 11 | 6 | 38 | 5 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 63 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 47 | 14 | 16 | 77.0% (KTC 95%: 65–86%) | 74.6% (KTC 95%: 63–84%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 7/12 = 58.3% (KTC 95%: 32–81%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 2/3
  - extra: 2/3
  - loose: 1/3
  - missing: 2/3
  - lọt: img_018.jpg (loose), img_026.jpg (missing), img_029.jpg (loose), img_028.jpg (extra), img_025.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
