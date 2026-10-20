# Thành viên 1 — Chuẩn bị dữ liệu và huấn luyện SFT

## Vai trò và phạm vi

Bạn tạo nền tảng ngôn ngữ cho các giai đoạn còn lại: model cần biết trả lời tiếng Việt, làm theo yêu cầu và sử dụng lịch sử hội thoại. SFT học từ các câu trả lời mẫu; chưa sử dụng reward model hay PPO.

Phụ trách `src/sft/dataset.py` (xử lý dữ liệu) và `src/sft/train_sft.py` (huấn luyện, lưu và kiểm thử). Dựa trên PDF mục II.1.1–II.1.5, trang 4–8.

## Đầu vào và quan hệ với nhóm

| Đầu vào | Vai trò |
|---|---|
| `unsloth/Llama-3.2-1B-Instruct-bnb-4bit` | Model nền trong PDF, nạp dạng 4-bit để giảm bộ nhớ |
| `5CD-AI/Vietnamese-Multi-turn-Chat-Alpaca` | Hội thoại nhiều lượt để model học cách phản hồi theo ngữ cảnh |
| Môi trường GPU và quyền Hugging Face | Huấn luyện, tải model/dữ liệu và lưu artifact khi cần |

Thống nhất với thành viên 2 và 3 ngay từ đầu về model nền, tokenizer, chat template và cách ghi revision. Thành viên 2 cần SFT để khởi tạo RM; thành viên 3 cần cùng bản SFT cho actor/reference. Thành viên 4 có thể nhận SFT làm baseline trước khi PPO hoàn thành.

## Bước 1. Chuẩn bị môi trường và model

Theo PDF II.1.1–II.1.2, chuẩn bị Unsloth, datasets, PyTorch, Transformers và TRL. Tài liệu dùng Unsloth 2025.4.7 và datasets 3.5.1; ghi đủ các phiên bản thực tế thay vì chỉ hai package này. Không ghi token Hugging Face vào notebook hoặc code.

Nạp model cùng tokenizer, giới hạn chuỗi 2048 và 4-bit quantization theo PDF. Tokenizer chuyển văn bản thành token; chat template quy định cách đánh dấu system/user/assistant mà model nhận biết. Cả hai phải được lưu cùng model.

**Cần xác nhận:** GPU nhận được, model nạp được, dtype phù hợp và đủ bộ nhớ cho một mẫu thử. BF16 chỉ dùng khi phần cứng hỗ trợ.

## Bước 2. Hiểu và làm sạch dữ liệu hội thoại

Theo II.1.3, dữ liệu nằm trong trường `conversations`; mỗi lượt có `from` và `value`. Xem nhiều mẫu, thống kê số lượt, độ dài, mẫu rỗng và vai trò xuất hiện thực tế. Không giả định mọi hội thoại có cùng số lượt.

Chuyển lượt human thành user, lượt phản hồi model thành assistant; thêm system prompt tiếng Việt như tài liệu. Kiểm tra role lạ thay vì tự quy mọi giá trị còn lại thành assistant. Giữ nguyên thứ tự hội thoại, vì câu hỏi sau có thể phụ thuộc câu trả lời trước.

Tách train/validation/test theo hội thoại trước khi xử lý; tránh để cùng ngữ cảnh xuất hiện ở nhiều tập. Lưu dữ liệu gốc trong `data/raw/`, bản xử lý trong `data/processed/`, kèm revision, số mẫu và lý do loại mẫu.

**Cần xác nhận:** một hội thoại sau chuyển đổi vẫn giữ đúng nội dung, tiếng Việt có dấu và các vai trò.

## Bước 3. Format và tokenization

Áp dụng chat template của chính tokenizer. Khi SFT, hội thoại đã chứa đáp án assistant nên không thêm lời mở đầu cho một lượt sinh mới; PDF đặt `add_generation_prompt=False`. Sau đó tokenize, giới hạn độ dài và tạo attention mask.

PDF dùng truncation và padding tới 2048. Kiểm tra tỷ lệ hội thoại bị cắt, đặc biệt phần đáp án cuối; không chỉ nhìn số lượng mẫu sau map. Khi text đã có special tokens từ template, tránh thêm chúng lần nữa. Đảm bảo phần padding không bị tính như đáp án cần học; ghi rõ loss tính trên toàn hội thoại hay chỉ lượt assistant khi triển khai trainer.

**Đầu ra:** dữ liệu có thể đưa vào trainer và vài mẫu decode để nhóm kiểm tra bằng mắt.

## Bước 4. Thiết lập QLoRA và lịch huấn luyện

4-bit giảm bộ nhớ của model nền; LoRA thêm các tham số nhỏ để học thay vì cập nhật toàn bộ trọng số nền. Gradient checkpointing giảm bộ nhớ kích hoạt bằng cách tính lại một phần phép tính.

