# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Họ và tên: Nguyễn Việt Hoàng

Công cụ gán nhãn đã dùng: YOLO26x Detection

## 1. Dữ liệu và cách chia tập

Video là một cảnh quay từ camera cố định; các frame cách nhau 0,4 giây thường chứa cùng xe và nền. Chia theo thời gian, kèm vùng đệm tối thiểu 4,4 giây giữa pool và test, giảm nguy cơ cùng xe xuất hiện ở cả hai tập. Chia ngẫu nhiên dễ làm số đo test cao rất fake do rò rỉ các cảnh gần trùng, dù cách chia hiện tại cũng chưa bảo đảm độc lập hoàn toàn.

## 2. Mô hình khởi đầu lạnh (cold start)

| Vòng | Model | Ảnh train | Box train | AP50 | Δ AP50 | P@0.25 | R@0.25 | F1 | R small | R medium | R large |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 0 | yolov8n cold start (COCO car+bus+truck) | 0 | 0 | 0.771 | — | 0.925 | 0.489 | 0.640 | 0.182 | 0.547 | 0.561 |

Trong `compare_round0.jpg`, nhiều box tham chiếu ở xa, tối, chỉ lộ đèn bị bỏ sót; các xe gần, rõ thân thường khớp hơn. Recall xe nhỏ chỉ 0,182 so với xe vừa 0,547 và xe lớn 0,561, nên lỗi tập trung ở xe nhỏ. Trước khi coi một box tham chiếu rất xa ở `frame_0050.jpg` là xe bị bỏ sót, cần người kiểm tra xem đó là thân xe hay chỉ hai chấm đèn/ánh phản chiếu; nhãn test do model tạo chưa được rà thủ công.

## 3. Chiến lược chọn mẫu

Điểm chọn bằng `0,5U + 0,3A + 0,2D`: `U` đo độ phân vân của năm box khó nhất, `A` đo số box có confidence mập mờ, `D` đo khoảng cách tới ảnh đã gán nhãn. Ở vòng đầu `D=1` cho mọi ảnh. `MIN_GAP_S=2` loại ứng viên cách ảnh đã chọn dưới 2 giây để bớt rà cảnh gần trùng. Ưu tiên `frame_0182.jpg` (0,9591; 18 box mập mờ), `frame_0331.jpg` (0,9154; 18 box mập mờ) và `frame_0369.jpg` (0,9324; 16 box mập mờ). `frame_0326.jpg` vẫn được model chọn vào lô 12 nhưng không lấy trong ngân sách năm ảnh: chỉ cách 0331 đúng 2 giây, có 15 box mập mờ; rà cả hai tốn công và có thể ít thông tin mới. Điểm bất định là tín hiệu để rà soát, không chứng minh nhãn đúng hay fine-tune sẽ cải thiện test.

## 4. Các vòng học chủ động (active learning)

| Vòng | Model | Ảnh train | Box train | AP50 | Δ AP50 so cold start | P@0.25 | R@0.25 | F1 | R small | R medium | R large |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 0 | yolov8n cold start (COCO car+bus+truck) | 0 | 0 | 0.771 | — | 0.925 | 0.489 | 0.640 | 0.182 | 0.547 | 0.561 |
| 1 | yolov8n fine-tune vong 1..1 | 12 | 328 | 0.436 | -0.335 | 1.000 | 0.092 | 0.168 | 0.000 | 0.054 | 0.512 |

Vòng 0 chưa có pre-label để sửa. Ở vòng 1, từ 169 box gợi ý, giữ 138, chỉnh 13, xóa 18 và thêm 177; còn 328 box trên 12 ảnh (`round1_diff.md`). AP50 giảm 0,3352 so với cold start, cũng là mức giảm so với vòng trước duy nhất. Recall xe nhỏ giảm 0,1818 → 0; xe vừa 0,5473 → 0,0541; xe lớn 0,5610 → 0,5122. P@0.25 tăng lên 1,000 vì chỉ còn 37 TP và 0 FP, trong khi FN tăng từ 206 lên 366: đây không phải cải thiện độ phủ.

Ở `frame_0050.jpg` trong `compare_round1.jpg`, TP giảm 11 → 3 và FN tăng 7 → 15 so với cold start; nhiều xe xa và xe vừa không còn được khớp ở ngưỡng 0,25. Có thể confidence giảm hoặc mô hình thiên về xe lớn sau fine-tune; cần kiểm dự đoán gốc và ngưỡng trước khi kết luận nguyên nhân. `BLIND_SCAN.md` ghi quan sát độc lập 22 xe ở `frame_0099.jpg`; đó không phải số box model. Sau rà soát, riêng frame này có 10 box thêm và 4 box chỉnh (`round1_diff.md`); `REVIEW_LOG.csv` ghi thêm phần xe bị cắt ở góc dưới phải, chỉ khoanh phần còn trong ảnh theo guideline, và xóa vệt đèn phản chiếu ở `frame_0107.jpg`. Kết quả trên ảnh test sau train là bằng chứng khác với lỗi pre-label đã sửa.

## 5. Kết luận và giới hạn

Vòng 1 kém cold start trên cùng test: AP50 0,7714 → 0,4362, recall 0,4888 → 0,0918. Tạm dừng train thêm để kiểm quy trình vì thêm ngay nhãn tốn công và có thể lặp lại lỗi. Nếu rà vòng sau, ưu tiên `frame_0024.jpg` (9,6 s: vòng 2 chỉ dự đoán 1 box dù contact sheet còn nhiều xe) và `frame_0072.jpg` (28,8 s: 6 box dự đoán, có 4 box mập mờ). Hai mốc cách nhau 19,2 giây; tránh lấy thêm `frame_0070.jpg` chỉ cách 0072 0,8 giây. Cần dự trù nhiều box phải vẽ tay như vòng 1 đã thêm 177 box.

Trước vòng train nữa, kiểm thống nhất class `car`, tọa độ và box đã sửa, tập train/validation, checkpoint, thay đổi confidence và dự đoán theo kích thước; đối soát log 0182. Test chỉ có 20 ảnh; 14 box tham chiếu cao dưới 16 px được bỏ qua, nên số đo không phản ánh xe cực nhỏ. Nhãn tham chiếu do model tạo chưa được người rà, nên AP50 chỉ đo mức khớp với chúng, chưa chứng minh chất lượng thực.
