# Báo cáo Day 08 — 2A202603054 - Trần Quang Vinh

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Cách làm và công cụ đã dùng: xem mục 8.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26m | 0.35 | Recall trên gold đứng ở 17/27 từ 0.2 đến 0.35, precision tăng 0.61 → 0.71; lên 0.4 mất thêm 1 người (`threshold_sweep.csv`). r01 → r02: lên 0.5 thì person trên pilot giảm 131 → 87, mất nhiều người ở xa |
| bicycle | YOLO26m | 0.2 | Recall 4/5 ở 0.2, tụt còn 3/5 ở 0.25. Chỉ 5 vật nên là ước lượng thô |
| motorbike | YOLO26m | 0.25 | Recall 10/13, precision 0.91; mặc định 0.3 mất thêm 1 xe (9/13). r02: lên 0.5 thì motorbike trên pilot giảm 55 → 32 |
| car | YOLO26m | 0.5 | Recall 7/8 không đổi từ 0.1 đến 0.85; precision 0.54 → 0.64. Script giới hạn trần 0.5 |
| bus, truck | YOLO26m | 0.3 (mặc định) | Gold có 1 bus, 0 truck (< 5 vật): không đủ để chọn ngưỡng riêng |
| three_wheeler | YOLOE-26s, prompt từ `taxonomy.json` cho `three_wheeler` | 0.15 | Ngưỡng do `threshold_sweep` đề xuất: recall 7/7, precision 7/9. Lên 0.25 thì recall còn 5/7. r03: YOLOE tìm được tuk-tuk img_002 (0.93) mà YOLO26 (r01) không có box nào |

- Lệnh `merge_prelabels.py` đã dùng:
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.35,bicycle=0.2,motorbike=0.25,car=0.5,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
  (YOLOE đặt trước nên box `three_wheeler` thắng box `car`/`truck` của YOLO26 khi IoU ≥ 0.6.)
- **Phiên bản SAM:** SAM 2.1 base (`sam2.1_b.pt`), chạy trên Colab T4 vì máy Mac Intel 8 GB không có GPU và không cài được torch bản mới. Có so thêm SAM 1 (`mobile_sam.pt`) ở Part 5.
- **Lỗi không sửa được bằng đổi ngưỡng:** YOLO26 gọi tuk-tuk là `car` (img_004 conf 0.37 và 0.34, img_013 0.42) hoặc bỏ sót hẳn (img_002), vì COCO không có lớp xe ba bánh. Đổi ngưỡng chỉ thêm/bớt box chứ không đổi lớp, nên phải dùng model open-vocabulary (YOLOE) và merge ưu tiên YOLOE. Tương tự, person trên gold tối đa chỉ 18/27 = 67% ở mọi ngưỡng: máy sót người bị che / ngồi trong xe, hạ ngưỡng không cứu được.
- **Box điểm cao mà sai / điểm thấp mà đúng:** img_013 YOLOE gọi xe buýt đỏ phía sau là `three_wheeler`; img_009 YOLOE gọi mái che sạp hàng là `three_wheeler` (0.27); img_002 xe bán tải bị YOLO26 gọi `truck` 0.73. Ngược lại, xích lô thứ 3 ở img_032 đúng nhưng chỉ 0.27.
- **Giới hạn:** ngưỡng được chọn trên chính 4 ảnh gold (61 vật), rồi máy lại được chấm trên gold, nên precision/recall của máy ở mục 2 là lạc quan. Lớp bicycle (5 vật), bus (1), truck (0) có quá ít mẫu.

## 2. AI error profile (từ `qc/review_stats.md`)

Đã loại 23 đối tượng quanh vùng bẫy khỏi bảng (lỗi nhân tạo không phản ánh máy).

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 433 | 86.3% | 502 box máy |
| edited | 16 | 3.2% | 502 box máy |
| deleted | 51 | 10.2% | 502 box máy |
| added | 35 | 7.2% | 485 đối tượng sau duyệt |

