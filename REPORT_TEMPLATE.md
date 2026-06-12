# CSC4005 Lab 5 Report – Vision Transformer for Smart Campus Scene Classification

## 1. Thông tin nhóm/cá nhân

- Họ tên: Nguyễn Văn Đạt
- Mã sinh viên: 1671040007
- Lớp: KHMT 16_01
- Link GitHub repo: https://github.com/FIT-DNU-CS-16-01/csc4005-lab5-khmt_1601_nhom_11
- Link W&B dashboard: https://github.com/FIT-DNU-CS-16-01/csc4005-lab5-khmt_1601_nhom_11

## 2. Mô tả bài toán

Viết ngắn gọn 5–7 dòng:

- Bài toán cần giải quyết là gì?
    + Bài toán cần giải quyết là phân loại ảnh cảnh trong nhà (Indoor Scene Classification) bằng mô hình Vision Transformer (ViT). Hệ thống nhận đầu vào là một ảnh và dự đoán ảnh đó thuộc loại không gian nào trong số các lớp đã định nghĩa. Đây là một bài toán phân loại ảnh nhiều lớp trong lĩnh vực Computer Vision.
- Vì sao bài toán này phù hợp với bối cảnh Smart Campus?
    + Trong mô hình Smart Campus, việc nhận diện các khu vực trong trường học có thể hỗ trợ nhiều ứng dụng như quản lý cơ sở vật chất, điều hướng thông minh, giám sát an ninh và hỗ trợ người dùng tìm kiếm vị trí. Một hệ thống AI có khả năng tự động nhận biết các khu vực khác nhau từ hình ảnh sẽ góp phần nâng cao tính tự động hóa và hiệu quả quản lý trong môi trường học tập hiện đại.
- Các lớp cần phân loại là gì?
    + Trong bài lab này, tập dữ liệu được xây dựng từ MIT Indoor Scenes 67 và chỉ sử dụng 5 lớp gồm:

    classroom
    computerroom
    library
    corridor
    office

    Mỗi ảnh đầu vào sẽ được mô hình dự đoán thuộc một trong năm lớp trên.

## 3. Dữ liệu

| Nội dung | Mô tả |
|---|---|
| Dataset gốc | MIT Indoor Scenes 67 |
| Subset sử dụng | classroom: 113, computerroom: 114, library: 107, corridor: 346, office: 109 |
| Số ảnh mỗi lớp | 789 |
| Train/Val/Test split | 557/ 116/ 116 |
| Tiền xử lý | resize: 224x224, normalization theo ImageNet, Data augmentation |

## 4. Mô hình ViT

Mô tả ngắn gọn kiến trúc:

```text
image → patch embedding → positional embedding → transformer encoder → classification head
```
Mô hình sử dụng trọng số pretrained của Vision Transformer (ViT-B/16) trên ImageNet và chỉ huấn luyện lớp phân loại cuối cùng (head-only training).

Điền thông số:

| Thành phần | Giá trị |
|---|---|
| model_name | vit_b_16 |
| train_mode | head_only |
| img_size | 224 |
| batch size | 16 |
| số epoch | 10 |
| learning rate | 0.001 |
| optimizer | Adamw |
| total params | 85,802,501 |
| trainable params | 3,845 |
| trainable ratio | 0.0000448 (≈0.0045%) |

## 5. Kết quả

| Metric | Validation | Test |
|---|---:|---:|
| Accuracy | 93.97% | 91.38% |
| Macro-F1 | 93.21% | 88.11% |
| Best epoch | 10 | 10 |

Chèn ảnh:

- Learning curves
![Biểu đồ Loss và Accuracy](outputs/debug_smoke/curves.png)

- Confusion matrix
![Biểu đồ Loss và Accuracy](outputs/debug_smoke/confusion_matrix.png)
## 6. Phân tích lỗi

Trả lời:

