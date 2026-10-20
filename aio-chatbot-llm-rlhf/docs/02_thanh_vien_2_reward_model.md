# Thành viên 2 — Huấn luyện mô hình điểm thưởng

## Vai trò và phạm vi

Bạn xây mô hình đánh giá phản hồi để cung cấp tín hiệu thưởng cho PPO. RM nhận ngữ cảnh cùng một câu trả lời và cho điểm scalar; nó không phải chatbot dùng để sinh câu trả lời cuối.

Phụ trách `src/reward_model/train_rm.py` và `src/reward_model/merge_lora.py`. Dựa trên PDF II.2.1–II.2.4, trang 9–10; phần giải thích vai trò ở trang 8.

## Đầu vào và quan hệ với nhóm

Nhận **model SFT đã merge, tokenizer và revision từ thành viên 1**. Dùng dữ liệu `thuanan/Vi-Alpaca-Preference` để học sự ưu tiên giữa chosen/rejected. Kết quả RM sẽ chuyển cho thành viên 3, nên thống nhất sớm cách format input, padding và lấy score.

Trong lúc chờ SFT, có thể khảo sát preference và chuẩn bị môi trường. Model thử dùng lúc phát triển phải ghi rõ; không nghiệm thu RM cuối trên một base khác mà không thông báo.

## Bước 1. Chuẩn bị OpenRLHF và tiếp nhận SFT

Theo II.2.1, OpenRLHF cung cấp pipeline RM; DeepSpeed hỗ trợ huấn luyện và quản lý bộ nhớ. Chọn phiên bản/commit phù hợp cùng GPU, ghi các dependency và xác nhận model SFT nạp được trong môi trường này.

Không mặc định CLI của bản mới giống PDF. Dùng môi trường riêng với SFT nếu dependency xung đột. Kiểm tra quyền tải model và nơi lưu checkpoint trước run.

**Cần xác nhận:** có đúng merged SFT và tokenizer, không chỉ adapter; revision khớp với artifact thành viên 1 công bố.

## Bước 2. Hiểu các cặp chosen/rejected

Chosen là phản hồi được ưu tiên; rejected là phản hồi kém được ưu tiên hơn cho cùng yêu cầu. Mục tiêu là học thứ tự `score(chosen) > score(rejected)`, không học một thang điểm tuyệt đối có ý nghĩa cố định giữa mọi run.

Kiểm tra schema thực tế: các trường có thể là text hoặc danh sách message tùy dữ liệu/revision. Với cả hai nhánh, giữ đúng prompt và lịch sử chung; tránh chấm một đáp án không kèm ngữ cảnh hoặc ghép prompt hai lần. Loại cặp rỗng, đảo nhãn, trùng nội dung hoặc không cùng context.

Chia train/validation/test theo context, không để cùng câu hỏi trùng sang tập đánh giá. Ghi số cặp hợp lệ và ví dụ minh họa cách diễn giải nhãn.

## Bước 3. Chuẩn hóa input và thiết lập reward head

Áp dụng chat template tương thích với model SFT; kiểm tra giới hạn độ dài không cắt hết phần trả lời cần so sánh. Chosen/rejected phải được xử lý theo cùng quy tắc.

Khởi tạo backbone từ SFT và thêm reward head trả về một điểm. PDF dùng prefix `score` để thuận tiện lưu/nạp như model phân loại chuỗi. Đây là thành phần phải được huấn luyện và giữ lại khi lưu/merge; thiếu head hoặc head bị khởi tạo mới sẽ làm mất ý nghĩa điểm thưởng đã học.

**Cần xác nhận:** một batch nhỏ đi qua cả hai nhánh, trả về điểm hữu hạn với đúng số lượng mẫu.

## Bước 4. Thiết lập huấn luyện theo tài liệu

