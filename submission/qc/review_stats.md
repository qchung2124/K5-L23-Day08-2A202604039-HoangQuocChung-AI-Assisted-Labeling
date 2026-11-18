# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 579; đối tượng sau duyệt: 559; ngưỡng accepted: IoU >= 0.9
- Đã **loại 27 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 504 | 504/579 = 87.0% | 579 box máy |
| edited (sửa) | 13 | 13/579 = 2.2% | 579 box máy |
| deleted (xoá) | 59 | 59/579 = 10.2% | 579 box máy |
| added (vẽ thêm) | 42 | 42/559 = 7.5% | 559 đối tượng sau duyệt |

Trong số deleted + added, **14 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 28.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 259 | 3 | 7 | 6 |
| bicycle | 6 | 0 | 2 | 4 |
| motorbike | 175 | 4 | 17 | 8 |
| car | 50 | 0 | 3 | 15 |
| bus | 2 | 0 | 1 | 2 |
| truck | 2 | 0 | 4 | 3 |
| three_wheeler | 10 | 6 | 25 | 4 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 75 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 55 | 21 | 20 | 72.4% (KTC 95%: 61–81%) | 73.3% (KTC 95%: 62–82%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 2/3
  - missing: 2/3
  - extra: 3/3
  - wrong_class: 3/3
  - lọt: img_023.jpg (missing), img_029.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
