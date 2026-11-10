# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Ngô Xuân Tùng
- MSSV: 2A202603225
- Email: 26ai.tungnx2@vinuni.edu.vn
- Lớp: L2A
- Link repo bài làm: https://github.com/zzwolfmoi1308/K5-L2A-DAY02-NgoXuanTung-2A202603225-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/1GQp9GI06kbwdHO7xWu2aZVhRiVe13Q6F?usp=sharing

## 2. Schema và mini guideline

- Chi tiết định nghĩa và quy tắc xem tại: [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md)
- **Tóm tắt ba ca mơ hồ đã quyết định:**
  1. **`drive_005.jpg` (Xe cứu hộ giao thông):** Đầu cabin giống van nhưng phía sau có kết cấu cơ khí, cần cẩu và càng kéo xe chuyên dụng lộ thiên $\rightarrow$ **Quyết định: `truck` (ID: 1)** theo định nghĩa phương tiện vận tải/kéo chở cơ giới, không phải khoang kín chở hàng nhẹ như `van`.
  2. **`drive_012.jpg` (Xe dáng cao cỡ nhỏ):** Phân vân giữa xe con hatchback/kei-car và minivan $\rightarrow$ **Quyết định: `car` (ID: 0)** vì xe có kích thước nhỏ, chiều dài cơ sở ngắn, 4–5 chỗ phục vụ gia đình/cá nhân, không có thân hộp dài chở khách hay khoang hàng thương mại.
  3. **`drive_019.jpg` (Minivan giao hàng):** Phân vân giữa xe con và minivan $\rightarrow$ **Quyết định: `van` (ID: 3)** vì xe có thiết kế thân hộp liền khối vuông vức và đặc biệt nhìn rõ **rãnh ray cửa trượt (sliding door rail)** bên hông sườn xe (*Nissan NV200*).
- **Phần cần giảng viên làm rõ:** Hiện tượng lệch phân lớp giữa `truck`, `bus` và `van` trong tập nhãn `gold_train_labels` và `eval` (ví dụ nhiều xe buýt lớn bị gán thành `van`, xe tải bị gán thành `bus`) cần được rà soát và chuẩn hóa lại để đảm bảo tính nhất quán của bộ ground truth.

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

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab, GPU T4, Python 3.13.16, Ultralytics 8.4.208, PyTorch 2.14.1+cu130
- Các tham số thay đổi so với notebook nếu có và lý do: Giữ nguyên tham số mặc định theo notebook (50 epochs, imgsz=640, batch=-1, seed=20260909) để đảm bảo tính nhất quán và khả năng tái lập kết quả.
- Đường dẫn best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.4070 |
| Recall | 0.2339 |
| mAP50 | 0.2737 |
| mAP50–95 | 0.2018 |

- Thư mục assets kết quả Colab: [assets/eval_predictions/](assets/eval_predictions/) và [assets/yolo26_runs/](assets/yolo26_runs/) (giải nén từ `day2_colab_outputs.zip`).

### Trả lời bốn câu hỏi cuối notebook:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - **Về số liệu thống kê:** Lớp **`bus`** có Recall ghi nhận là **`0.0`** (kế đến là `van` chỉ đạt `0.0833`, tức 1/12 đối tượng).
   - **Đối chiếu thực tế trên ảnh kết quả (`assets/eval_predictions/visuals/`):**
     - Bằng mắt thường, mô hình thực tế **đã phát hiện rất tốt các xe buýt lớn**: ví dụ xe buýt 2 tầng ở `drive_052.jpg` (dự đoán `bus 0.49`), xe buýt xanh dài và xe buýt đỏ ở `drive_053.jpg` (dự đoán `bus 0.85` và `bus 0.84`), xe buýt trắng ở `drive_056.jpg` (`bus 0.51`), xe buýt vàng ở `drive_058.jpg` (`bus 0.81`) và xe buýt trắng ở `drive_060.jpg` (`bus 0.89`).
     - **Nguyên nhân chỉ số Recall của `bus` về 0.0:** Khi đối chiếu file nhãn gốc của tập eval (`dataset/labels/eval/`), trong toàn bộ 10 ảnh eval chỉ có đúng 2 đối tượng được gán là `bus` (ở góc xa/khuất của `drive_051` và `drive_058`). Các xe buýt lớn rất rõ ràng ở `drive_052`, `drive_053`, `drive_056`, `drive_060` lại **bị nhãn chuẩn eval gán nhầm thành `van` (class 3)**. Vì mô hình học theo guideline đúng (xe buýt to là `bus`), nó dự đoán là `bus` nhưng không tìm thấy nhãn `bus` tương ứng trong ground truth của eval, đồng thời bỏ sót 2 xe buýt nhỏ ở góc xa trong nhãn eval, dẫn đến Recall bị tính bằng 0.0.
     - Lớp có Recall thấp nhất trên thực tế còn lại là **`van`** (`0.0833`): Ở ảnh `drive_053.jpg` và `drive_056.jpg`, nhiều xe trong nhãn eval là `van` nhưng mô hình chỉ bắt được 1 xe, còn lại bị bỏ sót hoặc nhận diện nhầm.

