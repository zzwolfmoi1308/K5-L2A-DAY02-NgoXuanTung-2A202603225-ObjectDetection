# Báo cáo cá nhân — Ngày 2 Object Detection

Đổi tên file theo `MSSV_HoTen.md`, ví dụ `20241234_NguyenVanAn.md`. Điền kết quả thật, xóa dòng hướng dẫn khi nộp. Xem [quy định nộp](../../SUBMISSION.md).

## 1. Thông tin học viên

- Họ tên: Ngô Xuân Tùng
- MSSV: 2A202603225
- Email: 26ai.tungnx2@vinuni.edu.vn
- Lớp: L2A
- Link repo bài làm: https://github.com/zzwolfmoi1308/K5-L2A-DAY02-NgoXuanTung-2A202603225-ObjectDetection
- Link Colab đã chạy: 

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Tóm tắt ba ca mơ hồ đã quyết định, cách áp dụng schema và phần cần giảng viên làm rõ.

## 3. Annotation và self-QC

- Số file nhãn train: 50
- Kết quả checker và lỗi đã xử lý: ĐẠT cấu trúc 50/50 file nhãn hợp lệ khi chạy `tools/check_yolo_labels.py` (đủ 50 file, đúng 5 cột normalized, không có box vượt biên ảnh).
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_005.jpg`, `drive_007.jpg`, `drive_008.jpg`, `drive_009.jpg`, `drive_015.jpg`, `drive_019.jpg`, `drive_021.jpg`, `drive_024.jpg`, `drive_026.jpg`, `drive_030.jpg`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:
  - Ảnh `drive_008.jpg`: Bỏ sót 1 xe buýt (`bus`), sau khi rà soát đã gán bổ sung bounding box ôm sát phần nhìn thấy của xe.
  - Ảnh `drive_009.jpg`: Gán nhầm class là `van`, đã sửa lại thành `truck` (ID 1) để khớp với mô tả trong `GUIDELINE_MINI.md`.
- Commit khóa nhãn ban đầu: d426db1c6cc3a8621fd67ae58820727ae21ce08c

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 489 | 488 |
| FP | 353 | 175 |
| FN | 164 | 165 |
| Precision | 0.5808 | 0.7360 |
| Recall | 0.7489 | 0.7473 |
| F1 | 0.6542 | 0.7416 |
| mean_iou_of_matches | 0.8305 | 0.8311 |
| passed | false | false |

- File đánh giá: [outputs/annotation_evaluation.json](../../outputs/annotation_evaluation.json)

### Phân tích quá trình rework và giải trình nguyên nhân:
1. **Các thao tác đã thực hiện trong vòng rework:**
   - Đã rà soát và xóa hầu hết các bounding box vẽ thừa (False Positive) ở các xe nằm ngoài vùng quan tâm (ROI), xe ở quá xa/mờ nhòe chân trời hoặc các xe ở làn đường phụ (điển hình tại các ảnh `drive_001`, `drive_003`, `drive_018`, `drive_028`, `drive_049`, `drive_050`).
   - Nhờ đó, số lượng box thừa giảm mạnh (FP giảm từ 353 xuống 175), giúp **Precision tăng vọt từ 0.5808 lên 0.7360**, kéo **F1 tăng từ 0.6542 lên 0.7416**.
   - Độ bám sát hình học của bounding box tiếp tục duy trì ở mức rất cao và ổn định (**mean_iou_of_matches = 0.8311**, vượt xa ngưỡng yêu cầu 0.70).

2. **Phát hiện bất thường trong bộ nhãn Gold và quyết định giữ nguyên:**
   - Khi tiếp tục đối chiếu chi tiết các lỗi False Negative và False Positive còn lại, học viên nhận thấy một số lượng đáng kể nhãn trong tập `gold_train_labels` bị gán nhầm/lệch class so với schema chuẩn của bài:
     - Phương tiện thuộc loại xe tải (`truck`) trong gold bị gán nhãn thành `bus`.
     - Phương tiện thuộc loại xe buýt/khách (`bus`) lại bị gán nhãn thành `van`.
     - Phương tiện dạng xe van/minivan (`van`) lại bị gán nhãn thành `truck`.
   - Căn cứ theo quy chế làm bài của BTC: *"Không sửa gold labels; chỉ sửa nhãn do mình tạo và ghi lý do"* và tinh thần bám sát schema, học viên quyết định **giữ nguyên phân lớp chuẩn theo đúng `GUIDELINE_MINI.md`** đã thiết lập ban đầu và dừng rework tại mốc $F1 = 0.7416$, bảo toàn tính nhất quán của bộ nhãn cá nhân thay vì sửa sai theo nhãn gold để đối phó chỉ số.

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

Ghi công cụ đã dùng, dùng ở bước nào và phần đã tự kiểm tra; không dùng thì ghi rõ. Liệt kê nguồn ngoài repo được sử dụng. Không ghi mật khẩu/token hoặc đính kèm gold train.
