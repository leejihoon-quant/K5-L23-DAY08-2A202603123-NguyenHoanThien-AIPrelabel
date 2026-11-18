# Báo cáo Day 08 — 2A202603123 - Nguyễn Hoàn Thiện

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person, bicycle, car, bus, truck | YOLO26m | 0.3 (mặc định) | r01, r02: YOLO26 bắt tốt người và xe; ngưỡng 0.5 giảm nhiễu (139 box so với 215) nhưng không sửa được tuk-tuk |
| motorbike | YOLO26m | 0.1 (theo threshold_sweep) | r01: xe máy dày, box đúng nhiều; trên gold recall 100%, precision 39% (7 vật gold). Lý do chấp nhận: muốn không bỏ sót xe máy; khi duyệt sẽ xóa các box thừa. |
| three_wheeler | YOLOE-26s, prompt: `three_wheeler` | 0.3 (mặc định) | r03: là model duy nhất gọi đúng tên tuk-tuk (0.93 trên img_002), nhưng điểm thấp và có box nhầm |

- Lớp không đạt recall 90% (person, car, three_wheeler): không ngưỡng nào ≤ 0.5 đạt 90% trên gold. Đây là giới hạn của model với lớp đó, không ép chọn ngưỡng. Dùng mặc định 0.3 để thấy đủ box và tự chỉnh; các lớp có ít box thì conf không ảnh hưởng nhiều.
- Lớp quá ít vật (bicycle 4 vật gold, bus 1, truck 1): không chọn ngưỡng riêng, vì mẫu quá nhỏ để kết luận.
- Số đo máy so với gold (4 ảnh, 47 đối tượng; xem mục 2): precision 54.9%, recall 83.0%.

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên, ngưỡng theo mục 1 ở trên):
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.1" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1 (`sam2.1_b.pt`). Chạy CPU mất 12.4 giây/ảnh (r04): với 2–3 ảnh của Part 5 (khoảng 25–40 giây) là chấp nhận được, nhưng không phù hợp để chạy cả lô. Chất lượng mask: xem mục 5. **Chưa so với SAM 1** (chỉ chạy SAM 2.1). SAM 3 không chọn vì cần GPU và quyền truy cập (`docs/MODELS.md`), còn máy chạy CPU (`device` = cpu trong `work/runs.csv`).
- Lỗi nào **không sửa được bằng đổi ngưỡng**: tuk-tuk bị gọi là `car` (img_013, 0.42) hoặc `truck` (img_011, 0.77). Ở ngưỡng 0.5 vẫn không bắt được tuk-tuk xanh-lá giữa img_002. Cách sửa: dùng YOLOE cho `three_wheeler` rồi duyệt tay.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng**: YOLOE đúng tuk-tuk ở img_002 với điểm 0.93, nhưng có box nhầm ở điểm 0.37 trên quầy hoặc xe bán tải bên phải img_002. YOLO26 có box `person` trùng nhau ở đám đông img_002 và img_011.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan. Pilot chỉ 8 ảnh, nên chưa đo được tỉ lệ sót của YOLOE.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 562 | 92.0% | 611 box máy |
| edited | 7 | 1.1% | 611 box máy |
| deleted | 42 | 6.9% | 611 box máy |
| added | 14 | 2.4% | 583 đối tượng sau duyệt |

(Nguồn: `submission/qc/review_stats.md`. Đã loại 23 đối tượng quanh bẫy khỏi bảng. Trong deleted + added có 6 trường hợp đổi lớp; số vật máy sót thật ≈ 8.)

- Lớp nào máy tệ nhất? `three_wheeler`: 12 deleted và 5 edited trên 8 box accepted, nhiều nhầm/lỏng. `motorbike`: 17 deleted trên 225 accepted, chủ yếu box thừa ở đám đông (img_007 và img_011 của pilot là ví dụ về box trùng).
- So với gold (4 ảnh, 47 đối tượng gold): máy có precision 54.9% (TP 39, FP 32; KTC 95%: 43–66%) và recall 83.0% (KTC 95%: 70–91%). Số này lạc quan vì ngưỡng chọn trên chính gold.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 10/12 = 83.3% (KTC 95%: 55–95%), đạt ngưỡng 80% (số của lượt export cuối; xem bảng các lượt ngay dưới). Trượt 2 bẫy `loose` ở img_023 và img_025. Mỗi loại khác đều bắt đủ: wrong_class 3/3, extra 3/3, missing 3/3.

  *(Đoạn bảng sau do trợ lý AI bổ sung từ các file `qc/review_stats_lan1.md`, `_lan2.md`, `_lan3.md` và `qc/review_stats.md`. Bạn kiểm lại và sửa nếu cần.)*

  | Lượt export `batch_reviewed.json` | Bắt bẫy | Bẫy còn lọt (loại) |
  |---|---|---|
  | 1 | 6/12 = 50.0% | img_026, img_005, img_028 (wrong_class); img_023, img_025 (loose); img_031 (missing) |
  | 2 | 9/12 = 75.0% | img_023, img_025 (loose); img_031 (missing) |
  | 3 (hiện hành) | 10/12 = 83.3% | img_023, img_025 (loose) |

  
