# Báo cáo Day 08 — 2A202604039 - Hoàng Quốc Chung

> Một file duy nhất cho cả bài (gộp recipe, QC và ghi chú cá nhân). Mọi con số đều kèm **mẫu số** và nguồn dữ liệu.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn run_id trong model_log.csv) |
|---|---|---|---|
| person, bicycle, car, bus, truck | YOLO26m (yolo26 size m) | 0.25 (mặc định tối ưu từ threshold_sweep) | Ở r01 (conf=0.25), YOLO26m bắt tốt các lớp phổ biến với tốc độ nhanh (0.87s/ảnh, 215 box/pilot). Khi tăng lên 0.5 (r02), số box giảm còn 139 nhưng bỏ sót nhiều đối tượng ở xa/bị che khuất. |
| motorbike | YOLO26m (yolo26 size m) | 0.15 | Hạ ngưỡng từ 0.25 xuống 0.15 giúp tăng đáng kể Recall cho xe máy đi san sát hoặc bị che một phần trong phố đông (chấp nhận thêm một ít box nhiễu dễ xóa trên CVAT). |
| three_wheeler | YOLOE-26s (yoloe size s, prompt: three_wheeler) | 0.15 | YOLO26m không có lớp three_wheeler nên gọi nhầm tuk-tuk thành car/truck (r01). YOLOE-26s (r03) nhận diện đúng xe ba bánh (0.36s/ảnh), đặt ngưỡng 0.15 để tránh bỏ sót các xe chở hàng cồng kềnh. |

- **Lệnh merge_prelabels.py đã dùng (dán nguyên):**
  python scripts/merge_prelabels.py --base work/prelabel/y26_val --extra work/prelabel/yoloe_val --thr "motorbike=0.15,three_wheeler=0.15" --out work/prelabel/merged_review
- **Phiên bản SAM đã chọn (sam2.1_b.pt vs mobile_sam.pt) và lý do:** Chọn **sam2.1_b.pt** làm mô hình phân đoạn chính thức. Trên img_013.jpg, sam2.1_b.pt (r05, 4.20s/ảnh) tạo ra 1 polygon liền mạch (317,255 px, 45.4% ảnh) bám sát khung xe tuk-tuk và bánh xe; trong khi mobile_sam.pt (r06, 1.50s/ảnh) tuy nhanh hơn ~2.8 lần nhưng mask bị đứt làm 2 polygon (245,541 px, 35.2% ảnh), mất phần khung ghế giữa xe và lem đỏ lên mặt tài xế.
- **Lỗi nào không sửa được bằng đổi ngưỡng? Sửa thế nào?** Lỗi **sai lớp do lệch taxonomy** (YOLO26m không được huấn luyện lớp three_wheeler nên luôn gán nhầm xe ba bánh thành car hoặc truck dù chỉnh conf=0.25 hay 0.5 ở r01/r02). Cách sửa: dùng mô hình open-vocabulary **YOLOE-26s** chuyên biệt cho three_wheeler, loại bỏ box trùng đè (IoU cao) với car/truck của YOLO26m khi merge và rà soát lại trên CVAT.
- **Box điểm cao mà sai hoặc điểm thấp mà đúng:**
  - *Điểm cao mà sai:* Trên img_005.jpg, bẫy wrong_class gán nhãn bicycle với conf=0.77 cho một chiếc xe máy (motorbike); hoặc trên img_013.jpg, YOLO26m gọi nhầm xe tuk-tuk thành truck với conf=0.47.
  - *Điểm thấp mà đúng:* Trên img_024.jpg và img_029.jpg, nhiều xe máy ở trung cảnh có conf = 0.16 – 0.19 nhưng vẫn đúng vị trí và đúng lớp nhờ hạ ngưỡng motorbike=0.15.
