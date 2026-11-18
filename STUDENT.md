# Thông tin học viên

- Họ và tên: Hoàng Quốc Chung
- MSHV: 2A202604039
- Lớp (L2 / L3 / khác): L23
- Tài khoản GitHub: qchung2124
- Repo bài nộp (URL): [https://github.com/qchung2124/K5-L23-Day08-2A202604039-HoangQuocChung-AI-Assisted-Labeling](https://github.com/qchung2124/K5-L23-Day08-2A202604039-HoangQuocChung-AI-Assisted-Labeling)
- Commit chốt bài (hash): cập nhật sau khi commit cuối (xem git log -1)
- Máy: Windows 11 (WSL2 Ubuntu), không có GPU (CPU)
- CVAT (Docker): v2.81

## Kết quả tóm tắt (điền cuối bài; số lấy từ submission/qc/*, kèm mẫu số)

- Máy: accepted / edited / deleted: 504/579 = 87.0% / 13/579 = 2.2% / 59/579 = 10.2% (mẫu số: 579 box máy); added: 42/559 = 7.5% (mẫu số: 559 đối tượng sau duyệt)
- So với gold: precision / recall của **máy**: 55/76 = 72.4% / 55/75 = 73.3% (mẫu số: 75 đối tượng gold trên 4 ảnh)
- Bắt bẫy: 10/12 = 83.3%
- Thời gian: gán tay 20 phút / 4 ảnh = 5.00 phút/ảnh; duyệt có pre-label 70 phút / 24 ảnh = 2.92 phút/ảnh
- Lỗi QC: 1/6 = 16.7% theo ảnh (6 ảnh mẫu), 1/123 = 0.8% theo đối tượng (123 đối tượng mẫu)
- **Quyết định:** **REWORK** — lý do một dòng: Tỷ lệ ảnh có lỗi trong mẫu QC đạt 1/6 = 16.7% > 2% (cận trên Wilson 95%: 56%) do sót 1 box person trên xe máy ở img_014.jpg, cần rà soát bổ sung theo guideline_v1.1_calibrated.md trước khi Release.
