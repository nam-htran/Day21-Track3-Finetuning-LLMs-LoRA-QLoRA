# Lab 21 — Evaluation Report

**Học viên**: <Họ tên> — <MSSV>  (Vui lòng điền thông tin của bạn)
**Ngày nộp**: 2026-06-25
**Submission option**: C (Code-only)

## 1. Setup
- **Base model**: `unsloth/Qwen2.5-3B-bnb-4bit`
- **Dataset**: Vietnamese Alpaca, 200 samples (180 train + 20 eval)
- **max_seq_length**: 512 (p95 = ~400, rounded up)
- **GPU**: Tesla T4, 15.6 GB VRAM
- **Training cost**: $0.07 (~12 phút @ $0.35/hr)

## 2. Rank Experiment Results

| Rank | Trainable Params | Train Time | Peak VRAM | Eval Loss | Perplexity |
|------|-----------------|------------|-----------|-----------|------------|
| 8    | 1,843,200       | 3.92 min   | 7.21 GB   | 1.557694  | 4.747861   |
| 16   | 3,686,400       | 4.21 min   | 6.61 GB   | 1.516083  | 4.554351   |
| 64   | 14,745,600      | 3.88 min   | 7.99 GB   | 1.476815  | 4.378976   |
| Base | -               | -          | -         | N/A       | N/A        |

## 3. Loss Curve Analysis
*(Loss curve đã được hiển thị trong notebook)*
- Quan sát: Train loss có xu hướng giảm dần qua các epochs. Đánh giá eval loss và perplexity giảm đều khi rank tăng (r=8 đến r=64), cho thấy mô hình học được các pattern từ tập dữ liệu. Không có dấu hiệu overfitting nghiêm trọng do eval perplexity vẫn giảm thuận lợi.

## 4. Qualitative Comparison (5 examples)

### Example 1
**Prompt**: Giải thích khái niệm machine learning cho người mới bắt đầu.
**Base**: Machine learning là một phân khúc của trí tuệ nhân tạo, nó tập trung vào việc thiết lập các mô hình máy móc để học tập từ dữ liệu...
**Fine-tuned (r=16)**: Machine learning là một bộ môn công nghệ máy tính dựa trên việc học tập và cải thiện các dự đoán dựa trên dữ liệu mà không có sự hướng dẫn trực tiếp...
**Nhận xét**: Cả hai đều cho câu trả lời tốt, bản Fine-tuned giải thích với cấu trúc gọn gàng hơn và đưa ra các nhóm thuật toán (có giám sát, không giám sát).

### Example 2
**Prompt**: Viết đoạn code Python tính số Fibonacci thứ n.
**Base**: Để tính số Fibonacci thứ n, bạn có thể sử dụng hàm đệ quy hoặc vòng lặp. Đây là một đoạn mã Python cho phép bạn tính số Fibonacci thứ n: `if n <= 0: return "N p...`
**Fine-tuned (r=16)**: Để tính số Fibonacci thứ n, bạn có thể viết một đoạn code Python như sau: `def fibonacci(n): if n < 0: raise ValueError("Input phải là một số nguyên dương.") elif n == 0:`...
**Nhận xét**: Bản Fine-tuned có xử lý exception validation chặt chẽ hơn (`raise ValueError`) theo style code Pythonic chuyên nghiệp thay vì chỉ trả về string.

### Example 3
**Prompt**: Liệt kê 5 nguyên tắc thiết kế UI/UX.
**Base**: 1. Thân thiện với người dùng: Mục đích của thiết kế UI/UX là cung cấp trải nghiệm...
**Fine-tuned (r=16)**: 1. Chuyển đổi: UI/UX thiết kế phải hướng tới việc giúp người dùng thực hiện hành động. 2. Thích ứng: UI/UX thiết kế phải thích ứng với nhiều loại thiết bị...
**Nhận xét**: Bản Fine-tuned trình bày các gạch đầu dòng dứt khoát, đi vào trọng tâm hơn so với cách diễn đạt hơi dài dòng của base model.

### Example 4
**Prompt**: Tóm tắt sự khác biệt giữa LoRA và QLoRA.
**Base**: LoRA (Low-Rank Adaptation) và QLoRA (Quantized LoRA) là hai phương pháp cải thiện hiệu năng của mô hình NLU (NLP) bằng cách sử dụng các phép biến đổi thấp độ phức tạp...
**Fine-tuned (r=16)**: LoRA (Layer-wise Adaptive Regularization Optimization) và QLoRA (Quantized LoRA) là hai phương pháp regularization...
**Nhận xét**: Bản FT có vẻ gặp hallucination khi giải nghĩa sai từ viết tắt LoRA thành Layer-wise Adaptive Regularization Optimization thay vì Low-Rank Adaptation. Base model lại đúng hơn ở điểm này. Cần chú ý thêm dữ liệu về LoRA vào tập train.

### Example 5
**Prompt**: Phân biệt prompt engineering, RAG, và fine-tuning.
**Base**: Prompt engineering, RAG (retrieval augmented generation), và fine-tuning là ba cách khác nhau để cải thiện hiệu suất...
**Fine-tuned (r=16)**: Prompt engineering, RAG và fine-tuning là ba kỹ thuật khác nhau được sử dụng trong lĩnh vực AI...
**Nhận xét**: Cả hai đều nêu được khái niệm cơ bản.

## 5. Conclusion về Rank Trade-off
- Rank nào cho ROI tốt nhất trên dataset này? Tại sao?
  Với rank=16, perplexity giảm so với rank=8 (4.74 -> 4.55), thời gian train chỉ tăng một chút (3.9 -> 4.2 phút) và lượng tham số huấn luyện vẫn ở mức rất nhỏ. Đây là điểm cân bằng hoàn hảo về thời gian, bộ nhớ và hiệu suất, mang lại ROI tốt nhất.
- Khi nào tăng rank không còn cải thiện perplexity (diminishing returns)?
  Khi tăng từ r=16 lên r=64, trainable params tăng gấp 4 lần, VRAM tăng nhưng perplexity chỉ giảm nhẹ (4.55 -> 4.37). Sự cải thiện này bắt đầu xuất hiện dấu hiệu của diminishing returns so với chi phí tài nguyên bỏ ra.
- Recommendation: nếu deploy production, bạn chọn rank nào? Tại sao?
  Nếu deploy production, tôi sẽ chọn rank=16. Rank=16 đủ để mô hình học được domain format/style mà không làm tăng đáng kể kích thước adapter, dễ dàng cho việc multi-tenant serving (nhiều adapter chạy trên cùng một GPU).

## 6. What I Learned
- Hiểu được rõ hơn đánh đổi (trade-off) giữa các mức rank khác nhau về thời gian huấn luyện và lượng RAM tiêu thụ.
- Nắm được cách pipeline SFT hoạt động: chuẩn bị dataset -> định hình prompt -> QLoRA wrapping -> Train.
- Nhận ra rằng mô hình fine-tuned vẫn có thể gặp hallucination nếu dataset không chứa đủ kiến thức liên quan (như ví dụ định nghĩa LoRA).
