# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 611; đối tượng sau duyệt: 583; ngưỡng accepted: IoU >= 0.9
- Đã **loại 23 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 562 | 562/611 = 92.0% | 611 box máy |
| edited (sửa) | 7 | 7/611 = 1.1% | 611 box máy |
| deleted (xoá) | 42 | 42/611 = 6.9% | 611 box máy |
| added (vẽ thêm) | 14 | 14/583 = 2.4% | 583 đối tượng sau duyệt |

Trong số deleted + added, **6 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 8.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 263 | 0 | 5 | 2 |
| bicycle | 7 | 0 | 2 | 1 |
| motorbike | 225 | 2 | 17 | 0 |
| car | 52 | 0 | 3 | 2 |
| bus | 3 | 0 | 0 | 3 |
| truck | 4 | 0 | 3 | 1 |
| three_wheeler | 8 | 5 | 12 | 5 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 47 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 39 | 32 | 8 | 54.9% (KTC 95%: 43–66%) | 83.0% (KTC 95%: 70–91%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 9/12 = 75.0% (KTC 95%: 47–91%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 3/3
  - loose: 1/3
  - extra: 3/3
  - missing: 2/3
  - lọt: img_023.jpg (loose), img_025.jpg (loose), img_031.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