- **Giới hạn của việc chọn ngưỡng trên tập gold:** Ngưỡng motorbike=0.15, three_wheeler=0.15 được tối ưu trực tiếp trên chính 4 ảnh tập gold (75 đối tượng gold, in-sample) nên các chỉ số Precision 72.4% và Recall 73.3% của máy trên gold mang tính chất lạc quan và có khoảng tin cậy 95% rộng (61%–82%).

## 2. AI error profile (từ qc/review_stats.md)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 504 | 87.0% | 579 box máy |
| edited (sửa kích thước/vị trí) | 13 | 2.2% | 579 box máy |
| deleted (xóa box thừa/sai lớp) | 59 | 10.2% | 579 box máy |
| added (vẽ thêm box sót) | 42 | 7.5% | 559 đối tượng sau duyệt |

*(Lưu ý: Trong tổng số deleted + added, có **14 trường hợp là đổi lớp** do máy đặt đúng khung nhưng sai nhãn; số đối tượng máy bỏ sót thực tế là 42 - 14 = 28 đối tượng).*

- **Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)?** Lớp **three_wheeler** có tỷ lệ lỗi cao nhất (25 box bị xóa và 6 box phải chỉnh sửa trên chỉ 10 box được giữ nguyên; tỷ lệ accepted chỉ đạt 10/41 = 24.4%). Nguyên nhân do xe tuk-tuk / xe ba gác chở hàng ở phố châu Á có hình dạng rất đa dạng, khung rỗng và thường đi kèm người lái/hàng hóa cồng kềnh khiến YOLOE-26s dễ bắt nhầm xe máy chở hàng (img_014.jpg, img_023.jpg) hoặc kéo box quá lỏng. Ngoài ra, lớp car cũng bị máy bỏ sót nhiều nhất (15 box phải vẽ thêm) ở các cảnh xe đỗ san sát hậu cảnh (img_024.jpg).
- **So với gold:** Precision của máy đạt **55/76 = 72.4%** (KTC 95%: 61%–81%), Recall của máy đạt **55/75 = 73.3%** (KTC 95%: 62%–82%) trên tổng số **75 đối tượng gold** (4 ảnh).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- **Bắt bẫy:** Đạt **10/12 = 83.3%** (vượt ngưỡng đạt 80%, KTC 95%: 55%–95%). Cụ thể: extra đạt 3/3, wrong_class đạt 3/3, loose đạt 2/3, và missing đạt 2/3.
- **Hai bẫy bị lọt và nguyên nhân:** Lọt img_023.jpg (bẫy missing — bỏ sót 1 xe ba bánh lớn ở trung tâm) và img_029.jpg (bẫy loose — box person bị cắt hụt phần đầu). Nguyên nhân do bị **thiên kiến neo theo khung gợi ý của máy (automation bias)**: khi thấy ảnh nhiều box có sẵn, mắt có xu hướng tập trung kiểm tra các box đang hiện trên màn hình mà bỏ qua bước ẩn toàn bộ nhãn máy để quét tổng thể vùng trống (img_023.jpg) và chưa zoom kỹ từng mép đầu/chân của người đứng gần rìa (img_029.jpg).
- **Tương quan accepted cao (87.0%) và bắt bẫy (83.3%):** Tỷ lệ accepted rất cao giúp tăng tốc độ duyệt nhưng cũng tạo tâm lý chủ quan ("máy làm đúng hầu hết rồi"), khiến người duyệt dễ lướt qua các lỗi tinh vi như box cắt cụt một phần cơ thể hoặc vật thể lớn bị máy bỏ trắng hoàn toàn.

## 4. Máy có đáng dùng không?

- **Thời gian (từ time_log.csv):**
  - Gán tay từ đầu (tập gold): 20 phút / 4 ảnh = **5.00 phút/ảnh** (76 đối tượng ≈ 15.8 giây/đối tượng).
  - Duyệt có pre-label (tập batch): 70 phút / 24 ảnh = **2.92 phút/ảnh** (575 đối tượng ≈ 7.3 giây/đối tượng).
  - **Thiên lệch cần lưu ý:** Tốc độ duyệt nhanh hơn **1.71 lần** (giảm **41.7%** thời gian mỗi ảnh), nhưng con số này có phần thuận lợi vì người duyệt đã gán 4 ảnh gold trước đó nên đã quen thuộc với bộ nhãn và giao diện CVAT.
