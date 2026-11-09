# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Hoàng Sỹ Khiêm
- MSSV: 2A202603244
- Email: hoangsikhiem3523@gmail.com
- Lớp: K5 Level 2
- Link repo bài làm: https://github.com/hoangkhiem3523/K5-L2-DAY02-HoangSyKhiem-2A202603244-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/github/hoangkhiem3523/K5-L2-DAY02-HoangSyKhiem-2A202603244-ObjectDetection/blob/main/notebooks/day2_annotation_to_yolo.ipynb#scrollTo=mfP3w8_lGjlg

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Tóm tắt ba ca mơ hồ đã quyết định:
1. `drive_001.jpg`: Hai xe đi nối đuôi sát nhau (Box 13 & 14) -> Quyết định vẽ hai bounding box độc lập ôm phần nhìn thấy của từng xe, không vẽ gộp hai xe thành một box duy nhất.
2. `drive_001.jpg`: Phương tiện ở rất xa cuối làn đường (vùng x~618, y~127, Box 7) kích thước 13x14px -> Quyết định vẫn gán nhãn `car` vì nhìn rõ hình khối thân xe sedan và vệt đèn hậu, tránh lỗi bỏ sót (False Negative).
3. `drive_005.jpg`: Xe bán tải (pickup truck) có thùng hở phía sau gây phân vân giữa `car` và `truck` -> Quyết định gán `car` (Class ID: 0) theo đúng quy định schema: "xe bán tải/cabin nhỏ thuộc car, không gán vào truck".

## 3. Annotation và self-QC

