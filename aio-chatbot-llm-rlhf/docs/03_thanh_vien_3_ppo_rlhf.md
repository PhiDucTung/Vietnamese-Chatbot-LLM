# Thành viên 3 — Căn chỉnh chatbot bằng PPO RLHF

## Vai trò và phạm vi

Bạn nối SFT và reward model thành vòng học tăng cường: sinh câu trả lời, chấm điểm, ước lượng advantage và cập nhật policy. Mục tiêu là cải thiện mức hữu ích của phản hồi mà không làm policy lệch quá xa model SFT.

Phụ trách `src/rlhf/train_ppo.sh` và `src/rlhf/ray_cluster.py`. Dựa trên PDF II.3, trang 11–13.

## Đầu vào và quan hệ với nhóm

| Nguồn | Đầu vào cần nhận | Dùng cho |
|---|---|---|
| Thành viên 1 | Merged SFT, tokenizer, revision | Actor ban đầu và reference cố định |
| Thành viên 2 | Merged RM, tokenizer và quy ước score | Chấm chất lượng phản hồi |
| Dataset | `thuanan/Prompt-Vi-Alpaca-Preference-2k` | Ngữ cảnh để sinh rollout |

Phải kiểm tra cả SFT lẫn RM trước khi chạy PPO. Đầu ra của bạn là **policy sinh văn bản** cho thành viên 4, không phải reward model hay chỉ log reward.

## Bước 1. Hiểu vai trò trong PPO

| Thành phần | Công việc | Có cập nhật khi PPO? |
|---|---|---|
| Actor/policy | Sinh phản hồi và học để tăng mục tiêu PPO | Có |
| Reference | Mốc SFT để tính độ lệch KL | Không |
| Reward model | Chấm prompt cùng phản hồi | Không |
| Critic | Ước lượng value hỗ trợ advantage | Có |
| vLLM | Tăng hiệu quả sinh rollout theo cấu hình OpenRLHF | Nhận trọng số policy theo cơ chế framework |

KL là tín hiệu kiểm soát độ lệch so với reference. Reward cao mà KL/độ dài tăng bất thường hoặc phản hồi bị lặp có thể cho thấy policy đang khai thác RM; không đánh giá chỉ qua một đường reward.

## Bước 2. Chuẩn bị Docker, dependency và lưu trữ

PDF dùng PyTorch container, OpenRLHF, Ray và vLLM. Chuẩn bị Linux/NVIDIA, GPU visibility trong container, phiên bản/commit cùng nơi lưu checkpoint bền vững. Mount nơi lưu dữ liệu/model để không mất kết quả khi container dừng.

Phân biệt thao tác trên host với bên trong container. Không gỡ package trên host theo ví dụ xử lý dependency trong container. Kiểm tra CLI của phiên bản đã khóa thay vì mặc định tên cờ còn giống tài liệu.

**Cần xác nhận:** mọi worker đọc được model, dữ liệu và package cần thiết; token cung cấp bằng secret/môi trường, không nằm trong file script hoặc log.

## Bước 3. Chuẩn bị prompts và giới hạn ngữ cảnh

PDF dùng trường `context_messages` và áp dụng chat template. Kiểm tra trường đó chứa các lượt hội thoại hợp lệ, thường kết thúc ở user để actor sinh lượt assistant. Không ghép sẵn câu trả lời mục tiêu vào prompt.

Tách một bộ prompt đánh giá riêng, tránh trùng context train. Giới hạn PDF là prompt 1024 token và sinh thêm 1024 token; kiểm tra tổng độ dài tương thích với actor, reference và RM, không chỉ actor.

## Bước 4. Phân bổ tài nguyên và khởi tạo Ray

Ray điều phối các tiến trình/actor; DeepSpeed quản lý huấn luyện phân tán, vLLM hỗ trợ sinh. Bạn cần lập bảng GPU thật và tài nguyên dành cho từng vai trò trước khi submit job.

PDF cấu hình mỗi vai trò actor/ref/critic/reward là 1 node × 1 GPU; colocate actor với ref và critic với reward, cùng 1 vLLM engine, tensor parallel 1. Colocate là bố trí dùng chung tài nguyên theo cơ chế framework, không phải bằng chứng toàn pipeline fit một GPU.

Ví dụ Ray của PDF khai báo 1 GPU nhưng không cung cấp phép đo đủ cho toàn layout. Không tăng tài nguyên Ray khai báo vượt phần cứng thật. Kiểm tra placement groups và bộ nhớ đồng thời, xử lý job pending trước khi tăng batch. Với nhiều node cần thống nhất địa chỉ head, mount, môi trường và kết nối worker.