- Accepted 92% nhưng bắt bẫy 83%: đã bắt được phần lớn lỗi rõ, còn lỗi lệch box (`loose`) thì dễ bỏ qua vì box trông "hợp lý". Chỉ 12 bẫy nên khoảng tin cậy rộng.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay gold 28 phút cho 4 ảnh (trung bình 7 phút/ảnh; từng ảnh 7/8/8/5). Duyệt batch 100 phút cho 24 ảnh (khoảng 4.2 phút/ảnh, đo bằng đồng hồ). So với gán tay gold (7 phút/ảnh), duyệt nhanh hơn khoảng 40%.

| | Gold (gán tay) | Batch (review) |
|---|---|---|
| Số ảnh | 4 | 24 |
| Số đối tượng | 51 | 595 |
| Đối tượng mỗi ảnh | khoảng 12,8 | khoảng 24,8 |
| Tổng thời gian | 28 phút | 100 phút |
| Phút mỗi ảnh | 7,0 | 4,2 |
| Phút mỗi đối tượng | 0,55 | 0,17 |

- Theo ảnh, review nhanh hơn **40%**; theo đối tượng nhanh hơn **69%**. Số này có thể **lạc quan** vì 4 ảnh gold được gán lúc bạn còn đang làm quen cách gán (làm chậm mốc gốc), và khi duyệt batch bạn đã quen cảnh. Số đối tượng cũng không đo chất lượng: duyệt nhanh chưa chắc đúng.

- Kết luận: dùng máy cho lớp lặp lại nhiều (`person`, `car`, `motorbike`), vẽ tay cho lớp ít vật hoặc dễ nhầm (`truck`, `three_wheeler`). Dựa trên `qc/review_stats.md`: person 263 accepted / 5 deleted; car 52 / 3; motorbike 225 / 17; truck 4 accepted nhưng 3 deleted và 1 added; three_wheeler 8 accepted nhưng 12 deleted và 5 edited.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Nghỉ 3–5 phút sau mỗi khoảng 10 ảnh.
  2. Không mở `traps_key.json` trước khi export `batch_reviewed.json`.
  3. Không bỏ qua box điểm cao khi duyệt; điểm không phải xác suất đúng.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_002 (tuk-tuk xanh-lá giữa ảnh) | SAM 2.1 (`sam2.1_b.pt`) | 1 dương + 1 âm (lần đầu 2 dương + 2 âm lỗi "không trả mask", chạy lại bằng ít click hơn) | Mask gộp hành khách ngồi trong cabin và phía sau vào xe (trái với bảng xích lô trong guideline) | Mask khớp thân xe và mái; vùng bánh xe và hành khách bên trong được tô theo | `submission/masks/tuktuk_img002.jpg` |
| img_013 (tuk-tuk đỏ-tím, YOLO26 gọi `car`) | SAM 2.1 (`sam2.1_b.pt`) | 2 dương + 1 âm | Mask có phủ một phần tài xế và hành khách phía sau | Mask khớp thân xe khá sát; diện tích 43.5% ảnh vì xe chiếm phần lớn khung hình; không thấy tràn sang xe máy bên phải | `submission/masks/tuktuk_img013.jpg` |
| img_011 (xe ba bánh mép trái) | SAM 2.1 (`sam2.1_b.pt`) | 2 dương + 1 âm | Xe bị cắt ở mép ảnh; có vùng tô hồng lạ giữa xe trắng và xe ba bánh | Vật bị cắt mép nên mask không đầy đủ | `submission/masks/tuktuk_img011.jpg` |