- **Kết luận:** **Có đáng dùng**, đặc biệt hiệu quả cao cho các lớp phổ biến như person (259 box accepted) và motorbike (175 box accepted) nơi máy đã làm đúng trên 87% khối lượng công việc. Riêng lớp three_wheeler vẫn cần dùng YOLOE-26s để gợi ý vị trí ban đầu nhưng bắt buộc người duyệt kiểm tra 100% thủ công.
- **Ba quy tắc giảm thiên kiến áp dụng trong quy trình:**
  1. **Quy tắc 5 giây quét mù (Blind Scan):** Bấm phím tắt ẩn toàn bộ nhãn máy trong 5 giây đầu khi mở ảnh mới để đếm các vật thể lớn (three_wheeler, car, bus) trước khi bật lại pre-label.
  2. **Kiểm tra 4 điểm neo (4-Corner Check):** Với mỗi box person hoặc motorbike, kiểm tra đủ 4 điểm tiếp xúc (đỉnh đầu – bàn chân/bánh xe – tay lái) để triệt tiêu lỗi loose (IoU < 0.9).
  3. **Rà soát theo cặp xe–người:** Không bao giờ duyệt đơn lẻ box motorbike hoặc bicycle mà không kiểm tra box person đi kèm (và ngược lại).

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hóa? | Lỗi quan sát | File trong masks/ |
|---|---|---|---|---|---|
| img_013.jpg | sam2.1_b.pt | 3 (2 pos, 1 neg) | Không gồm người lái (đã tách phần thân trên tài xế) | Vẫn dính nhẹ một phần ống quần và dép của tài xế ngồi lọt trong cabin; 1 polygon liền mạch (317,255 px) | tuktuk_img013_sam2.jpg |
| img_013.jpg | mobile_sam.pt | 3 (2 pos, 1 neg) | Tách tốt áo/quần tài xế nhưng lem nhẹ lên mặt | Mask bị đứt làm 2 polygon rời rạc (245,541 px), mất khung ghế giữa xe | tuktuk_img013_sam1.jpg |
| img_004.jpg | sam2.1_b.pt | 3 (2 pos, 1 neg) | Không gồm người đứng cạnh xe | Mask bám sát toàn bộ xe tuk-tuk đỏ từ mui xuống bánh xe, tách sạch người áo đen đứng sát bên trái (93,190 px, 1 polygon) | tuktuk_img004_sam2.jpg |
| img_017.jpg | sam2.1_b.pt | 3 (2 pos, 1 neg) | Không gồm xe sau và người đi bộ | Cắt biên chính xác toàn bộ xe tuk-tuk đỏ phía trước, không tràn sang xe tuk-tuk xanh đậu ngay phía sau (84,532 px, 1 polygon) | tuktuk_img017_sam2.jpg |

- **Đối chiếu guideline mục 4.2:** Quy tắc đặt click dương (--pos) lên các mảng màu/cấu trúc chính của xe (mui xe, thân xe) và click âm (--neg) lên người lái giúp tách biệt đối tượng rất tốt trên img_004.jpg và img_017.jpg. Tuy nhiên, guideline mục 4.2 cần bổ sung hướng dẫn đặt **tối thiểu 2 click âm (--neg)** (một ở thân trên, một ở chân/bàn đạp) đối với xe tuk-tuk có khoang lái hở như img_013.jpg để tránh mask tràn xuống chân tài xế.
- **Interactor SAM trong CVAT (tùy chọn):** Không dùng trực tiếp trên CVAT Docker do chạy thuần CPU (không có GPU) nên phản hồi chậm; chạy offline qua script sam2_points.py ổn định hơn, lưu lại được tọa độ click tái lập kết quả và dễ so sánh định lượng giữa các phiên bản mô hình.