## Bước 5. Thiết lập cấu hình PPO

| Thông số theo PDF trang 12 | Giá trị tham khảo | Ý nghĩa |
|---|---|---|
| Model khởi tạo | SFT thành viên 1 | Actor/reference bắt đầu cùng mốc |
| Reward model | RM thành viên 2 | Tín hiệu thưởng |
| Micro train / train batch | 2 / 32 | Khối lượng mỗi bước nhỏ và batch train tổng |
| Micro rollout / rollout batch | 4 / 1024 | Khối lượng sinh nhỏ và batch thu thập trải nghiệm |
| Max epochs | 1 | Lịch cập nhật theo framework |
| Prompt / generation length | 1024 / 1024 | Ngân sách token |
| Actor / critic learning rate | 5e-7 / 9e-6 | Tốc độ học hai model được cập nhật |
| Hệ số KL ban đầu | 0.01 | Mức kiểm soát lệch reference |
| DeepSpeed / precision | ZeRO 3 / BF16 | Phân chia trạng thái/tham số và dtype |
| Tối ưu | Normalize reward, packing, Adam offload, FlashAttention, gradient checkpointing | Hiệu quả và ổn định theo stack hỗ trợ |

Đối chiếu tên cờ và ràng buộc chia hết batch với commit đang dùng. Chạy mới khác với resume; xác định vị trí checkpoint và trạng thái cần phục hồi. Thiết lập WandB nếu dùng để ghi cấu hình, tài nguyên, reward/KL và các mẫu phản hồi.

## Bước 6. Chạy thử vòng PPO rồi huấn luyện đầy đủ

Bắt đầu ít prompt, batch nhỏ và output ngắn để kiểm tra trọn vòng: actor sinh → RM chấm → critic ước lượng → cập nhật policy → lưu checkpoint. Kiểm tra reference/RM giữ cố định và actor thực sự được cập nhật.

Sau đó mới mở rộng workload. Theo dõi reward thô/normalized nếu có, KL, actor/critic loss, độ dài phản hồi, GPU memory và thời gian từng pha. Lưu vài hội thoại định kỳ; khi metric NaN, reward bất thường hoặc chất lượng suy giảm, dừng để tìm nguyên nhân thay vì tiếp tục run dài.

## Bước 7. Đánh giá và phối hợp sửa lỗi

Dùng cùng bộ câu hỏi giữ riêng và cùng điều kiện sinh để so SFT/PPO về chính xác, tuân thủ yêu cầu, tiếng Việt, lịch sự và giữ ngữ cảnh. Báo cả thắng/hòa/thua và ví dụ chưa cải thiện. RM dùng để train chỉ là chỉ số phụ cho đánh giá cuối.

Nếu reward chấm sai rõ ràng, chuyển cặp prompt/response và điểm cho thành viên 2. Nếu SFT cũng có lỗi tương tự, cùng thành viên 1 xem lại dữ liệu/nền. Giữ revision của từng run để không so lẫn model khác nhau.

## Bước 8. Xuất policy và bàn giao cho thành viên 4

Lưu checkpoint resume đầy đủ (trạng thái huấn luyện) riêng với model policy inference. Checkpoint phân mảnh hoặc adapter chưa chắc nạp trực tiếp vào vLLM; export thành causal LM hoàn chỉnh, giữ tokenizer/chat template và nạp lại để thử sinh.

- [ ] Policy artifact/revision, tokenizer, cấu hình sinh và giới hạn context rõ ràng.
- [ ] Có SFT/RM/dataset revisions, GPU layout, config và log/WandB run.
- [ ] Smoke run và save/reload qua; kiểm tra resume riêng nếu bàn giao khả năng tiếp tục train.
- [ ] Có báo cáo SFT/PPO và hạn chế, không chỉ model có reward cao nhất.
- [ ] Thành viên 4 xác nhận vLLM nạp được policy hoặc có lỗi export cụ thể để phối hợp xử lý.

## Cách sử dụng hướng dẫn

Các thông số ghi “theo PDF” là cấu hình tham khảo trong tài liệu gốc, chưa phải cấu hình đã kiểm thử của repo. Điều chỉnh theo GPU và phiên bản thư viện, ghi rõ thay đổi khi bàn giao. Phần chia tập đánh giá, chạy thử, kiểm tra nạp lại và nghiệm thu là bổ sung để nhóm thực hiện dự án có kiểm soát. Tài liệu mô tả nhiệm vụ, không cung cấp code triển khai đầy đủ.