1. Lớp nào mô hình dự đoán tốt nhất?
- Dựa trên kết quả tổng thể, các lớp có đặc trưng thị giác rõ ràng như library hoặc computerroom thường được nhận diện tốt do có nhiều đối tượng đặc trưng như giá sách, máy tính hoặc bàn làm việc.
2. Lớp nào dễ bị nhầm nhất?
- Các lớp classroom và office thường có khả năng bị nhầm lẫn nhiều hơn vì đều chứa bàn ghế, bảng hoặc không gian học tập/làm việc tương đối giống nhau.
3. Cặp lớp nào dễ nhầm với nhau? Vì sao?
- Classroom và office là cặp dễ bị nhầm nhất. Nguyên nhân là cả hai đều là môi trường trong nhà có bố cục tương tự, bao gồm bàn ghế, thiết bị điện tử và ánh sáng nhân tạo. Ngoài ra corridor đôi khi cũng có thể bị nhầm với office nếu ảnh chỉ chứa một phần không gian hành lang hoặc khu vực làm việc.
4. Dữ liệu có mất cân bằng không?
- Có. Lớp corridor có 346 ảnh trong khi các lớp còn lại chỉ khoảng 100–114 ảnh. Điều này có thể khiến mô hình học tốt hơn trên lớp corridor so với các lớp khác.
5. Augmentation có giúp cải thiện không?
- Có. Data augmentation giúp tăng tính đa dạng của dữ liệu huấn luyện, giảm hiện tượng overfitting và cải thiện khả năng tổng quát hóa của mô hình. Đây là yếu tố quan trọng khi làm việc với tập dữ liệu có quy mô tương đối nhỏ.

## 7. Liên hệ với lý thuyết ViT

Trả lời ngắn gọn:

1. Patch embedding trong ViT tương tự bước nào trong NLP?
- Patch embedding tương tự quá trình biến đổi từ (word token) thành vector embedding trong NLP. Mỗi patch ảnh được xem như một token đầu vào cho Transformer.
2. Vì sao ViT cần positional embedding?
- Transformer không tự hiểu được vị trí không gian của các patch ảnh. Positional embedding được thêm vào để cung cấp thông tin về vị trí của từng patch, giúp mô hình học được cấu trúc không gian của ảnh.
3. Vì sao `head_only` train nhanh hơn `finetune`?
- Trong chế độ head_only, toàn bộ backbone ViT được đóng băng và chỉ huấn luyện lớp phân loại cuối. Do số lượng tham số cần cập nhật rất nhỏ nên thời gian huấn luyện nhanh hơn đáng kể.
4. Khi nào nên fine-tune toàn bộ backbone?
- Nên fine-tune toàn bộ backbone khi có tập dữ liệu đủ lớn hoặc khi dữ liệu mới khác biệt đáng kể so với dữ liệu mà mô hình pretrained đã được huấn luyện trước đó.

## 8. W&B evidence

- Link run: https://wandb.ai/datn89367-i-h-c-i-nam/csc4005-lab6-mit-indoor-vit?nw=nwuserdatn89367
- Screenshot dashboard:![Biểu đồ Loss và Accuracy](outputs/debug_smoke/dashboard.jpg)
- Các hyperparameter chính:
    Model: ViT-B/16
Train mode: head_only
Epochs: 10
Batch size: 16
Learning rate: 0.001
Weight decay: 0.0001
Image size: 224
Augmentation: Enabled

- Các metric được log:
Train Loss
Validation Accuracy
Validation Macro-F1
Learning Rate
Epoch Time
Test Accuracy
Test Macro-F1

## 9. Kết luận

Viết 5–8 dòng:

- Mô hình đạt kết quả như thế nào?
    + Trong bài thực hành này, mô hình Vision Transformer ViT-B/16 được áp dụng cho bài toán phân loại cảnh trong nhà thuộc bối cảnh Smart Campus. Kết quả đạt được khá tốt với Validation Macro-F1 đạt 93.21% và Test Accuracy đạt 91.38%. Điều này cho thấy khả năng tận dụng hiệu quả tri thức từ mô hình pretrained trên ImageNet thông qua phương pháp transfer learning.
- ViT có ưu/nhược điểm gì trên dataset nhỏ?
    + Ưu điểm của ViT là khả năng học các mối quan hệ toàn cục trong ảnh và tận dụng tốt các trọng số pretrained. Tuy nhiên, ViT thường yêu cầu lượng dữ liệu lớn và tài nguyên tính toán cao nếu huấn luyện từ đầu. Trên tập dữ liệu nhỏ như trong bài lab, việc sử dụng chế độ head-only giúp giảm thời gian huấn luyện và hạn chế overfitting.
- Nếu cải thiện, bạn sẽ cải thiện dữ liệu, mô hình hay quy trình huấn luyện?
    + Trong tương lai, có thể cải thiện kết quả bằng cách mở rộng tập dữ liệu, cân bằng số lượng mẫu giữa các lớp, thử nghiệm fine-tune toàn bộ backbone hoặc áp dụng các kỹ thuật augmentation nâng cao để tăng khả năng tổng quát hóa của mô hình.