| Cấu hình theo PDF trang 5–6 | Giá trị tham khảo | Ý nghĩa nhiệm vụ |
|---|---|---|
| LoRA rank / alpha / dropout | 16 / 16 / 0 | Quy mô adapter, mức tác động cập nhật và dropout |
| Target modules | q/k/v/o_proj, up/down/gate_proj | Các phép chiếu attention và MLP được gắn adapter |
| rsLoRA / seed | Bật / 42 | Cách scale adapter và mốc tái lập |
| Gradient checkpointing | Chế độ Unsloth | Giảm bộ nhớ khi huấn luyện |
| Batch mỗi thiết bị / tích lũy | 32 / 2 | Số mẫu mỗi bước nhỏ và số lần cộng gradient |
| Learning rate / optimizer | 1e-4 / paged_adamw_8bit | Cách cập nhật tham số |
| Scheduler / warmup ratio | Cosine / 0.05 | Điều chỉnh learning rate theo tiến trình |
| Max steps / logging | 400 / mỗi 20 bước | Thời lượng và tần suất theo dõi |
| Lưu checkpoint | Mỗi 50 bước, giữ 4 | Khả năng phục hồi và giới hạn dung lượng |

Batch 32 không phải yêu cầu cứng cho mọi GPU. Nếu giảm micro batch để tránh OOM, cân nhắc accumulation để giữ effective batch; effective batch còn phụ thuộc số GPU data parallel. Ghi lại cấu hình thực tế và lý do thay đổi.

## Bước 5. Huấn luyện và chọn checkpoint

Chạy vài bước với dữ liệu nhỏ trước: kiểm tra loss hữu hạn, gradient cập nhật, checkpoint lưu được và model nạp lại được. Sau đó mới chạy đủ lịch huấn luyện, theo dõi loss, bộ nhớ và các mẫu phản hồi.

Dùng validation để phát hiện học quá mức hoặc suy giảm chất lượng. PDF nạp checkpoint-400; không mặc định checkpoint cuối là tốt nhất nếu chưa có kết quả đánh giá. Lưu config và log để người khác biết model được tạo ra như thế nào.

## Bước 6. Kiểm thử hội thoại, merge và bàn giao

Theo II.1.5, thử một yêu cầu viết lại câu, sau đó hỏi tiếp dựa trên ngữ cảnh trước. Inference cần generation prompt cho lượt assistant mới. Chỉ hiển thị token mới sinh, không nhầm toàn bộ prompt với câu trả lời.

PDF dùng tối đa 128 token, greedy (`do_sample=False`), repetition penalty 1.3. Giữ cùng điều kiện khi so sánh checkpoint; temperature không điều khiển sampling khi sampling bị tắt. Bổ sung câu hỏi tiếng Việt khác để tránh đánh giá chỉ từ một ví dụ.

Giữ adapter riêng; merge/export thành model causal LM đầy đủ và lưu tokenizer. Nếu upload Hugging Face, dùng namespace của nhóm và ghi revision; trước upload phải kiểm tra bản local nạp lại được. Thành viên 2/3 không nên phải đoán model nền khi chỉ nhận một adapter.

## Bàn giao và tiêu chí hoàn thành

| Người nhận | Nhận gì | Kiểm tra cùng người nhận |
|---|---|---|
| Thành viên 2 | Merged SFT, tokenizer, base revision | RM loader nạp được để thêm reward head |
| Thành viên 3 | Cùng merged SFT và tokenizer | Khởi tạo actor/reference đúng bản nền |
| Thành viên 4 | SFT baseline, cấu hình sinh, hội thoại mẫu | So sánh với PPO trên cùng bộ test |

- [ ] Lưu model ID/path, revision, adapter, tokenizer và chat template.
- [ ] Có dữ liệu/split, config, phiên bản môi trường, log và kết quả validation.
- [ ] Model nạp lại được, trả lời tiếng Việt và xử lý câu hỏi tiếp nối.
- [ ] Ghi rõ lỗi còn gặp, không chỉ cung cấp ví dụ thành công.

## Cách sử dụng hướng dẫn

Các thông số ghi “theo PDF” là cấu hình tham khảo trong tài liệu gốc, chưa phải cấu hình đã kiểm thử của repo. Điều chỉnh theo GPU và phiên bản thư viện, ghi rõ thay đổi khi bàn giao. Phần chia tập đánh giá, chạy thử, kiểm tra nạp lại và nghiệm thu là bổ sung để nhóm thực hiện dự án có kiểm soát. Tài liệu mô tả nhiệm vụ, không cung cấp code triển khai đầy đủ.