2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Cần kiểm tra **độ bám sát và độ ôm khít của bounding box (box tightness / localization accuracy)**.
   - **Dữ liệu thực tế:** Trong bảng kết quả, lớp `car` có `mAP50 = 0.738` (khá cao) nhưng `mAP50–95 = 0.523` (tụt sâu hơn 21%); tính trung bình toàn bộ các lớp thì `mAP50 = 0.2737` trong khi `mAP50–95 = 0.2018`.
   - **Giải thích từ ảnh thực tế (`assets/eval_predictions/visuals/`):**
     - Quan sát ảnh `drive_059.jpg` (xe SUV màu đen `car 0.87`) hoặc `drive_056.jpg`: Mô hình khoanh đúng vùng có xe (đạt ngưỡng dễ $IoU \ge 0.50$ nên mAP50 cao), nhưng viền box phía dưới bị lấn một chút xuống mặt đường/bóng đổ, hoặc hai bên hông bị thừa một khoảng pixel.
     - Ngưỡng mAP50 chỉ yêu cầu mức chồng lấn tương đối ($IoU \ge 0.50$, chạm một nửa thân xe là tính đúng). Trong khi mAP50–95 là trung bình cộng khắt khe trên 10 ngưỡng IoU từ 0.50 đến 0.95 (bước nhảy 0.05). Khi box dự đoán bị lệch hoặc rộng hơn xe thật từ 5–10%, ở các ngưỡng IoU cao ($IoU \ge 0.75, 0.85, 0.95$), box sẽ bị tính là trượt, làm chỉ số mAP50–95 bị tụt sâu.

3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Hai lỗi phân loại trực tiếp liên quan đến guideline:
     1. **Nhầm lẫn giữa `bus` và `van`:** Mô hình phát hiện chuẩn xác các xe buýt lớn ở `drive_052`, `drive_053`, `drive_060` là `bus`, nhưng trong ground truth của eval chúng lại bị gán là `van`.
     2. **Nhầm lẫn giữa `car` và `van` (thể hiện rõ nét nhất trên ảnh):**
        - Tại ảnh **`drive_051.jpg`**: Chiếc sedan màu đỏ góc dưới bên trái rõ ràng là xe con nhưng mô hình dự đoán nhầm thành **`van 0.99`** (box xanh ngọc).
        - Tại ảnh **`drive_054.jpg`**: Chiếc xe con màu đỏ chạy làn bên phải hoàn toàn là xe con nhưng mô hình dự đoán nhầm thành **`van 0.97`**.
        - Tại ảnh **`drive_056.jpg`**: Chiếc xe màu đen bên làn phải sát lề đường là xe con nhưng bị dự đoán là **`van 0.83`**.
     - **Dữ kiện hỗ trợ:** Lớp `van` có Precision chỉ đạt `0.136` (tức là 86% trường hợp mô hình dự đoán là `van` đều bị sai, phần lớn đoán nhầm vào xe con màu đỏ hoặc đuôi đứng). Điều này phản ánh đúng vùng ranh giới mơ hồ mà `GUIDELINE_MINI.md` đã phải thảo luận và thiết lập tiêu chí nhận diện ở các ca `drive_012` (xe hatchback trần cao) và `drive_019` (minivan có ray trượt).

