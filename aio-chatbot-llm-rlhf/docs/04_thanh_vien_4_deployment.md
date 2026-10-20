# Thành viên 4 — Triển khai và giao diện chatbot

## Vai trò và phạm vi

Bạn biến policy đã huấn luyện thành sản phẩm nhóm có thể dùng thử: server vLLM cung cấp API sinh phản hồi và Gradio cung cấp giao diện chat. Cần phân biệt lỗi kết nối/phục vụ với lỗi nội dung do model.

Phụ trách `src/deployment/serve_vllm.sh` và `src/deployment/app.py`. Dựa trên PDF II.4, trang 13–15.

## Đầu vào và quan hệ với nhóm

Nhận policy PPO đã export, tokenizer, revision và cấu hình từ thành viên 3. Nhận model SFT cùng bộ hội thoại mẫu từ thành viên 1 để so sánh baseline. Không dùng reward model của thành viên 2 làm model sinh văn bản.

Trong khi chờ PPO, có thể dựng UI và kiểm tra API với model thử; ghi rõ model thử khác artifact chính thức. Demo cuối phải chạy đúng policy đã được bàn giao.

## Bước 1. Kiểm tra model và chuẩn bị môi trường

Yêu cầu thành viên 3 cung cấp artifact inference đầy đủ, không chỉ adapter hoặc checkpoint DeepSpeed. Kiểm tra model config, tokenizer, chat template, context limit và phiên bản export. Thử model trước khi quy lỗi cho UI.

PDF dùng môi trường serving Python 3.12 cùng vLLM, Gradio và client `openai`. Chọn các phiên bản tương thích GPU và tách môi trường này khỏi training nếu dependency xung đột. Pipeline trong PDF hướng tới GPU NVIDIA/Linux; UI có thể ở máy riêng kết nối tới server.

**Cần xác nhận:** model sẵn có, đủ VRAM, quyền truy cập đúng và nơi chạy server/UI đã rõ ràng.

## Bước 2. Khởi chạy vLLM để phục vụ model

vLLM giữ model trên GPU và nhận request qua API tương thích kiểu OpenAI. Bạn chịu trách nhiệm cấu hình model, cổng, khóa API và bộ nhớ; kiểm tra server sẵn sàng trước khi mở Gradio.

| Cấu hình theo PDF trang 13 | Giá trị tham khảo | Điều cần kiểm tra |
|---|---|---|
| Model | Policy PPO của nhóm | Thay model ví dụ của tác giả bằng artifact thực |
| Port | 8000 | Client kết nối đúng địa chỉ/cổng |
| Quantization | bitsandbytes | Phiên bản và model có hỗ trợ không |
| Prefix caching | Bật | Cấu hình tương thích server |
| Swap space | 16 GiB | Dung lượng CPU theo cấu hình vLLM, không coi là VRAM bổ sung trực tiếp |
| GPU memory utilization | 0.9 | Chừa đủ bộ nhớ cho workload và tiến trình khác |
| Max model length | 2048 | Tổng token input và output không vượt giới hạn |
| Request logging | Tắt theo ví dụ PDF | Kiểm tra cờ của phiên bản, không log secret |

Đây là cấu hình mẫu, không phải tất cả đều bắt buộc. Khởi đầu bằng cấu hình đơn giản đủ chạy, sau đó mới bật tối ưu và đo lại. Nếu lỗi load model, gửi log đã che secret và model revision cho thành viên 3.

## Bước 3. Kiểm tra API độc lập với UI

Xác nhận server liệt kê được model và nhận một yêu cầu chat đơn giản. Thống nhất ba thông tin với app:

| Biến | Nội dung |
|---|---|
| `OPENAI_API_KEY` | Khóa vLLM nội bộ, khác HF token |
| `OPENAI_API_BASE_URL` | Địa chỉ API có hậu tố `/v1` |
| `CHAT_MODEL` | Model ID mà server thực sự phục vụ |

Đây là quy ước kết nối của app trong PDF. App phải đọc và truyền đúng giá trị khi tạo client. Nếu server/UI khác máy, localhost của UI không trỏ tới server; cấu hình địa chỉ hoặc tunnel phù hợp. Không hardcode khóa vào code hay ảnh minh họa.

**Cần xác nhận:** một request nhận nội dung tiếng Việt, model ID đúng và lỗi sai khóa được nhận biết. Chỉ sau bước này mới nối UI để dễ khoanh vùng lỗi.

