# Hướng dẫn Git cho repo Vietnamese-Chatbot-LLM

Repository: [PhiDucTung/Vietnamese-Chatbot-LLM](https://github.com/PhiDucTung/Vietnamese-Chatbot-LLM).

Tài liệu dành cho nhóm 4 thành viên làm SFT, reward model, PPO và triển khai chatbot. Các lệnh dưới đây để bạn tự chạy; không chạy toàn bộ tài liệu như một script.

## 1. Vị trí chạy lệnh

Chạy lệnh Git từ **gốc repository**, nơi chứa `.git` và README chính. Mã dự án hiện nằm trong thư mục con `aio-chatbot-llm-rlhf/`. Không tạo thêm Git repo bên trong thư mục con này.

Nếu đã có repo trên máy hiện tại:

```bash
cd /Users/macbook/Vietnamese-Chatbot-LLM
git status
git branch --show-current
git remote -v
```

Nếu chưa có repo, clone từ một thư mục khác trên máy:

```bash
git clone https://github.com/PhiDucTung/Vietnamese-Chatbot-LLM.git
cd Vietnamese-Chatbot-LLM
```

Hai cách trên là lựa chọn thay thế nhau. Chủ repo cần mời các thành viên làm collaborator và họ phải chấp nhận lời mời để push branch vào repo chung.

## 2. Cấu hình lần đầu

Thay tên và email mẫu bằng thông tin của bạn. Có thể lấy email noreply trong GitHub Settings → Emails.

```bash
git config user.name "Tên của bạn"
git config user.email "email@example.com"
git config user.name
git config user.email
```

Cấu hình này chỉ áp dụng cho repo hiện tại. Xác thực push qua SSH hoặc HTTPS với token/credential manager. Không đưa token vào URL remote, mã nguồn hay ảnh chụp màn hình.

## 3. Luồng làm việc của nhóm

**Nhận nhiệm vụ → tạo branch → sửa và kiểm tra → commit → push → mở PR → review → merge vào main.**

- `main` là nhánh tích hợp đã được nhóm kiểm tra.
- Mỗi nhiệm vụ có một branch riêng, không dùng một branch cá nhân kéo dài suốt dự án.
- Commit lưu nội dung đã chọn trên máy; push mới chia sẻ commit lên GitHub.
- PR (pull request) là đề nghị review và hợp nhất code vào `main`.

| Thành viên | Nhiệm vụ và branch ví dụ |
|---|---|
| 1 — SFT | `feat/sft-dataset`, `feat/sft-training` |
| 2 — RM | `feat/rm-training`, `feat/rm-merge` |
| 3 — PPO | `feat/ppo-config`, `feat/ppo-training` |
| 4 — Triển khai | `feat/vllm-serving`, `feat/gradio-ui` |

Thống nhất trước khi sửa file chung như README, requirements hoặc cấu hình. Mỗi PR nên có một người khác review. Model lớn bàn giao qua Hugging Face/nơi lưu artifact; Git chỉ lưu code, cấu hình và thông tin model ID/revision.

## 4. Bắt đầu nhiệm vụ mới

Trước hết chạy `git status`. Nếu có thay đổi chưa lưu, commit trên branch hiện tại hoặc dùng stash ở phần 9 trước khi chuyển branch.

Khi working tree sạch:

```bash
git switch main
git pull --ff-only origin main
git switch -c feat/sft-dataset
```

Thay tên branch bằng nhiệm vụ thực tế. Nếu branch đã tồn tại và bạn muốn tiếp tục nó, dùng `git switch ten-branch` thay vì `-c`. Nếu `pull --ff-only` thất bại, kiểm tra lịch sử; không reset hoặc force push để vượt lỗi.

## 5. Stage và commit

Ví dụ bạn vừa sửa xử lý dataset:

```bash
git status
git diff
git add aio-chatbot-llm-rlhf/src/sft/dataset.py
git diff --cached --stat
git diff --cached
git commit -m "feat(sft): format multiturn conversations"
```

- `git diff` xem thay đổi chưa stage ở file đã được theo dõi; file mới chưa stage chỉ được liệt kê trong `git status`.
- `git add` chọn nội dung để commit. Nếu sửa tiếp sau đó, phải add lại để lấy phần mới.
- `git diff --cached` là nội dung chính xác sẽ được commit.
- Chọn file cụ thể thay vì dùng `git add .` khi chưa kiểm tra danh sách.

Nếu stage nhầm, bỏ stage mà giữ nguyên nội dung file:

```bash
git restore --staged aio-chatbot-llm-rlhf/src/sft/dataset.py
```

Một commit nên giải quyết một việc rõ ràng:

| Loại | Ví dụ message |
|---|---|
| Thêm chức năng | `feat(rm): add preference dataset loading` |
| Sửa lỗi | `fix(ui): preserve chat history` |
| Tài liệu | `docs: add Git workflow guide` |
| Tổ chức lại code | `refactor(sft): separate data formatting` |
| Cấu hình | `chore: update training dependencies` |

Chạy kiểm tra phù hợp trước commit. Không cần full training cho mọi thay đổi, nhưng ghi rõ trong PR nếu mới kiểm tra cú pháp hoặc chưa chạy GPU.

## 6. Push và mở pull request

Push lần đầu trên branch nhiệm vụ:

```bash
git push -u origin feat/sft-dataset
```

Những lần tiếp theo, sau khi commit thêm trên branch đó:

```bash
git push
```

Trên GitHub:

1. Mở repo → Pull requests → New pull request.
2. Chọn base `main`, compare là branch nhiệm vụ.
3. Ghi việc đã làm, cách kiểm tra, kết quả và phần còn thiếu; liên kết issue nếu có.
4. Chọn Draft PR nếu còn đang cấu hình/phát triển; chuyển Ready for review khi sẵn sàng.
5. Sửa theo góp ý bằng commit và push trên cùng branch; PR tự cập nhật.
6. Sau review, người được nhóm phân công merge chọn Squash and merge nếu repo cho phép.

Sau khi merge, từ working tree sạch:

```bash
git switch main
git pull --ff-only origin main
```

Tạo branch mới cho nhiệm vụ kế tiếp. Có thể xóa branch remote bằng nút Delete branch trên PR. Với branch local, thử `git branch -d ten-branch`; sau squash merge lệnh có thể từ chối do lịch sử khác. Giữ branch cũng không ảnh hưởng dự án; chỉ xóa bằng `-D` sau khi xác nhận không còn công việc riêng cần giữ.

## 7. Đồng bộ code và xử lý conflict

Khi đang ở branch nhiệm vụ, commit hoặc stash thay đổi trước rồi lấy code mới từ `main`:

```bash
git fetch origin
git merge origin/main
```

Nếu có conflict, `git status` chỉ ra file cần xử lý. Mở file, thống nhất nội dung cần giữ và xóa các dấu `<<<<<<<`, `=======`, `>>>>>>>`. Sau khi sửa và kiểm tra:

```bash
git add duong/dan/file-da-sua
git diff --cached
git commit
git push
```

Thay đường dẫn mẫu bằng file thật. Nếu muốn hủy lần merge đang xung đột:

```bash
git merge --abort
```

Nếu push bị từ chối vì remote branch có commit mới, fetch và kiểm tra với người cùng sửa branch. Không force push để ghi đè thay đổi của họ.

## 8. Trường hợp muốn push thẳng main

Chỉ dùng khi nhóm thống nhất cho lần setup và GitHub cho phép. Nếu `main` được bảo vệ, dùng PR. Không chạy quy trình này thay cho PR hằng ngày.

Nếu thay đổi đang ở `main` và chưa commit:

```bash
git status
git branch --show-current
```

Chỉ tiếp tục khi nhánh hiện tại đúng là `main`. Chọn file setup cần đưa lên, ví dụ với tài liệu Git này:

```bash
git add aio-chatbot-llm-rlhf/docs/05_huong_dan_git.md
git diff --cached
git commit -m "docs: add Git workflow guide"
git fetch origin
git merge origin/main
git push origin main
```

Nếu thay đổi đã commit trên branch khác, cập nhật `main` rồi merge branch đó vào; thay `ten-branch-setup` bằng tên thật:

```bash
git switch main
git pull --ff-only origin main
git merge ten-branch-setup
git push origin main
```

Nếu có conflict phải giải quyết và kiểm tra trước push. Hai trường hợp trên là lựa chọn thay thế nhau; không dùng `--force` lên `main`.

## 9. Lưu tạm và sửa sai

### Đang làm dở nhưng cần chuyển việc

```bash
git stash push -u -m "WIP: sft dataset"
git stash list
```

`-u` gồm cả file chưa được theo dõi, không gồm file bị ignore. Stash chỉ lưu local. Quay về đúng branch và kiểm tra danh sách trước khi khôi phục:

```bash
git stash apply 'stash@{0}'
```

Nếu có conflict, xử lý như phần 7. Khi xác nhận đã khôi phục đầy đủ, mới xóa đúng stash bằng `git stash drop 'stash@{0}'`.

### Sửa message commit cuối chưa push

Kiểm tra không có thay đổi staged ngoài ý muốn rồi chạy:

```bash
git commit --amend -m "docs: clarify Git workflow"
```

Với commit đã chia sẻ, ưu tiên commit sửa tiếp theo thay vì viết lại lịch sử.

### Hoàn tác commit đã chia sẻ

Xem mã commit bằng `git log --oneline -10`, tạo branch sửa từ `main` mới nhất, rồi dùng `git revert MA_COMMIT` với mã thật. Kiểm tra, push và mở PR. Với merge commit cần xác định parent riêng trước khi revert; không chạy máy móc. Tránh `git reset --hard` khi chưa hiểu vì có thể mất thay đổi local.

## 10. Những file không đưa vào commit

- Token Hugging Face, WandB key, khóa API, `.env` chứa secret.
- Dataset tải về, checkpoint, model weights và adapter lớn.
- Môi trường ảo, cache, log lớn; output notebook chứa secret/dữ liệu riêng.

`.gitignore` hiện nằm trong `aio-chatbot-llm-rlhf/`, không áp dụng cho file nằm ngoài thư mục đó. Quy tắc `.venv/` chưa bao phủ `.venv-sft/`; bổ sung ignore phù hợp nếu dùng tên khác. Kiểm tra một đường dẫn cụ thể bằng:

```bash
git check-ignore -v aio-chatbot-llm-rlhf/.env
git ls-files aio-chatbot-llm-rlhf/.env
```

`.gitignore` không tự bỏ theo dõi file đã commit. Muốn giữ file trên máy nhưng bỏ khỏi bản Git tiếp theo, dùng `git rm --cached duong/dan/file` và commit cùng quy tắc ignore; nội dung vẫn còn trong lịch sử. Nếu lộ token, thu hồi/đổi token ngay; xóa file hoặc revert không đủ.

## Checklist nhanh

- [ ] Đúng branch, đúng nhiệm vụ.
- [ ] Đã đọc staged diff và loại secret/artifact lớn.
- [ ] Commit có mục đích rõ ràng và đã kiểm tra phù hợp.
- [ ] Push đúng branch; PR mô tả đúng phần đã/chưa kiểm thử.
- [ ] Có review trước merge theo quy ước nhóm.

Tham khảo: [GitHub flow](https://docs.github.com/en/get-started/using-github/github-flow).
