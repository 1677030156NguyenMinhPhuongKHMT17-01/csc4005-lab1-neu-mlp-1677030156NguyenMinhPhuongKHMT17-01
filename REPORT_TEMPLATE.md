# CSC4005 – Lab 1 Report

## 1. Mục tiêu
Mục tiêu của bài lab là tìm hiểu và xây dựng một pipeline huấn luyện hoàn chỉnh cho mạng nơ-ron (MLP) dựa trên bộ dữ liệu ảnh NEU Surface Defect Database (phân loại 6 lớp lỗi bề mặt thép: Crazing, Inclusion, Patches, Pitted_Surface, Rolled-in_Scale, Scratches). Thông qua việc thay đổi các siêu tham số (hyperparameters), em đánh giá được hiện tượng overfitting, underfitting, sự hiệu quả của các phương pháp regularization (như dropout) và sự khác biệt giữa các trình tối ưu hóa (AdamW vs SGD).

## 2. Cấu hình thí nghiệm
Các yếu tố cấu hình chung được giữ nguyên: Epoch = 20, Batch Size = 32, Image Size = 64, LF = 0.001, Weight Decay = 0.0001, Early Stopping Patience = 5, có sử dụng augment.  
3 cấu hình cụ thể đã chạy để đối chiếu:
1. **Cấu hình 1 (Baseline)**: Optimizer = **AdamW**, Dropout = **0.3**
2. **Cấu hình 2 (Tăng Regularization)**: Optimizer = **AdamW**, Dropout = **0.5**
3. **Cấu hình 3 (Đổi Optimizer)**: Optimizer = **SGD**, Dropout = **0.3**

## 3. Kết quả
Bảng tổng hợp kết quả (đánh giá Validation cực đại):

| Run Name | Optimizer | Dropout | Best Val Loss | Best Val Acc | Test Acc |
|----------|-----------|---------|---------------|--------------|----------|
| `baseline_dropout_0.3` | AdamW | 0.3 | 1.499 | **41.85%** | **38.15%** |
| `exper_dropout_0.5` | AdamW | 0.5 | 1.760 | 18.15% | 18.15% |
| `exper_sgd_baseline` | SGD | 0.3 | 1.730 | 24.44% | 25.18% |

*(Đồ thị Learning Curves chi tiết được đính kèm bằng hình ảnh `curves.png` trong từng thư mục thuộc `outputs/`)*

*Nhận xét:*
- Baseline (AdamW + Dropout 0.3) có kết quả Val Loss và Val Acc tốt nhất trong khoảng thời gian train nhỏ (20 epochs).
- Khi đẩy Dropout lên quá cao (0.5), mô hình hoàn toàn mất khả năng học, độ chính xác Validation (18.15%) chỉ loanh quanh ở mức như việc đoán ngẫu nhiên 1 trong 6 lớp (khoảng 1/6 = 16.67%).
- SGD hội tụ rất chậm so với AdamW, do đó khi dừng huấn luyện ở 20 epochs, Val Acc của SGD khá thấp so với Baseline.

## 4. Phân tích
- **Cấu hình nào tốt nhất?** 
  Cấu hình `baseline_dropout_0.3` hoạt động tốt nhất.
- **Dấu hiệu overfitting / underfitting là gì?** 
  - *Underfitting:* Ở Cấu hình 2 (Dropout 0.5), ta thấy train_acc và val_acc đều rất thấp (18.15%), không tăng được. Mô hình đang bị underfitting do bị "phạt" quá nặng, dẫn tới việc không học nổi tính chất nào cả. Tương tự, SGD (Cấu hình 3) cũng bị underfitting ở giai đoạn đầu do tốc độ học quá chậm.
- **AdamW và SGD khác nhau ra sao trong thí nghiệm của bạn?**
  Với cùng mức Learning Rate (0.001) và số Epochs ngắn, AdamW thể hiện sức mạnh tối ưu cực kỳ nhanh, giúp độ chính xác vọt lên ngay ở những epoch đầu nhờ điều chỉnh tốc độ học tự động cho các tham số khác nhau. Trong khi đó, thuật toán lấy mẫu ngẫu nhiên SGD với tốc độ học cố định rơi vào tình trạng tiến triển rất chậm chạp. Nó cần số Epochs lớn hơn, hoặc Learning rate lớn hơn nhiều và thay đổi theo lịch trình (scheduler) thì mới bắt kịp AdamW.

## 5. Kết luận
- **Best Config:** Lựa chọn `baseline_dropout_0.3` (AdamW, Dropout: 0.3).
- **Lý do:** Mô hình này cho Validation Acc cao nhất (41.85%) và Test Acc cao nhất (38.15%). Việc dùng kết hợp giữa AdamW và Dropout ở mức vừa phải (0.3) giúp mạng học được các features một cách nhanh chóng mà không bị cản trở quá lớn. (Do các thí nghiệm trên mới dùng kiến trúc MLP đơn giản và Img_size 64 bé, nên độ chính xác tối đa chưa cao, nhưng nó là cấu hình tối ưu nhất để chứng minh mô hình học sâu có khả năng hội tụ nhanh).
