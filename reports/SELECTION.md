# Vì sao chọn lô này?

Nếu chỉ được rà **năm ảnh** trong 50 dòng đầu của `outputs/selection_round1.csv`, ưu tiên các frame sau:

| Ưu tiên rà | Frame | Điểm | Thời điểm | Thứ tự CSV | Lý do |
| ---: | --- | ---: | ---: | ---: | --- |
| 1 | `frame_0182.jpg` | 0.9591 | 72,8 s | 1 | Điểm cao nhất; 18/28 box dự đoán ở dải mập mờ, nên có nhiều vị trí cần người xác nhận. Contact sheet cho thấy các xe nhỏ và đèn xe trong cảnh tối. |
| 2 | `frame_0099.jpg` | 0.9063 | 39,6 s | 8 | `U=0.9460`, 14/29 box mập mờ; ở mốc thời gian sớm, giúp lô năm ảnh trải rộng. Bản quét độc lập `BLIND_SCAN.md` cũng ghi hai vị trí ở rìa ảnh cần xem kỹ, trong đó một vị trí chỉ thấy đèn xe và chưa đủ chắc để gán box. |
| 3 | `frame_0270.jpg` | 0.8878 | 108,0 s | 13 | Cách xa hai ảnh trên; có 14/35 box mập mờ. Contact sheet cho thấy một xe tải lớn ở gần camera và nhiều xe nhỏ phía xa, hữu ích để rà ranh giới xe và các trường hợp dễ bỏ sót. |
| 4 | `frame_0331.jpg` | 0.9154 | 132,4 s | 5 | Có 47 box dự đoán, 18 box mập mờ (`A=1`). Ưu tiên ảnh này hơn `frame_0326.jpg` ở 130,4 s vì hai ảnh chỉ cách 2 giây và trông cùng một đoạn giao thông; ảnh 0331 có nhiều box và vị trí mập mờ hơn (47/18 so với 39/15), dù điểm tổng thấp hơn 0.0001. |
| 5 | `frame_0369.jpg` | 0.9324 | 147,6 s | 2 | Điểm cao thứ hai, `U=0.9315`, 16/43 box mập mờ. Đây là một mốc muộn hơn 0331 khoảng 15 giây; contact sheet cho thấy luồng xe gần camera khác vị trí, đáng rà riêng. |

Ba ví dụ trong **lô 12 ảnh model đã chọn**: `frame_0182.jpg` (hạng 1, `selected=True`, 72,8 s), `frame_0331.jpg` (hạng 5, `selected=True`, 132,4 s) và `frame_0369.jpg` (hạng 2, `selected=True`, 147,6 s). Cả ba đều có ô ảnh kèm tên, thời điểm và điểm làm tròn trong `outputs/selection_round1.jpg`.

**Ảnh điểm cao không ưu tiên trong ngân sách năm ảnh:** `frame_0326.jpg` đứng hạng 4, điểm 0.9155, `selected=True` trong lô 12. Nó chỉ cách `frame_0331.jpg` 2 giây và có ít box mập mờ hơn, nên tôi dành suất rà cho mốc 108,0 s của `frame_0270.jpg`. Luật `MIN_GAP_S=2` vẫn cho cả 0326 và 0331 vào lô 12 vì khoảng cách đúng 2 giây; khi chỉ có năm suất, tôi thận trọng hơn với ảnh gần trùng. Tương tự, `frame_0372.jpg` hạng 6, điểm 0.9101 nhưng `selected=False`, cách `frame_0369.jpg` 1,2 giây, phù hợp với việc bị lọc theo khoảng cách thời gian.

Trong 50 dòng đầu, cột `empty` đều là `False`: chưa có ca model không dự đoán được box để đối chiếu trong phạm vi này. `D=1` ở cả 50 dòng vì đây là vòng đầu, chưa có frame đã gán nhãn; vì thế phải xem thời điểm và contact sheet để cân nhắc ảnh gần trùng. Số box, số box mập mờ và điểm chỉ mô tả **dự đoán của model**, không phải số xe thật hay mức sai nhãn. Phép chọn này chưa chứng minh năm ảnh sẽ cải thiện mô hình, chưa đo được độ đúng của các box hoặc số xe bị bỏ sót, và cũng chưa chứng minh mô hình hoạt động tốt trên video khác. Cần người rà nhãn và đánh giá lại trên tập kiểm thử độc lập; tập test hiện chỉ có 20 ảnh với nhãn tham chiếu do model tạo, chưa được rà thủ công.