- **Lớp máy tệ nhất: `three_wheeler`.** Trong 41 box máy của lớp này chỉ 7 được giữ nguyên, 11 phải sửa, 23 bị xoá (23/41 = 56%). YOLOE gọi nhầm mái che sạp (img_009), hai đầu gánh hàng (img_014), tủ điện (img_005), kệ menu (img_018) là xe ba bánh; box thật thì hay bị cắt đôi (tuk-tuk img_013, xích lô img_010 tách mui và thân thành 2 box). Lớp `car` có added cao nhất (11): máy sót xe ở xa và bị che (img_024 hàng xe phía trên trái).
- 8 trường hợp deleted + added là **đổi lớp** (box đúng chỗ, sai lớp), nên số vật máy **sót thật** ≈ 35 − 8 = 27 (27/485 = 5.6%).
- **So với gold:** precision của máy 46/57 = 80.7% (KTC 95%: 69–89%), recall 46/61 = 75.4% (KTC 95%: 63–84%); mẫu số recall là 61 đối tượng gold. Lạc quan vì lý do ở mục 1.
- `added` = 7.2% **không** có nghĩa máy chỉ sót 7%: nó chỉ đếm vật người duyệt nhìn thấy. Recall trên gold (75%) cho thấy máy sót khoảng 1/4 số vật, và QC (mục 7) còn tìm thêm vật cả máy lẫn người duyệt cùng sót.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- **Bắt bẫy (vòng 1, số đo chính thức): 8/12 = 66.7%** (KTC 95%: 39–86%), dưới ngưỡng 80%. Theo loại: extra 3/3, missing 2/3, wrong_class 2/3, loose 1/3.
- **Bẫy lọt và nguyên nhân:**

| Ảnh | Loại | Vì sao lọt | Bước checklist bị bỏ |
|---|---|---|---|
| img_027 | missing | Xe máy chở 2 người ở xa, cao ~36 px. Khi xem vùng xa chỉ để xoá box < 20 px, không quét lại để tìm vật thiếu | Bước 0–1: quét ảnh trước, tìm bỏ sót |
| img_025 | loose | Xe máy đỗ ở mép phải, box lệch ~23 px trái và ~31 px xuống; trông "gần đúng" nên được chấp nhận | Bước 4: so 4 cạnh box |
| img_022 | loose | Ảnh đêm, người cúi sau thùng carton, box lệch ~10 px khó thấy | Bước 4 |
| img_013 | wrong_class | Đã đổi lớp nhưng thành `bus` (đúng theo ảnh: xe buýt thành phố màu đỏ), còn bẫy chấm theo lớp gốc của máy (`three_wheeler`) | Không bỏ bước; ghi vào calibration CAL-06 |

- **Accepted cao (86.3%) mà bắt bẫy thấp (66.7%)** cho thấy đúng thiên kiến tự động hoá: box máy trông hợp lý thì được giữ mà không đối chiếu kỹ. Các bẫy `extra` (box ma) bắt được hết 3/3 vì dễ thấy, còn lỗi tinh (lệch vài chục px, vật nhỏ ở xa) thì lọt. Ba trong bốn bẫy lọt nằm ở mép ảnh, vùng xa hoặc ảnh tối.
- **Vòng 2 (Rework, sau khi mở `traps_key.json`):** sửa img_022, img_025, img_027 trong CVAT, đồng thời phát hiện thêm 2 lỗi thật không phải bẫy (img_013: hai xe máy sau tuk-tuk bị gộp 1 box; img_027: hai xe đỗ bị gộp). Kết quả 11/12 trong `qc/review_stats_round2.md` **không** dùng làm tỉ lệ bắt bẫy vì đã biết đáp án; nó chỉ cho thấy nhãn cuối đã sạch các lỗi đã biết.

## 4. Máy có đáng dùng không?