## Bước 4. Thiết kế Gradio ChatInterface

Theo trang 14, UI có vùng chat, ô nhập, system message và các thanh điều chỉnh. Giải thích rõ cho người dùng các lựa chọn ảnh hưởng câu trả lời:

| Thành phần | Mặc định trong PDF | Nhiệm vụ |
|---|---|---|
| System message | Lời hướng dẫn chatbot thân thiện | Nhóm nên thống nhất yêu cầu trả lời tiếng Việt với SFT |
| Max new tokens | 512; slider 1–2048 | Giới hạn output nhưng phải xét context còn lại |
| Temperature | 0.7; slider 0.1–4.0 | Điều chỉnh mức ngẫu nhiên khi sampling |
| Top-p | 0.95; slider 0.1–1.0 | Giới hạn tập token lấy mẫu theo xác suất |
| Cổng UI | 7860 | Tách biệt cổng API 8000 |

Phạm vi slider trong PDF không bảo đảm mọi request hợp lệ: nếu max context là 2048 thì không thể cho phép output 2048 khi đã có prompt/lịch sử. Cần tính hoặc giới hạn ngân sách token trước khi gửi.

## Bước 5. Dựng request hội thoại và hiển thị streaming

Mỗi request gồm system prompt, lịch sử đúng thứ tự và câu hỏi hiện tại đúng một lần. Chuẩn hóa role/content theo schema client/server; không đưa thêm message lặp sau mỗi lượt.

Streaming trả về từng phần phản hồi. App tích lũy các phần nội dung rồi cập nhật cùng một lượt assistant thay vì thêm nhiều bong bóng chat. Bỏ qua chunk không có nội dung; kiểm tra định dạng history/yield theo phiên bản Gradio đang dùng vì API có thể khác ví dụ PDF.

Thêm reset để bắt đầu cuộc hội thoại mới. Với hội thoại dài, có chính sách bỏ lượt cũ hoặc báo người dùng bắt đầu lại; không cắt tùy tiện làm đứt ngữ cảnh mà không giải thích.

## Bước 6. Xử lý lỗi và kiểm thử tích hợp

Kiểm thử lần lượt: một câu hỏi tiếng Việt → câu hỏi tiếp nối dựa trên lịch sử → reset → request quá dài → sai khóa hoặc mất kết nối. UI cần thông báo dễ hiểu và không lộ secret/stack trace.

Ghi model revision, điều kiện sinh, lỗi tái hiện được và thời gian phản hồi nếu có đo. So sánh SFT/PPO trên cùng bộ test với cùng điều kiện, phối hợp thành viên 3 khi PPO suy giảm. Nếu cả hai model đều lỗi tương tự, đưa ví dụ cho thành viên 1. RM không trực tiếp tham gia serving nên không mặc định mọi lỗi UI liên quan RM.

## Bước 7. Bàn giao demo và cách vận hành

Viết hướng dẫn ngắn về môi trường, cách chọn model, cấu hình kết nối, thứ tự khởi động API trước UI và cách tắt dịch vụ. Lưu ảnh giao diện vào `assets/` sau khi che thông tin nhạy cảm. Nhờ thành viên khác làm theo hướng dẫn để kiểm tra có thiếu bước nào.

- [ ] Policy PPO đúng revision nạp và phục vụ được.
- [ ] Chat tiếng Việt, streaming, nhiều lượt và reset hoạt động.
- [ ] Giới hạn context và các lỗi API được xử lý có thông báo.
- [ ] Có config không chứa secret, package versions, ảnh và kết quả kiểm thử.
- [ ] Thành viên khác khởi động lại demo được; ghi rõ hạn chế hiệu năng/chất lượng còn tồn tại.

## Cách sử dụng hướng dẫn

Các thông số ghi “theo PDF” là cấu hình tham khảo trong tài liệu gốc, chưa phải cấu hình đã kiểm thử của repo. Điều chỉnh theo GPU và phiên bản thư viện, ghi rõ thay đổi khi bàn giao. Phần chia tập đánh giá, chạy thử, kiểm tra nạp lại và nghiệm thu là bổ sung để nhóm thực hiện dự án có kiểm soát. Tài liệu mô tả nhiệm vụ, không cung cấp code triển khai đầy đủ.
