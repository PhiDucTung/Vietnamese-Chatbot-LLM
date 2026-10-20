# Hướng dẫn công việc cho 4 thành viên

Tài liệu mô tả các bước cần làm, kết quả cần đạt và nội dung bàn giao; không đi vào code hoặc lệnh chạy chi tiết. Mã nguồn dự án hiện là khung để từng thành viên triển khai.

## Phân công

| Thành viên | Phụ trách | Hướng dẫn |
|---|---|---|
| 1 | Dữ liệu hội thoại và SFT | [Các bước SFT](01_thanh_vien_1_sft.md) |
| 2 | Mô hình điểm thưởng | [Các bước reward model](02_thanh_vien_2_reward_model.md) |
| 3 | Căn chỉnh bằng PPO RLHF | [Các bước PPO](03_thanh_vien_3_ppo_rlhf.md) |
| 4 | Serving và giao diện chatbot | [Các bước triển khai](04_thanh_vien_4_deployment.md) |

## Làm việc trên GitHub

Xem [hướng dẫn Git cho repo](05_huong_dan_git.md) để clone, tạo branch, commit, push, mở PR và xử lý conflict.

## Thứ tự phối hợp

1. Thành viên 1 chuẩn bị dữ liệu và huấn luyện SFT, bàn giao model cho thành viên 2 và 3.
2. Thành viên 2 huấn luyện reward model từ model SFT, bàn giao cho thành viên 3.
3. Thành viên 3 dùng model SFT và reward model để huấn luyện PPO, bàn giao policy cuối cho thành viên 4.
4. Thành viên 4 triển khai chatbot; cả nhóm kiểm thử và so sánh chất lượng trước/sau PPO.

Trong thời gian chờ model, thành viên 2 và 3 có thể chuẩn bị dữ liệu, môi trường; thành viên 4 có thể thiết kế giao diện.

Mỗi lần bàn giao cần có model và tokenizer, thông tin phiên bản, cấu hình đã dùng, kết quả kiểm tra và hạn chế còn gặp. Không đưa token truy cập, dữ liệu lớn hoặc checkpoint vào Git.

## Tài liệu tham chiếu

Dựa trên PDF `[Description_v2]-Project_Vietnamses_Chatbot_using_LLM.pdf`, phần II (trang 3–15) và rubric phần IV (trang 19). Các bước kiểm tra và bàn giao là bổ sung cho việc phối hợp nhóm. Cấu hình trong PDF là tham khảo, cần điều chỉnh theo môi trường thực tế; các hướng dẫn này chưa chứng nhận pipeline đã chạy thành công.

[Quay về dự án](../README.md)