- **Thời gian** (`time_log.csv`): gán gold 25 phút / 4 ảnh = 6.25 phút/ảnh; duyệt có pre-label 30 phút / 24 ảnh = 1.25 phút/ảnh. Thiên lệch: gold gán trước nên đã quen cảnh; ảnh gold và batch có độ dày khác nhau.
- **Kết luận:** pre-label **đáng dùng cho person, motorbike, car** (phần lớn box được giữ nguyên, sửa nhanh hơn vẽ mới), **chưa đáng tin cho `three_wheeler`** (23/41 box bị xoá, sửa mất công hơn vẽ). Nhưng thời gian duyệt nhanh đi kèm 4/12 bẫy lọt: tiết kiệm thời gian chỉ có giá trị nếu giữ được chất lượng, nên lô này cần làm lại (mục 7).
- **Ba quy tắc giảm thiên kiến** (kiểm tra được):
  1. Ẩn toàn bộ nhãn máy và đếm vật theo 4 góc phần tư ở 24/24 ảnh trước khi xem nhãn; ghi số đếm mỗi ảnh vào log rồi so với số box sau duyệt.
  2. Phóng to và so đủ 4 cạnh của **mọi** box chạm mép ảnh hoặc nằm trong ảnh tối (kiểm bằng danh sách box mép/đêm đã duyệt; mục tiêu bẫy `loose` 3/3).
  3. Trước khi xoá box nhỏ (< 20 px), quét lại cả dải vật cao 20–40 px ở vùng xa để tìm vật thiếu; ghi số vật thêm được ở bước này.

## 5. Mask SAM cho 3 vật (Part 5)

Làm 1 vật khó (tuk-tuk có người lái, img_013) với 3 biến thể để so số click và phiên bản SAM; chưa làm đủ 3 vật khác nhau (hạn chế).

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 | SAM 2.1 base | 3 (2 dương: thân xe, mui; 1 âm: áo người lái) | **Có**: người lái vẫn bị tô gần hết, click âm chỉ khoét vài lỗ nhỏ | Ôm trọn mui và bánh; tràn qua khung hở sang người ngồi phía sau | `tuktuk_img013.jpg` |
| img_013 | SAM 2.1 base | 1 (dương ở thân xe) | Không | Mask chỉ là một mảnh nhỏ quanh điểm click, không ra cả xe | `tuktuk_img013_1click.jpg` |
| img_013 | SAM 1 (`mobile_sam`) | 3 (như dòng 1) | **Gần như không**: áo người lái không bị tô, chỉ dính mặt và bàn chân | Vẫn tô vùng ghế và nền nhìn xuyên qua khung hở | `tuktuk_img013_sam1.jpg` |

- **Đối chiếu guideline 4.2:** người là vật riêng nên mask xe không được gồm người lái; khoảng hở lớn trong khung xe phải bỏ ra (4.2.2), nhưng cả hai SAM đều tô vào. Guideline thiếu quy tắc cho kính chắn gió và khung hở của tuk-tuk (câu hỏi mở 5, chưa chốt).
- **Rút ra:** một click không đủ để SAM hiểu "cả chiếc xe"; muốn loại người lái cần nhiều click âm (thân và chân) hoặc thêm `--box`. Trên ảnh này SAM 1 tách người tốt hơn SAM 2.1 với cùng click, nên số click/phiên bản phải chọn theo vật, không cố định.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: không thử, vì CVAT Docker trên máy chưa cài serverless (Nuclio) và Mac Intel 8 GB không đủ RAM.

## 6. Calibration (Part 6)

- **Số quyết định:** 8 (`qc/calibration_decisions.csv`): 2 `loi_may`, 4 `cau_hoi_mo`, 1 `can_review`, 1 `bay_lot`.
- **Quan trọng nhất:** CAL-02 (YOLOE gọi vật không phải xe là `three_wheeler`) → thêm điều kiện tối thiểu: chỉ gán khi thấy thân xe và ≥ 2 bánh hoặc mui gắn trên khung. CAL-05 (lỗi QC img_024) → `ignore_region` không được chồng lên vật đã có box riêng.
- **Quy tắc mới trong `guideline_v1.1_calibrated.md`:** chốt câu hỏi mở 1 (xe đạp điện), 2 (khối xe dày), 3 (hàng nhô ra), 4 (người bị che: thay "≥ 20%" bằng "thấy đầu và một vai/thân"), 7 (không đặt `conf` tối thiểu); thêm ví dụ minibus cho `bus`. Câu 5, 6 ghi rõ chưa chốt.
- **Nguyên nhân gốc:** phân biệt rõ hai loại. CAL-01 và CAL-06 là **máy sai**, guideline v1.0 đã đúng nên không đổi. CAL-03, 04, 05, 07, 08 là **guideline mơ hồ** (ví dụ "20%" không ước lượng được bằng mắt), nên mới vào v1.1.