## 6. Calibration (Part 6)

- **Số quyết định:** **5 quyết định** (CAL-01 đến CAL-05 trong submission/qc/calibration_decisions.csv).
- **Quyết định quan trọng nhất:** **CAL-01** (chốt tiêu chí phân biệt xe đạp điện có bàn đạp là bicycle vs xe máy điện không bàn đạp là motorbike trên img_025.jpg) và **CAL-03 / CAL-05** (bắt buộc tách riêng và kiểm tra đủ cặp xe + người lái trên cả bounding box lẫn mask SAM tại img_013.jpg, img_014.jpg).
- **Quy tắc mới đã cập nhật vào guideline_v1.1_calibrated.md:**
  1. Tiêu chí phân loại xe đạp điện (bicycle) và xe máy điện (motorbike).
  2. Ngưỡng diện tích/cấu trúc tối thiểu (≥ 30%) để gán nhãn phương tiện bị cắt ở rìa ảnh.
  3. Quy tắc tách 100% người lái khỏi three_wheeler trên box và dùng tối thiểu 2 click âm SAM cho cabin hở.
  4. Quy trình duyệt 5 giây quét mù chống thiên kiến và kiểm tra theo cặp phương tiện – người ngồi trên xe.
- **Nguyên nhân gốc:** Kết hợp giữa **guideline v1.0 còn câu hỏi mở** (xe đạp điện, vật cắt rìa) và **hạn chế của mô hình pre-label** (YOLO26m thiếu lớp three_wheeler, dễ sót người ngồi trên xe ở góc ảnh).

## 7. QC và quyết định (Part 6)

- **Mẫu kiểm tra:** **6 ảnh** (1 ảnh gắn cờ can_review là img_025.jpg + 5 ảnh ngẫu nhiên img_003.jpg, img_006.jpg, img_013.jpg, img_014.jpg, img_033.jpg với seed=42), tổng cộng **123 đối tượng**. *Hạn chế:* Người kiểm tra QC trùng với người duyệt lô batch nên có thể còn điểm mù chủ quan ở các vật thể cực nhỏ ở hậu cảnh xa.
- **Tỷ lệ lỗi QC (từ qc_report_auto.md):**
  - Lỗi theo **đối tượng**: **1/123 = 0.8%**
  - Lỗi theo **ảnh**: **1/6 = 16.7%** (vượt ngưỡng Release ≤ 2% theo ảnh)
- **Top lỗi ghi nhận:**
  1. bo_sot (1 lỗi): Trên img_014.jpg (góc dưới bên phải), khung #136 motorbike đã gán cho chiếc xe máy đỏ nhưng **bỏ sót box person** cho người đang ngồi/chống chân trên yên xe.
- **Giới hạn mẫu nhỏ:** Với 1 ảnh có lỗi trên cỡ mẫu 6 ảnh (16.7%), cận trên khoảng tin cậy 95% (Wilson) của tỷ lệ ảnh lỗi thực tế lên tới **56%**. Do đó, mẫu QC 6 ảnh chưa đủ lực thống kê để khẳng định tỷ lệ lỗi toàn lô dưới 2% mà chỉ dùng để phát hiện lỗi hệ thống còn sót.
- **Script kiểm được / không kiểm được:** Các script tự động (validate_labels.py, qc_report.py) kiểm tra tốt tính hợp lệ cấu trúc COCO, tọa độ vượt biên, nhãn thuộc tính can_review và tính toán khoảng tin cậy Wilson; nhưng không thể tự phát hiện lỗi ngữ nghĩa như box ôm chưa sát viền (loose) hoặc sót người ngồi trên xe ở rìa ảnh (img_014.jpg) — phần này bắt buộc phải có mắt người kiểm tra trên ảnh trực quan work/qc/review/.

