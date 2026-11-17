# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603054 - Trần Quang Vinh

> Chỉ ghi phần thay đổi so với v1.0. Mỗi quy tắc khớp một dòng trong `submission/qc/calibration_decisions.csv`.
> Bản nháp do Claude (AI) soạn từ lỗi máy, bẫy lọt và QC; sinh viên đã đọc, sửa và chịu trách nhiệm từng quy tắc.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | Có bàn đạp **và** không có biển số → `bicycle`; có biển số **hoặc** không có bàn đạp → `motorbike`. Không thấy rõ → chọn lớp tin nhất + `can_review` | img_001 | CAL-08 |
| 2 | Khối xe máy đỗ dày gần camera: từng xe hay `ignore_region`? | Xe cao ≥ 20 px vẫn gán **từng xe**. `ignore_region` chỉ phủ phần không tách được (cao < 20 px hoặc nhoè) và **không được chồng lên vật đã có box riêng** | img_024, img_026 | CAL-05 |
| 3 | Hàng nhô ra: box xe có gồm hàng không? | Box xe **không** gồm hàng nhô ra khỏi khung xe > 20% chiều dài xe. Giỏ/thùng **gắn cố định** vẫn thuộc xe. Mask áp dụng như box | img_008, img_009, img_031 | CAL-03 |
| 4 | Người ngồi trong xe ba bánh có mui / ngồi sau bị che: ngưỡng "≥ 20%"? | Thay % bằng tiêu chí đếm được: gán `person` nếu thấy **đầu và ít nhất một vai/thân**; chỉ thấy tóc hoặc tay → không gán | img_020, img_021, img_027 | CAL-07 |
| 5 | Kính chắn gió / cửa sổ xe thuộc mask xe? | **Chưa chốt**: lab chỉ làm mask 1 vật (tuk-tuk img_013), chưa đủ ca để quyết | — | — |
| 6 | Box máy chồng nhau nhiều trong ảnh đêm: gộp hay giữ? | **Chưa chốt riêng**: ảnh đêm img_022, img_029 xử lý theo quy tắc chung (mỗi vật một box) | — | — |
| 7 | Mức `conf` tối thiểu để chấp nhận box máy không sửa? | **Không đặt.** `conf` chỉ để xếp thứ tự duyệt; bẫy lọt cho thấy box điểm cao vẫn có thể sai | img_025 | — |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_009, img_014, img_005, img_018 | Mái che sạp, gánh hàng, tủ điện, kệ menu | YOLOE gọi `three_wheeler` | Xoá | **Mục 4.3 (thêm):** chỉ gán `three_wheeler` khi thấy thân xe và ≥ 2 bánh hoặc mui gắn trên khung xe (CAL-02) |
| img_018, img_024 | Xe van / minibus chở khách | YOLO26 gọi `truck` hoặc `three_wheeler` | `bus` | **Mục 2, dòng `bus`:** xe chở khách từ 3 hàng ghế trở lên (≈ 12+ chỗ, cửa trượt hông) → `bus`; van nhỏ / MPV → `car` (CAL-04) |
| img_024 | `ignore_region` trùm lên xe đã gán từng chiếc (phát hiện ở QC) | — (lỗi người duyệt) | Thu nhỏ vùng ignore | **Mục 4.1.6:** `ignore_region` không chồng lên vật có box riêng (CAL-05) |
| img_004, img_013 | Tuk-tuk bị gọi `car` | YOLO26 | `three_wheeler` | Không đổi: v1.0 mục 2 đã đúng, lỗi do máy (CAL-01) |
| img_013 | Xe buýt đỏ: máy gọi `three_wheeler`, bẫy đổi thành `car` | YOLOE | `bus` theo ảnh thật | Không đổi: lỗi máy (CAL-06) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu hỏi mở 1, 2, 3, 4, 7; thêm điều kiện gán `three_wheeler` (mục 4.3); thêm ví dụ minibus cho `bus` (mục 2); `ignore_region` không chồng lên vật đã gán (mục 4.1.6). Câu 5, 6 chưa chốt | CAL-02, CAL-03, CAL-04, CAL-05, CAL-07, CAL-08 |