## 7. QC và quyết định (Part 6)

- **Mẫu:** 8 ảnh (3 ảnh `can_review` nhiều nhất + 5 ngẫu nhiên, seed 42), 189 đối tượng. **Hạn chế:** người kiểm QC cũng là người duyệt lô.
- **Lỗi theo đối tượng:** 4/189 = 2.1% · **theo ảnh:** 3/8 = 37.5% (ngưỡng Release ≤ 2%).
- **Top lỗi:**
  - `bo_sot` 3: img_004 xe hơi trắng bị che sau tuk-tuk xanh; img_018 người áo đỏ đạp xe phía sau hai phụ nữ (thiếu cả `person` và `bicycle`).
  - `sai_vi_tri_kich_thuoc` 1: img_024 `ignore_region` #273 trùm lên các xe đã gán từng chiếc.
  - Bỏ sót chiếm 3/4 lỗi → lỗi hệ thống, không chỉ sửa từng ảnh.
- **Giới hạn mẫu nhỏ:** với 3/8 ảnh lỗi, cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là 69% (`qc_report_auto.md`). Ngay cả khi mẫu 8 ảnh có 0 lỗi, tỉ lệ thật vẫn có thể tới ~3/8 ≈ 37%, nên ngưỡng 2% không thể chứng minh bằng mẫu cỡ này.
- **Script kiểm được / không kiểm được:** script đếm được trạng thái box, bẫy, tỉ lệ lỗi theo phiếu. Script **không** kiểm được box có ôm sát vật thật không, vật mà cả máy lẫn người duyệt cùng bỏ sót (ngoài gold và bẫy), hay người duyệt có thật sự theo checklist không: 3 lỗi bỏ sót ở QC chính là loại này.

**Quyết định: Rework**, ngày 2026-11-18. Lý do (đi theo Hold → Rework → Release): không Hold vì guideline đã được chốt ở v1.1 cho các ca gây lỗi và dữ liệu không hỏng; Rework vì ảnh lỗi 3/8 = 37.5% > 2%, lỗi bỏ sót lặp lại (3/4 lỗi), và bắt bẫy 8/12 = 66.7% < 80% nên lượt duyệt chưa đáng tin.
**Việc cần làm tiếp:** đã làm 1 vòng sửa (`qc_fixed.json`, 4 ảnh img_013, img_022, img_025, img_027; `release/` đã build lại). Còn phải: sửa 3 lỗi QC (img_004, img_018, img_024), duyệt lại cả 24 ảnh theo 3 quy tắc ở mục 4 và guideline v1.1, chạy lại `build_release.py` và `qc_sample.py --force` với seed khác, ghi kết quả vòng sau.

## 8. Ghi chú cá nhân
- **Máy / GPU / CVAT:** macOS 14.5, Intel Core i5-8259U, 8 GB RAM, không GPU; model chạy trên Google Colab T4 (cuda); CVAT v2.82.0 chạy Docker trên máy (bản lab dùng 2.81, không thấy khác ở các bước đã dùng). Trên máy chỉ cài `requirements-local.txt` (PyTorch không còn bản mới cho Mac Intel), nên `check_env.py` báo FAIL ở torch/ultralytics là đúng dự kiến.
- **Dùng cache `data/cache/`:** không, tự chạy model trên Colab.
- **Lỗi / bug script:** không sửa script nào. Ghi chú: notebook Colab ô 5 mặc định không gói các thư mục chạy thử `t_*`, phải tự zip thêm để có ảnh viz; `qc_fixed.
- **Ca không chắc quy tắc khi gán gold:** img_001 xe đỏ dựng trước cửa hàng (bicycle hay motorbike → CAL-08); img_020 xe tối màu phía sau không đếm được bánh, và hành khách tuk-tuk chỉ thấy thấp thoáng (→ CAL-07); img_036 người ngồi trên xe máy đỗ ở mép phải; img_036 cụm xe máy ở xa cao ~30 px gán `ignore_region` (→ CAL-05).