4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - **Vì sao không dùng để train:**  
     - Để tuân thủ nguyên tắc độc lập dữ liệu trong học máy, ngăn chặn triệt để hiện tượng **Rò rỉ dữ liệu (Data Leakage)** và **Học vẹt (Overfitting)**.
     - Nếu cho mô hình học thuộc lòng cả 10 ảnh eval, các trọng số mạng sẽ tối ưu hóa cục bộ trên chính 10 ảnh đó, khiến điểm số đo được cao giả tạo nhưng khi triển khai trên hệ thống ADAS thực tế gặp các cung đường, thời tiết hay góc máy mới sẽ mất hoàn toàn khả năng tổng quát hóa.
   - **Vai trò trong notebook:**  
     - 10 ảnh eval đóng vai trò là **Validation Set (Tập kiểm định độc lập)**:
     - Sau mỗi epoch huấn luyện (trong 50 epochs), notebook chạy mô hình qua 10 ảnh này để đo validation loss (`val/box_loss`, `val/cls_loss`) và mAP. Từ đó, hàm huấn luyện tự động theo dõi và lưu lại checkpoint có kết quả tốt nhất thành file trọng số **`best.pt`**.
     - Toàn bộ quá trình này chỉ đọc ảnh eval để đánh giá, hoàn toàn **không tính gradient ngược để cập nhật trọng số** của mô hình.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| **CP1 & CP2 — Phân định các ca mơ hồ (Schema)** | Khó phân biệt giữa xe con trần cao (hatchback/SUV) với minivan (`drive_012`, `drive_019`), và xe cứu hộ giao thông đầu giống van (`drive_005`). | Xây dựng tiêu chí phân biệt cụ thể trong `GUIDELINE_MINI.md`: dựa vào rãnh ray trượt (sliding door rail), thân hộp liền khối và mục đích chuyên chở để xếp `van` (ID 3); dựa vào sàn cẩu cơ giới chuyên dụng để xếp `truck` (ID 1). | Áp dụng nhất quán trên toàn bộ 50 ảnh train, không bị mâu thuẫn tiêu chuẩn giữa các ảnh. |
| **CP2 — Nạp nhãn chuẩn vào CVAT** | Khi nạp nhãn gold vào CVAT để so sánh, hệ thống báo lỗi thiếu file cấu hình `*.yaml` do chọn nhầm định dạng Ultralytics YOLO. | Chuyển sang định dạng native của CVAT là `CVAT 1.1` (XML) hoặc đóng gói đúng cấu trúc gồm `data.yaml` và các thư mục labels. | Nạp nhãn chuẩn thành công 100% vào một Task riêng trên CVAT để mở song song đối chiếu mà không bị đè nhãn cá nhân. |
| **CP5 — Đánh giá nhãn với Gold Set lần 1** | Chỉ số F1 ban đầu chưa đạt (F1 = 0.6542 < 0.80) do số lượng False Positive cao (FP = 353); học viên gán tới 842 box so với 653 box của Gold. | Mở song song tab Job của mình và Job Gold trên CVAT; rà soát và xóa các box vẽ xe ở quá xa, mờ nhòe chân trời hoặc làn đường phụ nằm ngoài vùng quan tâm (ROI) tại các ảnh chênh lệch lớn (`drive_001`, `drive_003`, `drive_018`, `drive_028`, `drive_049`, `drive_050`). | Giảm mạnh số box thừa (FP giảm từ 353 xuống 175), kéo Precision tăng vọt từ 0.5808 lên 0.7360, F1 tăng lên 0.7416; mean IoU duy trì ở mức cao 0.8311. |
| **CP5 — Rework và đối chiếu phân lớp với Gold** | F1 dừng ở mức 0.7416. Khi kiểm tra chi tiết các lỗi lệch còn lại, phát hiện tập `gold_train_labels` bị gán nhầm chéo class: `truck` bị gán là `bus`, `bus` bị gán là `van`, và `van` bị gán là `truck`. | Đối chiếu với quy chế bài thi: *"Không sửa nhãn gold, không sửa nhãn của mình sai theo gold chỉ để vượt gate"*. Quyết định giữ nguyên nhãn chuẩn đã xây dựng theo `GUIDELINE_MINI.md` thay vì sửa sai theo gold để ép chỉ số. | Giữ vững tính nhất quán và liêm chính học thuật của bộ nhãn cá nhân; trình bày giải trình chi tiết trong báo cáo để chấm theo tiêu chí rework của Rubric. |

## 7. Tự đánh giá theo rubric

Đối chiếu khách quan và trung thực theo từng tiêu chí trong [RUBRIC.md](../../RUBRIC.md):

