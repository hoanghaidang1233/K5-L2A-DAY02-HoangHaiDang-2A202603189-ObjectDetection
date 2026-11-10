# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Hoàng Hải Đăng
- MSSV: 2A202603189
- Email:
- Lớp: K5-L2A
- Link repo bài làm: https://github.com/hoanghaidang1233/K5-L2A-DAY02-HoangHaiDang-2A202603189-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/190inFZaKsSsY9NeIKyWziIFa1mSIUCXZ

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Tóm tắt ba ca mơ hồ đã quyết định, cách áp dụng schema và phần cần giảng viên làm rõ.

## 3. Annotation và self-QC

- Số file nhãn train: 50/50 (cùng basename với ảnh trong `dataset/images/train/`)
- Kết quả checker và lỗi đã xử lý: `python tools/check_yolo_labels.py` → "ĐẠT cấu trúc: 50/50 file nhãn hợp lệ" (cả trước và sau rework).
- Danh sách ít nhất 10 ảnh đã rà bằng mắt:
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:
- Commit khóa nhãn ban đầu: `b598013`

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 533 | 653 |
| FP | 229 | 109 |
| FN | 120 | 0 |
| Precision | 0.6995 | 0.8570 |
| Recall | 0.8162 | 1.0000 |
| F1 | 0.7534 | 0.9230 |
| mean_iou_of_matches | 0.9527 | 0.9553 |
| passed | false | true |

Kết quả sau rework: [`outputs/annotation_evaluation.json`](../../outputs/annotation_evaluation.json). Bản trước rework nằm ở commit `b598013`.

**Rework 1 — sai thứ tự ID lớp khi xuất nhãn (áp dụng cho cả 50 ảnh).**

- Lỗi đã kiểm: so khớp không phân biệt lớp (IoU ≥ 0.5) cho thấy mọi box `truck/bus/van` đều đúng vị trí nhưng sai ID theo một quy luật cố định: truck bị ghi `3`, bus bị ghi `1`, van bị ghi `2`. Riêng `car` (`0`) đúng. Nguyên nhân: thứ tự lớp khai báo trong công cụ gán nhãn khác với `data.yaml` (`car=0, truck=1, bus=2, van=3`).
- Quyết định sửa: chỉ đổi cột ID (`1→2`, `2→3`, `3→1`) cho 762 box, giữ nguyên tọa độ, không thêm hay xóa box nào.
- Kết quả: FN giảm từ 120 xuống 0, F1 tăng từ 0.75 lên 0.92.

**FP còn lại (109 box, chủ yếu `car`).** Nhiều nhất ở `drive_001`, `drive_028`, `drive_049` (8 box mỗi ảnh), tiếp theo là `drive_002`, `drive_031`, `drive_043`. _(TODO: mở các ảnh này để kiểm tra box thừa thuộc loại nào, như xe quá xa/nhỏ hay bị che, rồi ghi quyết định giữ/xóa và lý do.)_

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab, GPU Tesla T4 (15 GB), Python 3.13.16, Ultralytics 8.4.208, Torch 2.14.1+cu130. Notebook clone repo ở commit `15d1aa0` (nhãn sau rework).
- Các tham số thay đổi so với notebook nếu có và lý do: không thay đổi (yolo26n.pt, epochs=50, imgsz=640, batch=-1, seed=20260909).
- Đường dẫn best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt` (không commit file `.pt`).

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.7197 |
| Recall | 0.6256 |
| mAP50 | 0.6436 |
| mAP50–95 | 0.4406 |

Chi tiết theo lớp (cell đánh giá `best_model.val` trên 10 ảnh eval):

| Lớp | Ảnh | Instances | Precision | Recall | mAP50 | mAP50–95 |
|---|--:|--:|--:|--:|--:|--:|
| car | 10 | 94 | 0.724 | 0.809 | 0.886 | 0.616 |
| truck | 5 | 5 | 0.584 | 0.600 | 0.396 | 0.255 |
| bus | 2 | 2 | 0.790 | 0.500 | 0.524 | 0.419 |
| van | 6 | 12 | 0.780 | 0.593 | 0.768 | 0.472 |

Kết quả Colab (giải nén từ `day2_colab_outputs.zip`, đã bỏ file `.pt`):

- Ảnh dự đoán trên 10 ảnh eval (conf=0.25): [drive_051](assets/eval_predictions/visuals/drive_051.jpg) · [drive_052](assets/eval_predictions/visuals/drive_052.jpg) · [drive_053](assets/eval_predictions/visuals/drive_053.jpg) · [drive_054](assets/eval_predictions/visuals/drive_054.jpg) · [drive_055](assets/eval_predictions/visuals/drive_055.jpg) · [drive_056](assets/eval_predictions/visuals/drive_056.jpg) · [drive_057](assets/eval_predictions/visuals/drive_057.jpg) · [drive_058](assets/eval_predictions/visuals/drive_058.jpg) · [drive_059](assets/eval_predictions/visuals/drive_059.jpg) · [drive_060](assets/eval_predictions/visuals/drive_060.jpg)
- Log train theo epoch: [results.csv](assets/yolo26_runs/vehicle_annotation/results.csv), [results.png](assets/yolo26_runs/vehicle_annotation/results.png)
- Đánh giá best checkpoint: [confusion matrix](assets/yolo26_runs/vehicle_eval/confusion_matrix.png), [confusion matrix chuẩn hóa](assets/yolo26_runs/vehicle_eval/confusion_matrix_normalized.png), [PR curve](assets/yolo26_runs/vehicle_eval/BoxPR_curve.png), [F1 curve](assets/yolo26_runs/vehicle_eval/BoxF1_curve.png)
- So sánh nhãn gold và dự đoán trên batch eval: [val_batch0_labels.jpg](assets/yolo26_runs/vehicle_eval/val_batch0_labels.jpg) và [val_batch0_pred.jpg](assets/yolo26_runs/vehicle_eval/val_batch0_pred.jpg)

![Ma trận nhầm lẫn trên eval](assets/yolo26_runs/vehicle_eval/confusion_matrix_normalized.png)

_(TODO: chỉ ra ít nhất một ảnh và vật thể cụ thể khi trả lời các câu dưới.)_

Trả lời bốn câu hỏi cuối notebook:

1. Lớp nào recall thấp nhất? Ảnh nào minh họa?
2. Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?
3. Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?
4. Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| CP6 — lần chạy Colab đầu tiên | Notebook clone commit `b598013` (nhãn trước rework, ID truck/bus/van bị lệch). mAP50 = 0.277; bus mAP50 = 0.016, van mAP50 = 0.075. | Push bản nhãn đã sửa ID (`15d1aa0`), xóa runtime, chuyển sang GPU T4 và chạy lại toàn bộ notebook. | mAP50 tăng lên 0.644. Kết quả ở mục 5 là của lần chạy lại. |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md): phần nào đã có kết quả, phần nào còn thiếu hoặc chưa kiểm được. Không tự nhận hoàn thành bước chưa chạy.

## 8. Công cụ AI và tài liệu tham khảo

- Claude Code (Anthropic): dùng ở bước gold QC để phân tích file `annotation_evaluation.json`, phát hiện lỗi lệch ID lớp, viết script đổi ID và chạy lại checker/evaluator. Bước gán nhãn và vẽ box là tự làm.
- _(TODO: ghi phần bạn đã tự kiểm tra, ví dụ xác nhận lại thứ tự lớp trong công cụ gán nhãn.)_
