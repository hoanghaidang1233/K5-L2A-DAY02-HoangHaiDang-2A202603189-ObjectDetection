# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Hoàng Hải Đăng
- MSSV: 2A202603189
- Email:
- Lớp: K5-L2A
- Link repo bài làm: https://github.com/hoanghaidang1233/K5-L2A-DAY02-HoangHaiDang-2A202603189-ObjectDetection
- Link Colab đã chạy: _(TODO: dán link sau khi chạy notebook)_

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
| | | | |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md): phần nào đã có kết quả, phần nào còn thiếu hoặc chưa kiểm được. Không tự nhận hoàn thành bước chưa chạy.

## 8. Công cụ AI và tài liệu tham khảo

- Claude Code (Anthropic): dùng ở bước gold QC để phân tích file `annotation_evaluation.json`, phát hiện lỗi lệch ID lớp, viết script đổi ID và chạy lại checker/evaluator. Bước gán nhãn và vẽ box là tự làm.
- _(TODO: ghi phần bạn đã tự kiểm tra, ví dụ xác nhận lại thứ tự lớp trong công cụ gán nhãn.)_