- Số file nhãn train: 50/50 file (`drive_001.txt` đến `drive_050.txt`).
- Kết quả checker và lỗi đã xử lý: Chạy `tools/check_yolo_labels.py` ĐẠT cấu trúc 50/50 file nhãn hợp lệ, tọa độ chuẩn hóa trong khoảng [0, 1], không có file rỗng không hợp lệ.
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_001.jpg`, `drive_002.jpg`, `drive_003.jpg`, `drive_005.jpg`, `drive_008.jpg`, `drive_010.jpg`, `drive_015.jpg`, `drive_018.jpg`, `drive_024.jpg`, `drive_050.jpg`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:
  1. `drive_003.jpg`: Phát hiện xe khách to ở mép phải bị gán trùng 2 box đè lên nhau (`truck` và `bus`); đã xóa box `truck` thừa, giữ lại đúng nhãn `bus`.
  2. `drive_018.jpg`: Phát hiện xe bán tải màu đen đỗ ven đường bị cột điện bê tông che ngang thân khiến mô hình tự động bỏ sót; đã bổ sung 1 bounding box bao trọn thân xe qua cột điện với nhãn `car`.
  3. Quét toàn tập: Loại bỏ toàn bộ 49 cặp trùng lặp đa lớp (IoU > 0.80) bằng cơ chế Agnostic NMS, đưa tập nhãn về 820 box chuẩn ôm sát đáy lốp xe.
- Commit khóa nhãn ban đầu: `1a13325` (https://github.com/hoangkhiem3523/K5-L2-DAY02-HoangSyKhiem-2A202603244-ObjectDetection/commit/1a13325)
- Commit sau khi QC tinh chỉnh toàn diện: `ee0bcbe` (https://github.com/hoangkhiem3523/K5-L2-DAY02-HoangSyKhiem-2A202603244-ObjectDetection/commit/ee0bcbe)

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 440 | 612 |
| FP | 380 | 25 |
| FN | 213 | 41 |
| Precision | 0.5366 | 0.9608 |
| Recall | 0.6738 | 0.9372 |
| F1 | 0.5974 | 0.9488 |
| mean_iou_of_matches | 0.8451 | 0.8572 |
| passed | false | true |

Dẫn tới [outputs/annotation_evaluation.json](../../outputs/annotation_evaluation.json) trong repo.

### Ghi chép Rework cụ thể:
1. **Phân tích lỗi trước khi sửa (F1 = 0.5974, mean_iou = 0.8451):**
   - Độ ôm khít của các box đã ghép đạt rất cao (mean_iou = 0.8451 > 0.70), chứng tỏ kỹ thuật kéo box sát mép lốp xe của học viên rất tốt.
   - Tuy nhiên, số lượng box vẽ thừa (FP = 380) và bỏ sót/sai loại (FN = 213) còn cao do sự chênh lệch về phạm vi gán nhãn so với quy chuẩn của Giảng viên:
     - **Nguyên nhân FP (Vẽ thừa):** Học viên ban đầu gán cả các xe ở rất xa trong bãi đỗ hoặc làn đường phụ đối diện (ví dụ: `drive_001.jpg` học viên vẽ 16 box, `drive_028.jpg` vẽ 12 box), trong khi Gold Set của giảng viên chỉ tập trung vào các xe chính trên làn đường lưu thông (Ego-lane và các làn kế cận có ảnh hưởng trực tiếp đến xe tự hành; `drive_001.jpg` Gold chỉ có 5 xe).
     - **Nguyên nhân FN (Sai lớp/Bỏ sót):** Trước khi nhận Gold, các xe minivan/MPV (như trên `drive_005.jpg`, `drive_024.jpg`, `drive_026.jpg`) bị gom nhãn thành `car`. Trong Gold Set, giảng viên tách riêng 54 xe này thành lớp `van` (Class 3).
2. **Quyết định Rework và hành động khắc phục:**
   - Tiến hành rà soát và điều chỉnh phạm vi gán nhãn: Loại bỏ các box ở quá xa ngoài rìa để tránh tạo nhiễu False Positives.
   - Cập nhật đúng lớp `van` (Class 3) cho các dòng xe minivan/xe van chở khách/chở hàng theo đúng đáp án Gold Set.
3. **Kết quả sau Rework:**
   - Precision: 0.9608, Recall: 0.9372, F1: 0.9488, mean_iou_of_matches: 0.8572.
   - Vượt qua cổng kiểm định: **PASSED (true)**, đủ điều kiện tuyệt đối nộp bài.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab T4 GPU, Python 3.11, Ultralytics 8.4.145, PyTorch 2.6.0+cu124.
- Các tham số thay đổi so với notebook nếu có và lý do: Giữ nguyên mặc định theo notebook (`epochs=50`, `imgsz=640`, `seed=20260909`).
- Đường dẫn best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.5288 (52.88%) |
| Recall | 0.4454 (44.54%) |
| mAP50 | 0.3050 (30.50%) |
| mAP50–95 | 0.2180 (21.80%) |

Tài nguyên trực quan hóa đi kèm:
- [outputs/eval_predictions/visuals/](../../outputs/eval_predictions/visuals/) hoặc [assets/eval_predictions/visuals/](assets/eval_predictions/visuals/)
- [assets/yolo26_runs/vehicle_annotation/](assets/yolo26_runs/vehicle_annotation/)
- [assets/yolo26_runs/vehicle_eval/](assets/yolo26_runs/vehicle_eval/)

### Trả lời bốn câu hỏi cuối notebook:

1. **Lớp nào có recall thấp nhất? Chỉ ra một ảnh eval minh họa:**
   - Lớp có Recall thấp nhất là **`van` (Class 3) với Recall = 0.0000 (0.0%)**, tiếp theo là `truck` (39.0%) và `bus` (50.0%), trong khi `car` đạt Recall rất cao (89.36%).
   - Ảnh minh họa: **`drive_058.jpg`** (trong [assets/eval_predictions/visuals/drive_058.jpg](assets/eval_predictions/visuals/drive_058.jpg)). Trong ảnh này có một dãy xe van/minivan đỗ cạnh nhau (Ground Truth có 6 xe nhãn `van`), nhưng mô hình không phát hiện được hoặc nhận nhầm thành `car` do hiện tượng mất cân bằng dữ liệu (class imbalance) trầm trọng trong tập train chỉ có 50 ảnh.

2. **Nếu mAP50 cao nhưng mAP50-95 thấp, cần kiểm điều gì trên box?**
   - Hiện tượng mAP50 cao nhưng mAP50-95 thấp cho thấy mô hình đã phát hiện **đúng vị trí và phân loại đúng vật thể**, nhưng **độ ôm khít của Bounding Box (Localization Accuracy) chưa chuẩn**.
   - Cần kiểm tra:
     - 4 cạnh của box có bị thừa nền đường hoặc lẹm vào thân xe hay không.
     - Đáy box có bị nuốt vào bóng đổ (shadow) của xe trên mặt đường thay vì dừng ở tiếp điểm lốp xe.
     - Khi ngưỡng IoU tăng từ 0.50 lên 0.75, 0.85, 0.95, box chỉ cần lệch 3–5 pixel là IoU sẽ tụt xuống dưới ngưỡng và bị coi là False, kéo tụt mAP50-95.

3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi nhầm lẫn giữa **`car` vs `van`** và **`car` vs `truck`** (đặc biệt là xe bán tải Pickup Truck và xe Minivan/MPV).
   - Dữ kiện hỗ trợ:
     - Trên tập train, xe bán tải có thùng hở thường bị nhầm sang `truck`, nhưng theo guideline đã quy định rõ phải gán về `car`.
     - Các dòng xe 7–9 chỗ (Innova, Carnival, Starex) có thân hình hộp dài nhưng không đủ lớn như xe khách. Nếu guideline không nêu rõ tiêu chí phân biệt giữa xe cá nhân và xe chở khách/chở hàng, người gán nhãn rất dễ dán lúc là `car`, lúc là `van`, khiến mô hình học tín hiệu mâu thuẫn và Recall của `van` rơi về 0.

4. **Vì sao không dùng 10 ảnh eval để train dù chúng có nhãn? Vai trò validation của chúng trong notebook là gì?**
   - **Vì sao không đưa vào train:** Tránh lỗi rò rỉ dữ liệu (**Data Leakage**). Mục tiêu của mô hình AI trong ADAS là khả năng tổng quát hóa trên dữ liệu chưa từng thấy ngoài đời thực. Nếu đem 10 ảnh eval vào train, mô hình sẽ học vẹt (overfit) và chỉ số đo được sẽ là "điểm ảo", không phản ánh đúng năng lực an toàn khi lái xe.
   - **Vai trò validation trong notebook:** 10 ảnh eval đóng vai trò tập kiểm định (Validation Set) độc lập trong quá trình huấn luyện:
     1. Theo dõi độ mất mát (`val/loss`) sau mỗi epoch để phát hiện sớm overfitting.
     2. Dùng làm tiêu chí chọn trọng số tốt nhất (**`best.pt`**) tại epoch mà mAP trên tập eval đạt đỉnh cao nhất.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Gán nhãn ban đầu | Xe bị che khuất bởi cột điện (`drive_018.jpg`) bị mô hình tự động bỏ sót do đứt gãy đặc trưng | Kiểm tra thủ công (Human-in-the-loop), phát hiện và vẽ 1 box bao trọn cả đầu và đuôi xe qua cột | Box ôm trọn xe, không chia 2 hộp, đúng chuẩn ADAS |
| Xử lý box trùng lặp | 49 vị trí xe to bị dán trùng 2-3 nhãn đè lên nhau (`car` + `truck` + `bus`) | Chạy thuật toán lọc Agnostic NMS khử trùng lặp đa lớp | Khử sạch 100% box trùng lặp (IoU > 0.45 về 0), tránh bị phạt 140+ lỗi FP |
| Đường dẫn dataset trên local/Colab | Ultralytics hiểu nhầm đường dẫn tương đối `path: dataset` | Viết script chuyển `path` thành đường dẫn tuyệt đối trong runtime | Chạy training mượt mà trên cả Colab T4 GPU và local GPU |
| Gold QC & Rework | F1 ban đầu đạt 0.5974 do vẽ thừa xe ở quá xa và chưa tách lớp `van` | Rework loại bỏ xe ngoài rìa và cập nhật đúng lớp `van` theo Gold Set | F1 đạt 0.9488, mean IoU đạt 0.8572, vượt qua cổng kiểm tra tự nhiên |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md):
- [x] Đã fork repo, hoàn thành 50 file nhãn YOLO trong `dataset/labels/train/`.
- [x] Vượt qua kiểm tra cấu trúc `check_yolo_labels.py` (50/50 PASS).
- [x] Hoàn thiện `GUIDELINE_MINI.md` với định nghĩa 4 lớp và 3 ca mơ hồ cụ thể.
- [x] Đã khóa nhãn ban đầu qua các commit và push lên GitHub cá nhân.
- [x] Đã hoàn thành Gold QC và rework, vượt qua cổng kiểm tra với `outputs/annotation_evaluation.json` (F1 = 0.9488 >= 0.80, mean_iou = 0.8572 >= 0.70).
- [x] Huấn luyện thành công model `yolo26n.pt` 50 epochs trên Google Colab GPU và đánh giá trên 10 ảnh eval.
- [x] Đã giải nén toàn bộ assets vào `report/individual/assets/` và trả lời đầy đủ 4 câu hỏi phân tích.

## 8. Công cụ AI và tài liệu tham khảo

- Sử dụng mô hình YOLO11n hỗ trợ pre-annotation ban đầu, sau đó kết hợp AI Agent (Antigravity) và kiểm tra thủ công bằng mắt trên CVAT để thực hiện Self-QC, khử trùng lặp và vẽ bổ sung các ca che khuất.
- Sử dụng Google Colab (T4 GPU) và Ultralytics để huấn luyện `yolo26n.pt`.
- Công cụ đánh giá chất lượng nhãn: `tools/evaluate_annotations.py`.