| Cấu hình PDF II.2.2 | Giá trị tham khảo | Mục đích |
|---|---|---|
| Backbone | Merged SFT của thành viên 1 | Khởi đầu từ model đã học hội thoại |
| Train batch / micro batch | 96 / 8 | Batch tổng và batch mỗi bước nhỏ |
| Epoch / max length | 1 / 2048 | Thời lượng và độ dài input |
| Learning rate | 5e-6 | Mức cập nhật RM |
| DeepSpeed ZeRO | Stage 2 | Phân chia trạng thái huấn luyện để giảm bộ nhớ |
| LoRA rank / alpha | 16 / 32 | Quy mô và mức scale adapter |
| Precision | BF16 | Dùng khi GPU hỗ trợ |
| Tối ưu bộ nhớ | FlashAttention, packing, gradient checkpointing, Adam offload | Giảm chi phí bộ nhớ/tăng hiệu quả theo stack hỗ trợ |
| Logging / save / eval steps | 1 / -1 / -1 | Theo ví dụ PDF; kiểm tra ngữ nghĩa theo phiên bản |

Không coi batch tổng là batch mỗi GPU. Nếu điều chỉnh micro batch, kiểm tra accumulation và số worker để phù hợp batch tổng. Packing phải giữ ranh giới mẫu và logic lấy score đúng. Với run mới không bật resume chỉ vì PDF có cờ load checkpoint; chỉ resume khi checkpoint thực sự có và tương thích.

## Bước 5. Huấn luyện và kiểm tra khả năng xếp hạng

Chạy thử ít cặp để kiểm tra forward, loss, save/reload trước full epoch. Theo dõi loss và phân bố điểm; lỗi NaN, mọi mẫu cùng điểm hoặc chênh điểm cực lớn cần điều tra dữ liệu/dtype/head.

Trên tập validation chưa dùng huấn luyện, tính tỷ lệ chosen có điểm cao hơn rejected. Báo số mẫu, tỷ lệ bằng điểm và margin chosen−rejected; xem trực tiếp các cặp chấm sai. Quy ước ties không được tính là thắng và phải ghi rõ. Loss giảm không đủ chứng minh RM hữu ích cho PPO.

## Bước 6. Merge adapter và bảo toàn head

Theo II.2.3, hợp nhất adapter vào đúng model SFT nguồn bằng tiện ích merge dành cho RM; PDF dùng lora_combiner và chế độ RM. Kiểm tra công cụ tương ứng với phiên bản đã chọn, không sao chép merge causal LM rồi bỏ reward head.

Lưu bản merged riêng với adapter. Nạp bản merged trong tiến trình mới, kiểm tra cảnh báo missing keys/head mới và chấm lại cùng vài cặp đã dùng trước merge. Score có thể sai khác nhỏ theo dtype, nhưng không được thay đổi thứ tự hàng loạt mà chưa giải thích.

## Bước 7. Bàn giao cho PPO

Theo II.2.4, có thể lưu lên Hugging Face sau khi kiểm tra local. Tên `Llama-3.2-1B-RM-DPO` trong PDF là tên artifact; nhiệm vụ này không triển khai DPO policy training.

Thành viên 3 cần RM đầy đủ để chấm phản hồi actor sinh ra. Hai bên chạy cùng một nhóm prompt/response, xác nhận cách lấy score và tokenizer/padding nhất quán trước khi PPO chạy dài.

- [ ] Bàn giao merged RM, tokenizer, adapter lưu riêng và model SFT nguồn/revision.
- [ ] Có preprocessing, quy ước lấy score, cấu hình và phiên bản môi trường.
- [ ] Có pairwise accuracy, ties, ví dụ đúng/sai trên validation.
- [ ] Loader PPO nạp được, không tạo lại head, trả về scalar hữu hạn.
- [ ] Ghi rõ hạn chế; phối hợp thành viên 1 nếu lỗi từ base/tokenizer và thành viên 3 nếu reward dễ bị khai thác.

## Cách sử dụng hướng dẫn

Các thông số ghi “theo PDF” là cấu hình tham khảo trong tài liệu gốc, chưa phải cấu hình đã kiểm thử của repo. Điều chỉnh theo GPU và phiên bản thư viện, ghi rõ thay đổi khi bàn giao. Phần chia tập đánh giá, chạy thử, kiểm tra nạp lại và nghiệm thu là bổ sung để nhóm thực hiện dự án có kiểm soát. Tài liệu mô tả nhiệm vụ, không cung cấp code triển khai đầy đủ.
