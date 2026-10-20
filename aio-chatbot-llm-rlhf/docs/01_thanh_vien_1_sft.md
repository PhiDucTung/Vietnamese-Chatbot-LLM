# Thành viên 1 — Chuẩn bị dữ liệu và huấn luyện SFT

## Mục tiêu

Tinh chỉnh Llama-3.2-1B-Instruct bằng QLoRA để chatbot trả lời tiếng Việt và xử lý hội thoại nhiều lượt.

Phụ trách: `src/sft/dataset.py` và `src/sft/train_sft.py`.

## Đầu vào cần chuẩn bị

- Bộ dữ liệu `5CD-AI/Vietnamese-Multi-turn-Chat-Alpaca`.
- Model nền `unsloth/Llama-3.2-1B-Instruct-bnb-4bit` theo PDF.
- Môi trường huấn luyện GPU phù hợp, chẳng hạn Google Colab, cùng quyền truy cập Hugging Face khi cần.

## Các bước thực hiện

1. **Chuẩn bị môi trường:** kiểm tra GPU và thư viện phục vụ SFT, bảo đảm các phiên bản tương thích trước khi huấn luyện.
2. **Khảo sát dữ liệu:** xem cấu trúc hội thoại, số lượt trao đổi, độ dài và các mẫu lỗi hoặc thiếu nội dung.
3. **Làm sạch và chia tập:** loại mẫu không hợp lệ, tách tập huấn luyện và đánh giá, tránh trùng hội thoại giữa các tập. Lưu dữ liệu gốc trong `data/raw/`.
4. **Chuẩn hóa hội thoại:** chuyển các lượt thành vai trò system, user, assistant; dùng lời hướng dẫn chatbot trả lời bằng tiếng Việt.
5. **Xử lý dữ liệu:** áp dụng chat template của tokenizer, tokenization và giới hạn độ dài phù hợp. Lưu bản xử lý trong `data/processed/`.
6. **Thiết lập QLoRA:** chọn cấu hình LoRA, batch size, learning rate và số bước dựa trên PDF và bộ nhớ GPU thực tế.
7. **Huấn luyện thử rồi chạy đầy đủ:** bắt đầu bằng một lượng dữ liệu nhỏ để kiểm tra luồng xử lý, sau đó theo dõi loss và lưu checkpoint trong quá trình huấn luyện.
8. **Đánh giá model:** thử câu hỏi tiếng Việt, yêu cầu viết lại và hội thoại nhiều lượt; ghi lại cả phản hồi tốt lẫn lỗi. Chọn checkpoint theo đánh giá, không chỉ theo số bước cuối.
9. **Lưu và bàn giao:** lưu adapter LoRA, model đã merge và tokenizer; kiểm tra model đã lưu có thể nạp lại và sinh phản hồi.

## Kết quả cần bàn giao

- Model SFT đã merge và tokenizer để thành viên 2, 3 sử dụng; giữ adapter riêng để tiếp tục tinh chỉnh khi cần.
- Thông tin model nền, dữ liệu, cách chia tập và cấu hình huấn luyện.
- Kết quả đánh giá cùng một số hội thoại minh họa và hạn chế còn gặp.

## Tiêu chí hoàn thành

- Model nạp lại được và trả lời bằng tiếng Việt.
- Model sử dụng được ngữ cảnh của các lượt hội thoại trước.
- Thành viên 2 và 3 xác nhận dùng được model SFT bàn giao.

Tham khảo: PDF mục II.1, trang 4–8. [Về phân công](README.md).
