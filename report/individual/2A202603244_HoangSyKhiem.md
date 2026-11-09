# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Hoàng Sỹ Khiêm
- MSSV: 2A202603244
- Email: hoangsikhiem3523@gmail.com
- Lớp: K5 Level 2
- Link repo bài làm: https://github.com/hoangkhiem3523/K5-L2-DAY02-HoangSyKhiem-2A202603244-ObjectDetection
- Link Colab đã chạy: (Cập nhật sau khi chạy Colab)

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Tóm tắt ba ca mơ hồ đã quyết định:
1. `drive_001.jpg`: Hai xe đi nối đuôi sát nhau (Box 13 & 14) -> Quyết định vẽ hai bounding box riêng ôm phần nhìn thấy của từng xe, không vẽ gộp.
2. `drive_001.jpg`: Phương tiện ở rất xa cuối làn đường (Box 7) kích thước 13x14px -> Quyết định vẫn gán nhãn `car` vì nhìn rõ hình dạng và đèn hậu.
3. `drive_005.jpg`: Xe bán tải (pickup truck) có thùng hở -> Quyết định gán `car` (Class 0) theo đúng quy định schema.

## 3. Annotation và self-QC

- Số file nhãn train: 50/50 file (`drive_001.txt` đến `drive_050.txt`).
- Kết quả checker và lỗi đã xử lý: `check_yolo_labels.py` ĐẠT cấu trúc 50/50 file nhãn hợp lệ, không có lỗi tọa độ hay vượt biên.
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_001`, `drive_002`, `drive_003`, `drive_005`, `drive_010`, `drive_015`, `drive_020`, `drive_025`, `drive_030`, `drive_050`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa: Phát hiện trường hợp hai xe sát nhau ở làn giữa dễ bị chồng lấn box quá sâu; đã tinh chỉnh lại ranh giới tiếp xúc giữa hai box để đảm bảo IoU chuẩn.
- Commit khóa nhãn ban đầu: `1a13325` (https://github.com/hoangkhiem3523/K5-L2-DAY02-HoangSyKhiem-2A202603244-ObjectDetection/commit/1a13325)

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | | |
| FP | | |
| FN | | |
| Precision | | |
| Recall | | |
| F1 | | |
| mean_iou_of_matches | | |
| passed | | |

Dẫn tới `outputs/annotation_evaluation.json` trong repo. Nếu rework, ghi ảnh, lỗi đã kiểm, quyết định sửa/giữ và lý do. Không suy nguyên nhân chỉ từ FP/FN. Không có rework thì ghi rõ.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch:
- Các tham số thay đổi so với notebook nếu có và lý do:
- Đường dẫn best checkpoint trong phiên chạy:

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | |
| Recall | |
| mAP50 | |
| mAP50–95 | |

Đặt cả 2 folder `assets/eval_predictions/` và `assets/yolo26_runs/` (giải nén từ `day2_colab_outputs.zip`) cạnh báo cáo và dẫn link tương đối. Không chỉ ghi số; chỉ ra ít nhất một ảnh và vật thể cụ thể.

Trả lời bốn câu hỏi cuối notebook:

1. Lớp nào recall thấp nhất? Ảnh nào minh họa?
2. Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?
3. Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?
4. Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Gán nhãn ban đầu | Hai xe đi sát nhau dễ bị vẽ đè lên nhau gây giảm IoU | Zoom to trên CVAT và tách ranh giới rõ ràng | Đảm bảo IoU trên 0.70 |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md): Đã hoàn thành 50 file nhãn YOLO, vượt qua `check_yolo_labels.py`, điền xong `GUIDELINE_MINI.md`. Đang chờ nhận Gold Set từ giảng viên để chấm chất lượng nhãn.

## 8. Công cụ AI và tài liệu tham khảo

Sử dụng mô hình YOLO11n hỗ trợ pre-annotation và kiểm tra đối chiếu; sử dụng CVAT để gán nhãn và kiểm tra trực quan.