**Quyết định: REWORK**, ngày **2026-11-18**.
- **Lý do (theo cây quyết định Hold -> Rework -> Release):** Không **Hold** vì toàn bộ các câu hỏi mở đã được chốt dứt điểm trong guideline_v1.1_calibrated.md (guideline_gap = 0) và dữ liệu ảnh hợp lệ (data_problem = 0). Tuy nhiên chưa thể **Release** ngay vì tỷ lệ ảnh có lỗi trong mẫu QC là **1/6 = 16.7% > 2%** (cận trên Wilson 95% là 56%), đồng thời phát hiện lỗi sót cặp xe – người ở rìa ảnh (img_014.jpg).
- **Việc cần làm tiếp cho vòng Rework:**
  1. Rà soát lại toàn bộ 28 ảnh (4 gold + 24 batch) theo quy tắc cặp xe hai bánh + người ngồi của guideline_v1.1_calibrated.md, bổ sung ngay box person bị thiếu trên img_014.jpg và các ảnh có xe đỗ ở góc/rìa.
  2. Chạy lại build_release.py -> rút mẫu QC mới lớn hơn (--n 10) và kiểm tra lại để đảm bảo tỷ lệ ảnh lỗi ≤ 2% trước khi chuyển sang trạng thái **Release**.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** Sử dụng trợ lý AI để giải thích thông báo lỗi môi trường Docker/WSL2, kiểm tra cú pháp câu lệnh Python/CLI cho các script threshold_sweep.py, merge_prelabels.py, sam2_points.py, qc_report.py và rà soát đối chiếu số liệu từ các file JSON/CSV. Mọi thao tác gán nhãn trên CVAT, chọn tọa độ click SAM và kiểm tra từng ảnh đều do học viên tự thực hiện và kiểm chứng bằng script validate_labels.py / check_submission.py.
- **Gold có bị nhiễm không?** **Không.** Tập 4 ảnh gold (img_010.jpg, img_020.jpg, img_028.jpg, img_035.jpg) được gán tay hoàn toàn từ đầu trên CVAT trước khi chạy bất kỳ mô hình pre-label nào (Part 1) và không dùng pre-label để sửa đè lên gold.
- **Máy / GPU / CVAT dùng; vướng mắc cài đặt:** Máy Windows 11 (chạy môi trường code trên WSL2 Ubuntu .venv), **không có GPU** (chạy suy luận trên CPU); sử dụng **CVAT v2.81** qua Docker Desktop. Vướng mắc gặp phải là thời gian chạy SAM trên CPU khá chậm (4.2s–6.75s/ảnh) và cần ánh xạ đúng định dạng COCO 1.0 khi import/export giữa CVAT và thư mục submission/.
- **Dùng cache data/cache/ thay vì tự chạy?** **Không.** Học viên tự chạy toàn bộ các bước tải model (download_models.py), suy luận YOLO26m, YOLOE-26s, quét ngưỡng, gộp nhãn, tiêm bẫy và sinh mask SAM trên máy cá nhân.
- **Lỗi / bug script đã gặp:** Khi điền phiếu qc_review.csv ban đầu dùng từ khóa tiếng Anh (ok, missing), script qc_report.py báo cảnh báo bỏ qua vì chỉ chấp nhận bộ mã lỗi tiếng Việt không dấu (khong, bo_sot, thua, sai_lop, sai_vi_tri_kich_thuoc, sai_thuoc_tinh, sai_mask); đã sửa lại đúng chuẩn và chạy lại thành công.
- **Một điều sẽ làm khác lần sau:** Sẽ thực hiện nghiêm ngặt bước "5 giây ẩn nhãn máy quét toàn cảnh" trước khi chạm chuột vào các box gợi ý ở Pha 4 để đạt tỷ lệ bắt bẫy **12/12 (100%)** ngay trong lần duyệt đầu tiên thay vì phải quay lại sửa các ca loose/missing.
