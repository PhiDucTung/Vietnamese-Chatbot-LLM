# Thành viên 2 — Huấn luyện mô hình điểm thưởng

## Mục tiêu

Tạo reward model có khả năng chấm điểm phản hồi, ưu tiên câu trả lời được chọn so với câu trả lời bị từ chối, để phục vụ PPO.

Phụ trách: `src/reward_model/train_rm.py` và `src/reward_model/merge_lora.py`.

## Đầu vào cần chuẩn bị

- Model SFT đã merge và tokenizer từ thành viên 1.
- Bộ dữ liệu `thuanan/Vi-Alpaca-Preference` gồm các cặp chosen/rejected.
- Môi trường GPU có OpenRLHF và DeepSpeed tương thích.

## Các bước thực hiện

1. **Tiếp nhận model SFT:** kiểm tra phiên bản model nền, tokenizer và khả năng nạp model từ artifact bàn giao.
2. **Chuẩn bị môi trường:** chọn phiên bản OpenRLHF, DeepSpeed và các thư viện liên quan phù hợp GPU. Đối chiếu hướng dẫn của đúng phiên bản vì công cụ có thể khác PDF.
3. **Khảo sát dữ liệu preference:** xác định cách biểu diễn prompt, chosen và rejected; bảo đảm hai phản hồi trong một cặp thuộc cùng ngữ cảnh.
4. **Làm sạch và chia tập:** loại cặp rỗng, lỗi hoặc hai phản hồi giống nhau; tách tập đánh giá không trùng ngữ cảnh với tập huấn luyện.
5. **Chuẩn hóa dữ liệu:** dùng chat template phù hợp model SFT và giới hạn độ dài sao cho vẫn giữ được phần phản hồi cần chấm điểm.
6. **Thiết lập reward model:** khởi tạo từ model SFT, bổ sung đầu chấm điểm (reward head), chọn cấu hình LoRA và DeepSpeed theo tài nguyên thực tế.
7. **Huấn luyện:** chạy thử với ít mẫu trước, sau đó huấn luyện đầy đủ; theo dõi loss, điểm số và việc lưu checkpoint.
8. **Đánh giá khả năng xếp hạng:** đo tỷ lệ cặp mà chosen được chấm cao hơn rejected trên tập đánh giá; xem các trường hợp chấm sai hoặc bằng điểm.
9. **Merge và kiểm tra:** hợp nhất adapter vào model, bảo đảm giữ reward head; nạp lại để kiểm tra điểm số trước/sau merge nhất quán.
10. **Bàn giao:** cung cấp reward model hoàn chỉnh và cách sử dụng cho thành viên 3.

## Kết quả cần bàn giao

- Reward model đã merge, tokenizer và adapter nếu có.
- Phiên bản model SFT nguồn, cấu hình huấn luyện và cách chuẩn hóa/chấm điểm dữ liệu.
- Kết quả đánh giá chosen/rejected, các ví dụ sai và hạn chế.

## Tiêu chí hoàn thành

- Model nạp lại được mà không làm mất hoặc khởi tạo lại reward head.
- Có kết quả đánh giá trên dữ liệu chưa dùng huấn luyện.
- Thành viên 3 xác nhận model trả về điểm thưởng dùng được cho PPO.

Tên checkpoint có `RM-DPO` trong PDF là tên artifact; phần việc này là huấn luyện reward model, không phải huấn luyện policy bằng DPO.

