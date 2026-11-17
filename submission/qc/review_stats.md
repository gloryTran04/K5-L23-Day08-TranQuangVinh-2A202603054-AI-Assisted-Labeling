# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 502; đối tượng sau duyệt: 485; ngưỡng accepted: IoU >= 0.9
- Đã **loại 23 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 433 | 433/502 = 86.3% | 502 box máy |
| edited (sửa) | 16 | 16/502 = 3.2% | 502 box máy |
| deleted (xoá) | 51 | 51/502 = 10.2% | 502 box máy |
| added (vẽ thêm) | 35 | 35/485 = 7.2% | 485 đối tượng sau duyệt |

Trong số deleted + added, **8 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 27.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 239 | 2 | 9 | 10 |
| bicycle | 5 | 1 | 4 | 3 |
| motorbike | 141 | 2 | 9 | 5 |
| car | 36 | 0 | 2 | 11 |
| bus | 3 | 0 | 0 | 2 |
| truck | 2 | 0 | 4 | 1 |
| three_wheeler | 7 | 11 | 23 | 3 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 61 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 46 | 11 | 15 | 80.7% (KTC 95%: 69–89%) | 75.4% (KTC 95%: 63–84%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 8/12 = 66.7% (KTC 95%: 39–86%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 3/3
  - missing: 2/3
  - loose: 1/3
  - wrong_class: 2/3
  - lọt: img_027.jpg (missing), img_025.jpg (loose), img_013.jpg (wrong_class), img_022.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
