# Thông tin học viên

- Họ và tên: Trần Quang Vinh
- MSHV: 2A202603054
- Lớp (L2 / L3 / khác): L2A
- Tài khoản GitHub: 
- Repo bài nộp (URL): https://github.com/gloryTran04/K5-L2-Day08-TranQuangVinh-2A202603054-AI-Assisted-Labeling
- Commit chốt bài (hash): (điền sau commit cuối)
- Máy: macOS 14.5, Intel i5-8259U, không GPU
- CVAT (Docker): v2.82.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 86.3% / 3.2% / 10.2% (mẫu số: 502 box máy); added: 7.2% (35/485 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 80.7% (46/57) / 75.4% (46/61), lạc quan vì chọn ngưỡng trên chính gold
- Bắt bẫy: 8/12 = 66.7% (vòng 1, số chính thức; vòng 2 sau khi xem đáp án 11/12, không dùng làm số đo)
- Thời gian: gán tay 6.25 phút/ảnh (25/4); duyệt có pre-label 1.25 phút/ảnh (30/24); cả hai có AI làm nháp
- Lỗi QC: 37.5% theo ảnh (3/8), 2.1% theo đối tượng (4/189)
- **Quyết định:** Rework — lý do một dòng: ảnh lỗi 3/8 > 2%, bỏ sót lặp lại (3/4 lỗi), bắt bẫy 8/12 < 80%
