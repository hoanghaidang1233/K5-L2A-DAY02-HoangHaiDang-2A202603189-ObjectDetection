# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Hoàng Hải Đăng
- MSSV: 2A202603189
- Email:26ai.danghh2@vinuni.edu.vn
- Lớp: K5-L2A
- Link repo bài làm: https://github.com/hoanghaidang1233/K5-L2A-DAY02-HoangHaiDang-2A202603189-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/190inFZaKsSsY9NeIKyWziIFa1mSIUCXZ#scrollTo=3e0508d8

## 2. Schema và mini guideline

Guideline của nằm ở file [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Gán nhãn theo đúng thứ tự lớp trong `data.yaml`: car = 0, truck = 1, bus = 2, van = 3. Lớp là loại xe chứ không phải trạng thái, nên xe đang đỗ em vẫn gán nhãn.

Ba ca cảm thấy khó và cách xử lý:

| Ảnh | Tình huống | Quyết định |
|---|---|---|
| `drive_015` | Nhiều xe đỗ sát nhau, xe trước che một phần xe sau | Vẽ riêng box cho từng xe, chỉ lấy phần nhìn thấy |
| `drive_027` | Xe ở xa, không rõ là car hay van | Thân xe vuông, nóc cao thì gán van, dáng thấp thuôn thì gán car |
| `drive_042` | Xe bị cột đèn, dải phân cách che ngang | Vẽ box phần thân xe nhìn thấy, bỏ qua xe máy và người đi bộ |

Có 2 chỗ em muốn hỏi lại giảng viên:
- Số ID lớp trong gold khác với `data.yaml` (em giải thích ở mục 4).
- Gold không gán nhãn xe đỗ ven đường, trong khi README nói xe đỗ vẫn là car.

## 3. Annotation và self-QC

- Số file nhãn train: 50/50, tên file trùng với tên ảnh.
- Kết quả checker và lỗi đã xử lý: chạy `python tools/check_yolo_labels.py` ra "ĐẠT cấu trúc: 50/50 file nhãn hợp lệ". Không có lỗi định dạng.
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_001`, `drive_002`, `drive_008`, `drive_009`, `drive_012`, `drive_014`, `drive_021`, `drive_024`, `drive_028`, `drive_037`, `drive_049`. Em vẽ box của mình lên ảnh để xem, sau khi có gold thì vẽ thêm box gold để so.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa: không thấy box nào bị lỏng hay gán sai loại xe. Chỗ khác với gold chủ yếu là số ID lớp và xe đỗ (mục 4).Không sửa nhãn, bản cuối giống bản đã khóa.
- Commit khóa nhãn ban đầu: `b598013`

## 4. Gold QC và rework

| Chỉ số | Trước sửa (`b598013`) | Bản thử đổi ID theo gold (`4ad9ba4`, đã bỏ) |
|---|---:|---:|
| TP | 533 | 653 |
| FP | 229 | 109 |
| FN | 120 | 0 |
| Precision | 0.6995 | 0.8570 |
| Recall | 0.8162 | 1.0000 |
| F1 | 0.7534 | 0.9230 |
| mean_iou_of_matches | 0.9527 | 0.9553 |
| passed | false | true |

File kết quả: [`outputs/annotation_evaluation.json`](../../outputs/annotation_evaluation.json), chấm trên nhãn gốc `b598013`. F1 của em là 0.75.


Cắt ảnh các xe theo từng lớp của gold ra xem thì thấy:

| Gold ghi | Xe thật trong ảnh | ghi | Theo `data.yaml` là |
|---|---|---|---|
| truck (1) | xe van, minibus màu bạc (`drive_002`, `drive_008`, `drive_012`, `drive_021`, `drive_024`) | van (3) | van |
| bus (2) | xe tải ben (`drive_008`), xe tải thùng xanh (`drive_009`), xe cứu hộ (`drive_014`), xe bồn (`drive_037`) | truck (1) | truck |
| van (3) | xe buýt thành phố (`drive_002`), buýt nối toa vàng (`drive_008`), buýt đỏ trắng (`drive_009`) | bus (2) | bus |

Nhãn eval cũng bị lệch giống vậy. Ví dụ xe buýt hai tầng ở giữa ảnh `drive_052` và xe buýt vàng trong `drive_058` đều bị ghi là van.

**109 box FP còn lại** (ở 31 ảnh, gần như toàn car). Nhiều nhất là `drive_001`, `drive_028`, `drive_049` (mỗi ảnh 8 box), rồi tới `drive_002`, `drive_031`, `drive_043` (7 box).

- Chỉ có 1 box đè một phần lên box gold, 108 box còn lại nằm ở chỗ gold không vẽ gì. Vậy không phải do vẽ box lệch hay vẽ trùng.
- Vẽ box của mình và box gold lên `drive_001`, `drive_028`, `drive_049` thì thấy phần lớn là xe đỗ bên lề đường. Gold chỉ gán nhãn xe đang chạy trên đường chính.
- Trong 109 box: 44 box rất nhỏ (dưới 0.2% diện tích ảnh), 48 box nằm ở phía trên ảnh (xe ở xa), 16 box còn lại bình thường.
- Giữ các box này vì README có ghi xe đỗ vẫn là car, và guideline (ca `drive_015`) cũng gán nhãn xe đỗ.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Colab, GPU Tesla T4, Python 3.13.16, Ultralytics 8.4.208, Torch 2.14.1+cu130. Notebook clone repo ở commit `1461672`.
- Các tham số thay đổi so với notebook nếu có và lý do:  (yolo26n.pt, 50 epoch, imgsz 640, batch=-1, seed 20260909).
- Đường dẫn best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt` (không đưa file `.pt` lên repo).

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.3156 |
| Recall | 0.3066 |
| mAP50 | 0.2740 |
| mAP50–95 | 0.2006 |

Kết quả từng lớp. Cột cuối là lần em chạy thử với nhãn đã đổi ID cho giống gold (commit `4ad9ba4`, đã bỏ) để so sánh:

| Lớp | Số xe trong eval | Precision | Recall | mAP50 | mAP50–95 | mAP50 lần chạy thử |
|---|--:|--:|--:|--:|--:|--:|
| car | 94 | 0.629 | 0.777 | 0.738 | 0.521 | 0.886 |
| truck | 5 | 0.391 | 0.200 | 0.235 | 0.203 | 0.396 |
| bus | 2 | 0.000 | 0.000 | 0.008 | 0.003 | 0.524 |
| van | 12 | 0.243 | 0.250 | 0.115 | 0.075 | 0.768 |
| all | 113 | 0.316 | 0.307 | 0.274 | 0.201 | 0.644 |

File kết quả từ Colab (giải nén từ `day2_colab_outputs.zip`, đã bỏ file `.pt`):

- Ảnh dự đoán 10 ảnh eval: [drive_051](assets/eval_predictions/visuals/drive_051.jpg) · [drive_052](assets/eval_predictions/visuals/drive_052.jpg) · [drive_053](assets/eval_predictions/visuals/drive_053.jpg) · [drive_054](assets/eval_predictions/visuals/drive_054.jpg) · [drive_055](assets/eval_predictions/visuals/drive_055.jpg) · [drive_056](assets/eval_predictions/visuals/drive_056.jpg) · [drive_057](assets/eval_predictions/visuals/drive_057.jpg) · [drive_058](assets/eval_predictions/visuals/drive_058.jpg) · [drive_059](assets/eval_predictions/visuals/drive_059.jpg) · [drive_060](assets/eval_predictions/visuals/drive_060.jpg)
- Log train: [results.csv](assets/yolo26_runs/vehicle_annotation/results.csv), [results.png](assets/yolo26_runs/vehicle_annotation/results.png)
- Ảnh đánh giá: [confusion matrix](assets/yolo26_runs/vehicle_eval/confusion_matrix.png), [confusion matrix normalized](assets/yolo26_runs/vehicle_eval/confusion_matrix_normalized.png), [PR curve](assets/yolo26_runs/vehicle_eval/BoxPR_curve.png), [F1 curve](assets/yolo26_runs/vehicle_eval/BoxF1_curve.png)
- Nhãn gold và dự đoán trên eval: [val_batch0_labels.jpg](assets/yolo26_runs/vehicle_eval/val_batch0_labels.jpg), [val_batch0_pred.jpg](assets/yolo26_runs/vehicle_eval/val_batch0_pred.jpg)

Nhận xét:

- Precision 0.32 nghĩa là trong các box model vẽ ra, khoảng 1/3 là đúng so với gold. Recall 0.31 nghĩa là trong 113 xe ở eval, model tìm đúng được khoảng 1/3.
- Lớp car ổn (mAP50 0.738). Car là lớp duy nhất mà nhãn và gold dùng cùng số ID.
- Truck, bus, van gần như bằng 0.Lý do chính không phải model không nhận ra xe, mà là gold eval bị lệch ID như ở mục 4. Ví dụ ảnh `drive_052`: model vẽ đúng xe buýt hai tầng và ghi là bus, nhưng gold ghi là van nên bị tính sai.
- Khi chạy thử với nhãn đã đổi ID cho giống gold, mAP50 tăng từ 0.274 lên 0.644. Ảnh, model và tham số giống hệt, chỉ khác nhãn. Thấy nhãn ảnh hưởng tới kết quả model nhiều hơn .

Trả lời bốn câu hỏi cuối notebook:

1. Lớp nào recall thấp nhất? Ảnh nào minh họa?

   Lớp bus, recall bằng 0. Gold eval chỉ có 2 box bus, ở góc trên bên phải ảnh `drive_051` và mép phải ảnh `drive_058`. Xem ảnh thì cả hai đều là xe tải (xe tải thùng ở `drive_051`, xe cứu hộ ở `drive_058`). Model học theo nhãn của em nên gọi chúng là truck, vì vậy không khớp được với bus của gold. Lớp này chỉ có 2 xe nên chỉ cần sai 1 xe là recall đã giảm 50%.

2. Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?

   Nghĩa là model tìm đúng chỗ có xe nhưng box chưa sát, có thể to hơn, nhỏ hơn hoặc lệch so với xe. Ở bài của em, car có mAP50 là 0.738 nhưng mAP50–95 chỉ 0.521, xem lại box của các xe nhỏ ở xa (ví dụ hàng xe phía trên ảnh `drive_058`) và các xe bị che hoặc bị cắt ở mép ảnh.

3. Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?

   Lỗi lớn nhất là số ID lớp giữa gold và `data.yaml` không giống nhau (gold 1 là van, 2 là truck, 3 là bus). Dữ kiện là mọi box truck/bus/van đều trùng vị trí nhưng lệch ID theo cùng một kiểu, và ảnh cắt ở mục 4 cho thấy đúng loại xe. Ngoài ra car và van cũng dễ nhầm với minivan hoặc SUV, giống ca `drive_027` trong guideline của em. Ví dụ trong `drive_053` có một chiếc minivan màu tối, em thấy rất dễ gán nhầm thành car.

4. Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?

   Nếu train bằng ảnh eval thì model đã thấy trước đáp án, điểm sẽ cao nhưng không cho biết model làm tốt trên ảnh mới hay không. Trong notebook, 10 ảnh eval được dùng sau mỗi epoch để chọn `best.pt`, rồi lại dùng để chấm điểm cuối. Vì vậy điểm này có thể hơi cao hơn thực tế, vì nó không phải tập test riêng.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| CP5 — Gold QC | F1 0.7534, thấp hơn 0.80. Box truck/bus/van lệch ID so với gold. | Lúc đầu nghĩ mình xuất sai thứ tự lớp nên đổi ID cho giống gold (`4ad9ba4`). Sau đó cắt ảnh từng lớp gold ra xem. | Thấy gold mới là bên lệch, em trả lại nhãn gốc (`1461672`). Cần giảng viên xác nhận. |
| CP6 — Colab | Lần đầu chạy bằng CPU nên rất chậm. Notebook đang để chế độ không lưu output. | Đổi sang GPU T4, xóa runtime và chạy lại từ đầu mỗi lần đổi nhãn. | Kết quả ở mục 5 là lần chạy trên `1461672`. |

## 7. Tự đánh giá theo rubric

So với [RUBRIC.md](../../RUBRIC.md):

## 8. Công cụ AI và tài liệu tham khảo


