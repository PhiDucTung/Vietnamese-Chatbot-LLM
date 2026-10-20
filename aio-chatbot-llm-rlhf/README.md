# AIO Chatbot LLM RLHF

Khung dự án xây dựng chatbot tiếng Việt qua SFT, reward modeling, PPO RLHF và triển khai phục vụ.

## Hướng dẫn cho 4 thành viên

Xem [phân công và các bước thực hiện](docs/README.md). Mỗi thành viên có một tài liệu riêng mô tả đầu vào, các bước làm, kết quả bàn giao và tiêu chí hoàn thành, không đi vào code chi tiết.

## Phân công

- Thành viên 1: `src/sft/` — xử lý dữ liệu multiturn, SFT Llama-3.2-1B với QLoRA.
- Thành viên 2: `src/reward_model/` — huấn luyện reward model và merge LoRA.
- Thành viên 3: `src/rlhf/` — PPO với OpenRLHF / Ray.
- Thành viên 4: `src/deployment/` — vLLM server và giao diện Gradio.

## Dữ liệu và notebooks

Đặt dữ liệu gốc (Alpaca, preference, prompts) trong `data/raw/`; lưu dữ liệu sau chat template / tokenization vào `data/processed/`. Các notebook trong `notebooks/` là khung hợp lệ để phát triển và chạy trên Google Colab. Hình ảnh, sơ đồ pipeline và biểu đồ được lưu trong `assets/`.

## Cài đặt và sử dụng

Các script hiện chỉ là file khung, chưa triển khai huấn luyện hay serving. Danh sách thư viện dự kiến trong `requirements.txt` đang được chú thích; cần chọn phiên bản tương thích với GPU/CUDA và từng giai đoạn trước khi cài đặt.

```bash
python3 -m venv .venv
source .venv/bin/activate
# Sau khi chốt và bỏ chú thích các dependency cần dùng:
pip install -r requirements.txt
```

Phát triển lần lượt: chuẩn bị dữ liệu → SFT → reward model → PPO → serving. Không commit dữ liệu, checkpoint hoặc token truy cập.

## Giấy phép

Chưa chọn giấy phép. Điền giấy phép được nhóm thống nhất vào `LICENSE` trước khi phát hành.
