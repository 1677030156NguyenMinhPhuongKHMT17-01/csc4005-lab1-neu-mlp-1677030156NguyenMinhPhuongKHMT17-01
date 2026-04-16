# CSC4005 – Lab 1 Report

## 1. Mục tiêu
Mục tiêu của bài lab là tìm hiểu và xây dựng một pipeline huấn luyện hoàn chỉnh cho mạng nơ-ron (MLP) dựa trên bộ dữ liệu ảnh NEU Surface Defect Database (phân loại 6 lớp lỗi: Crazing, Inclusion, Patches, Pitted_Surface, Rolled-in_Scale, Scratches). Thông qua việc điều chỉnh các siêu tham số (hyperparameters), em quan sát sự khác biệt giữa các cấu hình, đánh giá hiện tượng overfitting/underfitting, sự tác động của regularization và so sánh hiệu quả của các Optimizer (AdamW vs SGD).

## 2. Cấu hình thí nghiệm
Các yếu tố cấu hình chung được giữ nguyên: Epoch = 20, Batch Size = 32, Image Size = 64, Early Stopping Patience = 5, có sử dụng data augmentation (--augment).  
3 cấu hình cụ thể theo đúng chuẩn yêu cầu của Lab đã chạy trên **W&B**:
1. **Run A (baseline_adamw)**: Optimizer = **AdamW**, LR = 0.001, Weight Decay = 0.0001, Dropout = **0.3**
2. **Run B (run_b_sgd)**: Optimizer = **SGD**, LR = **0.01**, Weight Decay = **0.0**, Dropout = **0.3**
3. **Run C (run_c_strong_reg)**: Optimizer = **AdamW**, LR = **0.0005**, Weight Decay = **0.001**, Dropout = **0.5**

## 3. Kết quả
Bảng tổng hợp kết quả trích xuất từ dữ liệu log thực tế:

| Run Name | Optimizer | LR | Weight Decay | Dropout | Best Val Loss | Best Val Acc | Test Acc |
|----------|-----------|----|--------------|---------|---------------|--------------|----------|
| aseline_adamw | AdamW | 0.001 | 0.0001 | 0.3 | 1.499 | 41.85% | 38.15% |
| un_b_sgd | SGD | 0.01 | 0.0 | 0.3 | **1.443** | **48.89%** | **47.41%** |
| un_c_strong_reg | AdamW | 0.0005 | 0.001 | 0.5 | 1.628 | 32.96% | 34.44% |

*Nhận xét từ biểu đồ learning curve:*
- Lượt chạy **SGD (un_b_sgd)** đạt hiệu suất cao nhất. Với Learning Rate được chỉnh lên 0.01, tốc độ giảm loss và tăng độ chính xác vượt trội hơn hẳn so với AdamW baseline.
- Lượt chạy **Baseline (aseline_adamw)** học khá nhanh ở những epoch rọi nhưng bắt đầu dao động lớn về độ chính xác xác cũng như loss.
- Lượt chạy **Strong Regularization (un_c_strong_reg)** (đường curve cao nhất ở loss) do bị kìm kẹp bởi Dropout cao (0.5), Weight Decay lớn (0.001) và LR thấp (0.0005) nên tiến trình học bị chậm đi rõ rệt, kết quả accuracy xếp chót trong 3 cấu hình sau 20 epoch.

## 4. Phân tích
- **Cấu hình nào tốt nhất?** 
  Cấu hình **un_b_sgd** là cấu hình làm việc tốt nhất trên tập Validation và cả tập Test thực tế.
- **Dấu hiệu overfitting / underfitting là gì?** 
  - *Underfitting:* Mô hình un_c_strong_reg đang có dấu hiệu underfitting (tốc độ hội tụ quá chậm). Việc kết hợp giữa giảm tốc độ học (LR nhỏ) cộng với việc tăng hình phạt quá khắc nghiệt (Dropout = 0.5, Weight Decay gấp 10 lần) làm mô hình MLP bị thu hẹp đáng kể khả năng tối ưu hóa trọng số. Mất 20 epoch nhưng val_loss vẫn chỉ xuống được ~1.62 và kết quả độ chính xác chỉ nhích lên được tầm hơn 30%.
- **AdamW và SGD khác nhau ra sao trong thí nghiệm của bạn?**
  Mặc dù AdamW nổi tiếng là hội tụ cực nhanh (như ở aseline), nhưng trong bài toán cụ thể này nếu chọn đúng hệ số học phù hợp (LR=0.01 cho SGD), thì mô hình sử dụng thuật toán kinh điển **SGD** lại vươn lên dẫn đầu. SGD không có thành phần momentum nội tại cực tinh vi như AdamW nhưng bù lại với learning rate lớn, bộ trọng số tránh được những hố nông và đạt được độ chính xác val/test xuất sắc (48.89%).

## 5. Kết luận
- **Best Config:** Lựa chọn un_b_sgd (Optimizer: SGD, LR: 0.01, Weight Decay: 0.0, Dropout: 0.3).
- **Lý do:** Đây là mô hình đạt **Validation Accuracy** cực đại cao nhất (48.89%), và **Test Accuracy** cao nhất (47.41%). SGD kết hợp với LR=0.01 đã cung cấp bước tính toán độ dốc phù hợp nhất trên không gian tập dữ liệu ảnh bị nhiễu hạt này. Khả năng dự đoán trên Confusion Matrix của mô hình tốt hơn rệt, ít bị thiên lệch tập trung vào một lớp so với các model còn lại.
