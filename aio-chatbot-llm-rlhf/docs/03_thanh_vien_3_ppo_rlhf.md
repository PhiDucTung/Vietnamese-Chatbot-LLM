# Thành viên 3 — Căn chỉnh chatbot bằng PPO RLHF

## Mục tiêu

Cải thiện policy SFT bằng tín hiệu từ reward model, đồng thời theo dõi mức thay đổi so với model tham chiếu để giữ chất lượng phản hồi.

Phụ trách: `src/rlhf/train_ppo.sh` và `src/rlhf/ray_cluster.py`.

## Đầu vào cần chuẩn bị

- Model SFT và tokenizer từ thành viên 1.
- Reward model đã kiểm tra từ thành viên 2.
- Bộ prompt `thuanan/Prompt-Vi-Alpaca-Preference-2k`.
- Môi trường Linux/GPU NVIDIA phù hợp OpenRLHF, Ray và các thành phần sinh rollout; Docker nếu chọn cách triển khai theo PDF.

## Các bước thực hiện

1. **Kiểm tra artifact đầu vào:** xác nhận model SFT sinh được phản hồi và reward model chấm được điểm; thống nhất tokenizer và cách format dữ liệu.
2. **Chuẩn bị môi trường:** chọn bộ phiên bản tương thích, nơi lưu checkpoint và cách theo dõi thí nghiệm. Không mặc định một GPU Colab đáp ứng cấu hình PPO trong PDF.
3. **Chuẩn bị prompt:** kiểm tra trường `context_messages`, làm sạch và tách prompt đánh giá. Prompt huấn luyện không chứa sẵn câu trả lời mục tiêu.
4. **Xác định các vai trò:** actor là policy được cập nhật; reference là SFT cố định; reward model cung cấp điểm; critic ước lượng giá trị cho PPO.
5. **Phân bổ tài nguyên:** xác định GPU cho các vai trò và vLLM, kiểm tra khả năng chạy đồng thời và cơ chế chia sẻ tài nguyên của phiên bản sử dụng.
6. **Thiết lập Ray và PPO:** cấu hình cluster, batch size, độ dài prompt/phản hồi, learning rate và hệ số KL; dùng thông số PDF làm điểm tham khảo.
7. **Chạy thử toàn bộ vòng PPO:** dùng ít prompt để kiểm tra sinh phản hồi, tính thưởng, cập nhật policy và lưu checkpoint trước khi chạy đầy đủ.
8. **Huấn luyện và theo dõi:** quan sát reward, KL, actor/critic loss, độ dài phản hồi và bộ nhớ GPU; xem trực tiếp các câu trả lời định kỳ.
9. **Đánh giá so với SFT:** dùng cùng bộ câu hỏi và điều kiện sinh để so sánh độ chính xác, tự nhiên, lịch sự và giữ ngữ cảnh. Không kết luận model tốt hơn chỉ vì reward tăng.
10. **Xuất model và bàn giao:** lưu policy dùng inference cùng tokenizer; giữ checkpoint resume riêng và kiểm tra khả năng nạp lại trước khi chuyển cho thành viên 4.

## Kết quả cần bàn giao

- Policy sau PPO ở định dạng model sinh văn bản hoàn chỉnh, kèm tokenizer.
- Checkpoint phục hồi huấn luyện, cấu hình môi trường/GPU và log thí nghiệm.
- Phiên bản SFT, RM, dữ liệu nguồn và báo cáo so sánh SFT/PPO.

## Tiêu chí hoàn thành

- Vòng PPO hoạt động và các chỉ số không xuất hiện giá trị bất thường chưa giải thích được.
- Có đánh giá phản hồi thực tế, ghi rõ cả trường hợp PPO chưa cải thiện.
- Thành viên 4 nạp được policy để triển khai; không bàn giao reward model thay policy.

Tham khảo: PDF mục II.3, trang 11–13. [Về phân công](README.md).
