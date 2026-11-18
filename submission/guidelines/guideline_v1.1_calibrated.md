# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603123 - Nguyễn Hoàn Thiện

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện: bicycle hay motorbike? | motorbike (xe có động cơ, theo định nghĩa v1.0 mục 2) | chưa có ảnh ví dụ | CAL-02 |
| 2 | Cụm xe máy đỗ dày sát camera: từng xe hay ignore_region? | ignore_region nếu cao < 20 px; không vẽ từng xe | img_006 | CAL-03 |
| 4 | Người ngồi trong xe ba bánh có mái che: ngưỡng ≥ 20% có hợp lý? | còn thấy ≥ 20% người thì gán `person` | chưa có ảnh ví dụ | CAL-04 |

Câu 3 (hàng nhô ra), 5 (kính chắn gió), 6 (box chồng ảnh đêm), 7 (conf tối thiểu) **chưa chốt**: trong bài này không có ca nào thuộc các câu đó.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_002, img_011 | Tuk-tuk (CAL-01) | YOLO26 gọi là `car` hoặc `truck` | gán `three_wheeler`, sửa tay | không thêm: đã có ở v1.0 mục 2 (lỗi của máy) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt 3 câu hỏi mở (xe đạp điện, cụm xe máy dày, người trong xe ba bánh); thêm mục 2 ghi lỗi máy tuk-tuk | CAL-02, CAL-03, CAL-04; CAL-01 (không đổi quy tắc) |