- Đối chiếu guideline 4.2: điều 1 (lệch biên ≤ ~3 px) và điều 4 (vật chạm nhau không chồng lên nhau) là hai quy tắc áp dụng được. Mắt thường chỉ thấy mask bám khá sát, chưa đo lệch bằng px. Mask img_002 và img_013 gộp hành khách/tài xế vào mask xe, trái với bảng xích lô trong guideline (người là vật riêng, mỗi người một `person`). Guideline chưa nói rõ với tuk-tuk có người ngồi thấp thoáng; đây là ca cho mục 6.

## 6. Calibration (Part 6)

- Số quyết định: 4 ca (CAL-01 đến CAL-04) trong `qc/calibration_decisions.csv`. Quan trọng nhất: **CAL-03** (cụm xe máy đỗ dày sát camera: `ignore_region` nếu cao < 20 px, không vẽ từng xe) và **CAL-02** (xe đạp điện có động cơ là `motorbike`).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: 3 câu hỏi mở của v1.0 đã chốt (xe đạp điện, cụm xe dày, người trong xe ba bánh ≥ 20%). CAL-01 (tuk-tuk bị máy gọi là `car`/`truck`) là lỗi của máy, không thêm quy tắc mới vì v1.0 mục 2 đã nêu.
- Nguyên nhân gốc: guideline mơ hồ (CAL-02, CAL-03, CAL-04); máy sai lớp (CAL-01).

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ca khó + 5 ngẫu nhiên, seed 42): img_003, img_006, img_014, img_031, img_035; 116 đối tượng.
- Lỗi theo **đối tượng**: 0/116 = 0.0% · theo **ảnh**: 0/5 = 0.0% (ngưỡng Release ≤ 2%).
- Top lỗi: không ghi nhận lỗi nào trong mẫu. Hai ảnh cần chú ý khi xem lại: img_006 (cụm xe máy dày, xử lý theo CAL-03) và img_031 (box xe đạp có gom hàng, chưa có quy tắc rõ về hàng nhô ra).
- **Giới hạn mẫu nhỏ**: 0 lỗi trong 5 ảnh, cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là 43% (`qc/qc_report_auto.md`). Mẫu này **không chứng minh** được ngưỡng 2%.
- Script kiểm được: tên file, lớp, box trong ảnh, đủ ảnh, số liệu accepted/edited/deleted/added, bẫy. Không kiểm được: box có ôm sát vật thật không, lỗi chung giữa bạn và máy, việc theo checklist thật. Các việc này đã kiểm bằng mắt trên ảnh.

**Quyết định: Release** (theo cây quyết định: không có lỗi trong mẫu, guideline đã chốt 3 câu hỏi mở), ngày 2026-11-18. Lý do: 0 lỗi trong 116 đối tượng và bẫy đạt 83.3%. Nhưng mẫu chỉ 5 ảnh nên đây là kết luận **yếu**, không phải chứng minh chất lượng ≤ 2%.
Việc cần làm tiếp: (1) bạn xác nhận 3 ảnh kiểm sơ bộ (img_003, img_014, img_035); (2) mở rộng mẫu QC để có kết luận đáng tin.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Claude (Claude Code). Đã dùng để: giải nén repo, cài thư viện (`transformers`, CLIP) và tải model; tạo Project/Task trên CVAT; chạy các script của repo (`prelabel.py`, `threshold_sweep.py`, `merge_prelabels.py`, `inject_traps.py`, `review_stats.py`, `build_release.py`, `qc_sample.py`, `qc_report.py`, `check_submission.py`); hướng dẫn cách đọc số và cách điền báo cáo; kiểm tra sơ bộ 3 ảnh QC (mục 7); đẩy repo lên GitHub. AI không vẽ gold và không review 24 ảnh batch. 
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? (CHƯA ĐIỀN: chỉ bạn biết; ghi thật.)
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: CPU (không GPU), chạy SAM 2.1 bằng `scripts/sam2_points.py`. CVAT v2.74.1 chạy bằng Docker (localhost:8080), Project `Day08-AI-Prelabel`; Task tạo qua REST API vì đường `cvat-core` báo lỗi \"No media data found\".
- Dùng cache `data/cache/` thay vì tự chạy? Không dùng: toàn bộ pre-label do `prelabel.py` chạy trên máy (`work/prelabel/*`, `work/runs.csv`).
- Lỗi / bug script đã gặp (nếu có): lần chạy đầu `sam2_points.py` không trả mask với 2 dương + 2 âm ở img_002; đổi thành 1 dương + 1 âm thì được.
- Một điều bạn làm khác lần sau: Chỉ dùng model vẽ 1 số nhãn nhất định, những nhán ít, sai nhiều thì nên vẽ tay từ đầu
