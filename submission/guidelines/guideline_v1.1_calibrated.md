# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202604039 - Hoàng Quốc Chung

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 và thêm quy tắc mới rút ra từ lỗi của máy, bẫy lọt và kiểm tra QC. Mỗi ca khó đều có ảnh ví dụ (img_0xx) và khớp một dòng trong qc/calibration_decisions.csv.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | case_id |
|---|---|---|---|---|
| 1 | Xe đạp điện: gán bicycle hay motorbike? | Gán **bicycle** nếu phương tiện có bàn đạp rõ ràng, bánh nhỏ và khung mảnh kiểu xe đạp; gán **motorbike** nếu là xe máy điện thân lớn, sàn để chân kín và không có bàn đạp. | img_025.jpg | CAL-01 |
| 2 | Phương tiện bị cắt ở rìa ảnh hoặc bị che khuất một phần có gán nhãn không? | Có gán nếu nhìn rõ **≥ 30%** cấu trúc nhận diện đặc trưng của phương tiện; chỉ bỏ qua (hoặc dùng ignore_region) khi vật thể quá nhỏ/mờ ở hậu cảnh xa không thể phân biệt lớp. | img_020.jpg | CAL-02 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box can_review và QC)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013.jpg | Xe tuk-tuk có tài xế ngồi bên trong cabin, khung rỗng dễ lẫn với người | YOLO26m gọi nhầm three_wheeler thành truck/car; SAM 2.1 dễ dính mask vào chân tài xế | Dùng YOLOE-26s cho three_wheeler (conf=0.15), tách riêng box xe và đặt điểm âm (--neg) lên người lái khi cắt mask SAM | **Mục 3 & 4.2:** Tách riêng 100% khung box và vùng mask của người lái khỏi phương tiện three_wheeler |
| img_023.jpg, img_029.jpg | Bỏ sót vật thể lớn (missing) hoặc box cắt cụt chân/đầu người (loose) khi duyệt nhanh | Pre-label không sinh box (do bẫy missing) hoặc sinh box hụt biên (loose) | Vẽ bổ sung three_wheeler ở img_023.jpg, kéo sát 4 biên person ở img_029.jpg | **Mục 5 (Checklist duyệt):** Bắt buộc quét mắt toàn cảnh ảnh và kiểm tra đủ 4 cạnh (đầu – chân/bánh xe) trước khi chấp nhận box gợi ý |
| img_014.jpg | Xe máy đang dừng/đỗ có người ngồi trên yên ở góc dưới phải bị sót box người | Máy bắt được motorbike (conf=0.88) nhưng bỏ sót person ngồi trên xe | Gán bổ sung box person cho người ngồi/chống chân trên xe máy | **Mục 3 (Quy tắc cặp đối tượng):** Luôn kiểm tra theo cặp (phương tiện 2 bánh + người điều khiển/ngồi trên xe), không để sót box person khi xe bị cắt góc |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, case_id) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt quy tắc phân loại xe đạp điện vs xe máy điện, ngưỡng nhận diện vật cắt rìa ảnh (≥ 30%), quy tắc tách người lái khỏi three_wheeler trên box & mask SAM, quy tắc kiểm tra cặp xe–người và quét toàn cảnh chống thiên kiến | CAL-01 (img_025.jpg), CAL-02 (img_020.jpg), CAL-03 (img_013.jpg), CAL-04 (img_023.jpg, img_029.jpg), CAL-05 (img_014.jpg) |
