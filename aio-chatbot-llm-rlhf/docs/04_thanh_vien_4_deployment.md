# Thành viên 4 — Triển khai và giao diện chatbot

## Mục tiêu

Đưa policy sau PPO lên dịch vụ sinh phản hồi bằng vLLM và xây giao diện Gradio để người dùng trò chuyện tiếng Việt.

Phụ trách: `src/deployment/serve_vllm.sh` và `src/deployment/app.py`.

## Đầu vào cần chuẩn bị

- Policy PPO hoàn chỉnh, tokenizer và thông tin phiên bản từ thành viên 3.
- Môi trường phục vụ vLLM có GPU phù hợp; giao diện có thể chạy trên máy riêng và kết nối tới server.
- Bộ câu hỏi kiểm thử do cả nhóm thống nhất.

## Các bước thực hiện

1. **Tiếp nhận model:** kiểm tra artifact là model sinh văn bản đầy đủ, không chỉ adapter hoặc checkpoint huấn luyện chưa xuất; xác nhận tokenizer và chat template đi kèm.
2. **Chuẩn bị môi trường serving:** chọn phiên bản vLLM, Gradio và thư viện client tương thích; xác định máy chạy model và máy chạy giao diện.
3. **Thiết lập dịch vụ vLLM:** chọn model, cổng, giới hạn ngữ cảnh và cấu hình bộ nhớ theo GPU; quản lý khóa API qua biến môi trường, không ghi vào mã nguồn.
4. **Kiểm tra API trước:** xác nhận server đã sẵn sàng, model được nhận diện và một yêu cầu chat đơn giản trả về tiếng Việt.
5. **Xây dựng giao diện Gradio:** tạo vùng hội thoại, ô nhập câu hỏi và các tùy chọn cần thiết như system prompt, giới hạn phản hồi, temperature và top-p.
6. **Kết nối UI với API:** gửi đúng lịch sử hội thoại và câu hỏi hiện tại, hiển thị phản hồi dần khi streaming; tránh gửi trùng lượt người dùng.
7. **Quản lý ngữ cảnh:** bảo đảm tổng lịch sử, câu hỏi và phản hồi nằm trong giới hạn model; hỗ trợ bắt đầu hội thoại mới.
8. **Xử lý lỗi sử dụng:** hiển thị thông báo dễ hiểu khi mất kết nối, sai khóa, model chưa sẵn sàng hoặc hội thoại quá dài.
9. **Kiểm thử cùng nhóm:** thử hỏi đáp tiếng Việt, hội thoại nhiều lượt, reset lịch sử và lỗi kết nối; kiểm tra chất lượng cùng tốc độ phản hồi trên môi trường thực tế.
10. **Hoàn thiện bàn giao:** ghi các bước khởi động/tắt dịch vụ và giao diện, cấu hình cần thiết, kết quả kiểm thử và giới hạn còn tồn tại; bổ sung ảnh minh họa vào `assets/` khi có.

## Kết quả cần bàn giao

- Dịch vụ model và giao diện chatbot hoạt động cùng nhau.
- Hướng dẫn vận hành ngắn gọn, thông tin môi trường/model và cấu hình kết nối không chứa secret.
- Ảnh giao diện, các hội thoại kiểm thử và báo cáo lỗi hoặc hạn chế.

## Tiêu chí hoàn thành

- Người dùng nhận được phản hồi tiếng Việt từ policy PPO qua giao diện.
- Hội thoại giữ ngữ cảnh, hiển thị streaming và reset hoạt động đúng.
- Người khác trong nhóm có thể làm theo hướng dẫn để khởi động hệ thống.