| Tiêu chí | Điểm tối đa | Điểm tự đánh giá | Kết quả thực tế & Minh chứng | Phần còn thiếu / Hạn chế |
|---|---:|---:|---|---|
| **1. Hoàn thành và định dạng YOLO** | 10 | **10 / 10** | Đủ 50/50 file nhãn `.txt` cùng basename với ảnh; đúng 5 cột toạ độ normalized trong `[0, 1]`; đúng 4 class ID (0–3); script `check_yolo_labels.py` chạy đạt 50/50. | Không có thiếu sót. |
| **2. Đầy đủ vật thể** | 20 | **16 / 20** | Độ bao phủ tốt, bắt được 488 xe hợp lệ; Recall đối chiếu với Gold đạt `0.7473` (gần 75%). | Vẫn còn 165 xe bị bỏ sót (FN = 165), chủ yếu ở các xe rất nhỏ ở chân trời xa hoặc các góc khuất bị che lấp nhiều mà học viên chủ động không gán để tránh noise. |
| **3. Độ bám sát và hình học box** | 20 | **20 / 20** | Điểm `mean_iou_of_matches` đạt **0.8311 (83.11%)**, vượt xa ngưỡng yêu cầu (0.70). Bounding box ôm rất sát thân xe, không lấy thừa mặt đường, bóng đổ hay bầu trời. | Không có lỗi nghiêm trọng về hình học box. |
| **4. Phân lớp nhất quán** | 15 | **14 / 15** | Tuân thủ nhất quán định nghĩa schema 4 lớp; không gán các vật thể ngoài scope (xe máy, xe đạp, người đi bộ). Đã phát hiện và sửa lỗi gán nhầm van thành truck ở `drive_009`. | Vẫn còn thách thức ở ranh giới giữa xe minivan cỡ nhỏ và xe con trần cao (SUV/hatchback) ở một số góc nhìn xa. |
| **5. Mini guideline và quyết định mơ hồ** | 10 | **10 / 10** | Hoàn thành đầy đủ [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md) với định nghĩa 4 lớp, 4 quy tắc vẽ box và 3 ca mơ hồ cụ thể (`drive_005`, `drive_012`, `drive_019`) có lập luận kỹ thuật rõ ràng theo schema. | Không có thiếu sót. |
| **6. Gold-set annotation QC** | 15 | **10 / 15** | Đã thực hiện vòng Rework nghiêm túc: giảm mạnh FP từ 353 xuống 175, kéo Precision tăng từ 0.58 lên 0.736, F1 tăng từ 0.65 lên 0.7416; Mean IoU đạt 0.8311. Có phần giải trình chi tiết về việc phát hiện nhãn Gold bị gán lệch chéo class (`truck` $\leftrightarrow$ `bus` $\leftrightarrow$ `van`) và quyết định giữ nguyên liêm chính nhãn theo rubric. | F1 dừng ở mốc `0.7416` (chưa chạm ngưỡng tự động 0.80), nhưng đủ điều kiện xét điểm theo nhánh *"hoặc có vòng rework được giải thích rõ"*. |
| **7. Pipeline YOLO và phân tích** | 10 | **10 / 10** | Chạy Colab thành công trên GPU Tesla T4; có file `outputs/model_eval_metrics.json` và nộp đủ 2 folder `assets/eval_predictions/` và `assets/yolo26_runs/`; trả lời sâu sắc cả 4 câu hỏi phân tích gắn liền với ảnh dự đoán thực tế. | Không có thiếu sót. |
| **TỔNG ĐIỂM** | **100** | **90 / 100** | **Xếp loại: Xuất sắc / Giỏi.** Bài làm tuân thủ đầy đủ quy trình, trung thực về số liệu và bảo vệ vững chắc tính nhất quán của dữ liệu. | |

## 8. Công cụ AI và tài liệu tham khảo

- **Công cụ AI sử dụng:** Google Gemini.
- **Các bước và phạm vi sử dụng:**
  1. *Phân định rõ schema & ca mơ hồ (CP1):* Tham khảo gợi ý về các đặc trưng nhận diện hình thái xe (như rãnh ray trượt của minivan, sàn cẩu cơ giới của xe cứu hộ giao thông) để hỗ trợ thiết lập bảng quyết định cho 3 ca mơ hồ trong `GUIDELINE_MINI.md`.
  2. *Rà soát và nhận xét output từ Colab (CP6):* Sử dụng AI để hỗ trợ đọc hiểu bảng chỉ số metrics (`precision`, `recall`, `mAP50`, `mAP50–95`), đối chiếu các hiện tượng quan sát được trên 10 ảnh kết quả dự đoán và hỗ trợ trau chuốt câu từ cho phần trả lời 4 câu hỏi phân tích ở Mục 5.
- **Phần học viên tự thực hiện và tự kiểm tra (Bảo đảm liêm chính học thuật):**
  - Tự tay vẽ bounding box và gán nhãn thủ công toàn bộ 50 ảnh train trên CVAT.
  - Tự tay thực hiện vòng rework, xóa các box thừa ngoài ROI và kiểm tra lại class.
  - Tự chạy toàn bộ các script `check_yolo_labels.py`, `evaluate_annotations.py` trên máy cục bộ và chạy notebook huấn luyện trên Google Colab.
  - Mọi số liệu và lập luận trong báo cáo đều được học viên tự đối chiếu, xác thực trực tiếp trên dữ liệu và kết quả thực tế.
- **Tài liệu tham khảo ngoài repo:**
  - Tài liệu chính thức của Ultralytics YOLO: [https://docs.ultralytics.com](https://docs.ultralytics.com)
  - Hướng dẫn định dạng dữ liệu CVAT: [https://docs.cvat.ai](https://docs.cvat.ai)
