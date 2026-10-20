# Hướng dẫn sử dụng Git cho repo Vietnamese-Chatbot-LLM

Repo chung: [PhiDucTung/Vietnamese-Chatbot-LLM](https://github.com/PhiDucTung/Vietnamese-Chatbot-LLM).

Tài liệu dành cho 4 thành viên thực hiện SFT, reward model, PPO và triển khai chatbot. Các lệnh là hướng dẫn để bạn tự chạy; việc thêm tài liệu này không tự commit, push hoặc cấu hình GitHub.

## 1. Hiểu cấu trúc và luồng làm việc

Hiện repository chứa thư mục dự án con `aio-chatbot-llm-rlhf/`. Chạy mọi lệnh Git trong tài liệu từ **gốc repository**, nơi chứa thư mục `.git`; các đường dẫn ví dụ giữ nguyên thư mục con. Không chạy `git init` thêm bên trong thư mục dự án.

- `main`: nhánh tích hợp các thay đổi đã kiểm tra và review.
- Branch nhiệm vụ: nơi phát triển một chức năng hoặc sửa một lỗi.
- Commit: bản ghi thay đổi đã chọn, lưu trên máy của bạn.
- Push: đưa commit lên GitHub.
- Pull request (PR): đề nghị review và hợp nhất branch nhiệm vụ vào `main`.

Luồng chung: **Issue → branch → sửa và kiểm tra → commit → push → PR → review → merge**.

Mỗi nhiệm vụ có một branch; không giữ một branch cố định cho mỗi người suốt dự án. Nhóm chưa cần nhánh `develop`.

## 2. Chuẩn bị lần đầu

### Nếu máy đã có repo

Trên máy hiện tại:

```bash
cd /Users/macbook/Vietnamese-Chatbot-LLM
git status
git remote -v
git branch --show-current
```

Kiểm tra `origin` trỏ tới repo ở đầu tài liệu. Không clone thêm vào bên trong repo đang có.

### Nếu thành viên chưa có repo

Chạy ở thư mục muốn lưu dự án trên máy thành viên đó:

```bash
git clone https://github.com/PhiDucTung/Vietnamese-Chatbot-LLM.git
cd Vietnamese-Chatbot-LLM
```

Chủ repo cần mời các thành viên làm collaborator trên GitHub để họ push branch vào repo chung. Chỉ các file đã commit và push mới xuất hiện khi người khác clone; file mới chưa được theo dõi trên máy chủ repo chưa được chia sẻ.

### Thiết lập danh tính commit

Thay tên và email mẫu bằng thông tin của bạn; có thể dùng email noreply từ GitHub Settings → Emails nếu muốn giữ riêng email cá nhân.

```bash
git config user.name "Tên của bạn"
git config user.email "email@example.com"
git config user.name
git config user.email
```

Cấu hình trên chỉ áp dụng cho repo hiện tại. Xác thực push bằng SSH hoặc HTTPS với token/credential manager; không dùng mật khẩu tài khoản GitHub làm token và không ghi token vào URL remote.

## 3. Đưa khung dự án hiện tại lên GitHub

Phần này dành cho người đang giữ các file khung chưa commit. Kiểm tra trạng thái trước; nếu đã có PR setup được merge thì bỏ qua và dùng quy trình ở phần 4.

```bash
git status
git switch -c chore/project-setup
```

Branch mới giữ nguyên các file chưa commit đang có. Xem danh sách file, rồi stage khung dự án:

```bash
git add aio-chatbot-llm-rlhf/README.md aio-chatbot-llm-rlhf/.gitignore
git add aio-chatbot-llm-rlhf/requirements.txt aio-chatbot-llm-rlhf/LICENSE
git add aio-chatbot-llm-rlhf/src aio-chatbot-llm-rlhf/docs
git add aio-chatbot-llm-rlhf/data aio-chatbot-llm-rlhf/assets
git diff --cached --stat
git diff --cached
```

Chỉ commit sau khi đã xác nhận không có dataset, model, môi trường ảo hoặc secret trong nội dung staged:

```bash
git commit -m "chore: add project scaffold and team guides"
git push -u origin chore/project-setup
```

Mở PR theo phần 7. Không cần chuyển cấu trúc thư mục để dùng quy trình Git này.

## 4. Bắt đầu một nhiệm vụ mới

Trước khi chuyển branch, chạy `git status`. Nếu có việc đang làm, hãy commit trên branch hiện tại hoặc lưu bằng stash theo phần 9; không bỏ thay đổi để chuyển branch cho nhanh.

Khi working tree sạch:

```bash
git switch main
git pull --ff-only origin main
git switch -c feat/sft-dataset
```

`--ff-only` chỉ cập nhật khi có thể tiến thẳng theo lịch sử remote; nếu thất bại, kiểm tra lịch sử thay vì reset hoặc force push.

| Người phụ trách | Branch nhiệm vụ gợi ý |
|---|---|
| Thành viên 1 | `feat/sft-dataset`, sau đó `feat/sft-training` |
| Thành viên 2 | `feat/rm-data`, `feat/rm-training`, `feat/rm-merge` |
| Thành viên 3 | `feat/ppo-config`, `feat/ppo-training` |
| Thành viên 4 | `feat/vllm-serving`, `feat/gradio-ui` |

Các tên là ví dụ; thay theo nhiệm vụ thực tế. Mỗi issue nên có người phụ trách, mô tả đầu ra và tiêu chí hoàn thành. Nếu branch cùng tên đã tồn tại và bạn muốn tiếp tục nó, dùng `git switch ten-branch` thay vì tạo lại bằng `-c`.

## 5. Chọn thay đổi và commit

Ví dụ sau khi sửa phần xử lý dataset:

```bash
git status
git diff
git add aio-chatbot-llm-rlhf/src/sft/dataset.py
git diff --cached --stat
git diff --cached
git commit -m "feat(sft): format multiturn conversations"
```

- `git diff`: nội dung chưa stage của file đã được Git theo dõi; file mới chưa stage được liệt kê trong `git status`.
- `git add`: chọn nội dung cho commit tiếp theo.
- `git diff --cached`: kiểm tra chính xác nội dung đã chọn.
- Nếu sửa file thêm sau khi stage, cần `git add` lại để đưa phần mới vào commit.
- Chỉ stage file liên quan; tránh `git add .` khi chưa xem danh sách file.

Chọn riêng từng phần thay đổi trong một file:

```bash
git add -p aio-chatbot-llm-rlhf/src/sft/dataset.py
```

Bỏ stage mà vẫn giữ nội dung đã sửa:

```bash
git restore --staged aio-chatbot-llm-rlhf/src/sft/dataset.py
```

Mỗi commit giải quyết một việc có ý nghĩa. Quy ước message của nhóm:

| Tiền tố | Mục đích | Ví dụ |
|---|---|---|
| `feat` | Thêm chức năng | `feat(rm): add preference data loading` |
| `fix` | Sửa lỗi | `fix(ui): preserve conversation history` |
| `docs` | Tài liệu | `docs: explain Git workflow` |
| `refactor` | Tổ chức lại code | `refactor(sft): separate data formatting` |
| `chore` | Cấu hình, dependency | `chore: update training dependencies` |

Không cần huấn luyện full model trước mọi commit. Kiểm tra đúng phần đã sửa, và ghi rõ smoke test nào đã chạy, phần nào chưa kiểm chứng khi mở PR.

## 6. Push và đồng bộ với nhóm

Push branch lần đầu:

```bash
git push -u origin feat/sft-dataset
```

Sau các commit tiếp theo trên cùng branch:

```bash
git push
```

Để lấy thay đổi mới từ `main`, commit hoặc stash công việc trước rồi chạy trên branch nhiệm vụ:

```bash
git fetch origin
git merge origin/main
```

`fetch` cập nhật thông tin remote; `merge` đưa thay đổi từ `main` vào branch đang làm. Sau khi kiểm tra kết quả merge, push branch. Không cần rebase hoặc force push trong quy trình cơ bản của nhóm.

Nếu push bị từ chối vì remote branch có commit mới, fetch rồi xem lịch sử và trao đổi với người cùng sửa branch. Không dùng force push để ghi đè công việc của họ.

## 7. Tạo PR, review và merge

Trên trang GitHub của repo:

1. Chọn **Compare & pull request**, hoặc tab **Pull requests → New pull request**.
2. Chọn base `main`, compare là branch nhiệm vụ.
3. Viết mục tiêu, nội dung thay đổi, cách kiểm tra và hạn chế; liên kết issue nếu có.
4. Chọn **Draft** nếu còn đang phát triển; nhờ ít nhất một người khác review khi đã sẵn sàng.
5. Tiếp nhận góp ý bằng cách sửa, commit và push trên cùng branch; PR tự cập nhật.
6. Sau khi review đạt và các thảo luận đã xử lý, người phụ trách merge chọn **Squash and merge** nếu repo cho phép.

Không ghi “đã huấn luyện thành công” khi mới kiểm tra cú pháp hoặc chưa chạy GPU. Quy tắc một review là quy ước nhóm; chỉ được GitHub bắt buộc nếu chủ repo đã cấu hình bảo vệ `main` phù hợp quyền/gói đang dùng.

Sau khi PR được merge, working tree sạch thì cập nhật local:

```bash
git switch main
git pull --ff-only origin main
```

Có thể xóa branch remote bằng nút **Delete branch** trên PR. Xóa branch local bằng `git branch -d ten-branch`. Sau squash merge, `-d` có thể từ chối do lịch sử khác; có thể giữ branch lại. Chỉ dùng `git branch -D ten-branch` khi đã xác nhận PR được merge và không còn công việc riêng cần giữ.

## 8. Khi gặp merge conflict

Chạy `git status` để xem file xung đột. Mở từng file, thống nhất nội dung cần giữ và xóa các dấu `<<<<<<<`, `=======`, `>>>>>>>`.

Ví dụ conflict ở dataset:

```bash
git status
git add aio-chatbot-llm-rlhf/src/sft/dataset.py
git diff --cached
git commit
git push
```

Chỉ chạy phần stage/commit sau khi đã sửa xong và kiểm tra. Không chọn toàn bộ “ours/theirs” nếu chưa hiểu phần thay đổi của người kia. Nếu chưa thể giải quyết và muốn hủy lần merge đang xung đột:

```bash
git merge --abort
```

## 9. Lưu tạm và sửa sai

### Cần chuyển việc nhưng chưa muốn commit

```bash
git stash push -u -m "WIP: sft dataset"
git stash list
```

`-u` lưu cả file chưa được Git theo dõi, không bao gồm file đã bị ignore. Stash chỉ lưu local, không phải bản sao lưu trên GitHub. Để khôi phục, quay về đúng branch và kiểm tra danh sách stash:

```bash
git stash apply 'stash@{0}'
```

Lệnh có thể gây conflict nếu code đã thay đổi. Sau khi xác nhận công việc đã khôi phục đầy đủ, mới xóa đúng stash bằng `git stash drop 'stash@{0}'`.

### Muốn sửa message của commit cuối chưa push

Đảm bảo không có nội dung staged ngoài ý muốn rồi chạy:

```bash
git commit --amend -m "feat(sft): format multiturn conversations"
```

Nếu commit đã chia sẻ trên branch chung, ưu tiên tạo commit sửa tiếp theo thay vì viết lại lịch sử.

### Cần hoàn tác một commit đã chia sẻ

```bash
git log --oneline -10
```

Tạo branch sửa lỗi từ `main` mới nhất, sau đó dùng `git revert MA_COMMIT` với mã commit thực. Revert tạo một commit đảo thay đổi; kiểm tra, push và mở PR như bình thường. Với merge commit cần xác định parent trước, không dùng máy móc lệnh revert thông thường. Không dùng `git reset --hard` để xử lý lỗi trên branch chung.

## 10. File nào đưa lên GitHub?

| Đưa vào Git | Giữ ngoài Git |
|---|---|
| Mã nguồn, script và docs | Dataset tải về, checkpoint, model weights, adapter |
| Cấu hình có placeholder | `.env`, HF token, WandB key, API key |
| Dependency và hướng dẫn cài | Môi trường ảo, cache, log lớn |
| Notebook với output gọn, không secret | Output notebook chứa secret hoặc dữ liệu riêng |
| `.gitkeep` | Artifact huấn luyện lớn |

Model/dữ liệu lớn lưu ở Hugging Face hoặc nơi lưu artifact của nhóm; trong repo ghi ID, revision và cấu hình cần tái tạo.

`.gitignore` dự án hiện ở trong `aio-chatbot-llm-rlhf/`, nên không bảo vệ file đặt bên ngoài thư mục đó. Mẫu `.venv/` cũng chưa bao phủ `.venv-sft/`. Kiểm tra ignore cho file thực tế, ví dụ:

```bash
git check-ignore -v aio-chatbot-llm-rlhf/.env
```

Lệnh không có output nghĩa là đường dẫn không khớp quy tắc ignore hoặc đã được theo dõi; kiểm tra thêm `git ls-files`. Cần bổ sung quy tắc đúng vị trí trước khi stage file mới.

File đã commit không tự ngừng được theo dõi khi thêm `.gitignore`. Muốn giữ file local nhưng bỏ khỏi bản Git tiếp theo, dùng `git rm --cached duong/dan/file`, rồi commit cùng quy tắc ignore phù hợp. Thao tác này không xóa nội dung khỏi lịch sử cũ. Nếu lộ token, phải thu hồi/đổi token ngay; xóa file hoặc revert chưa đủ.

## 11. Quy ước phối hợp cho 4 thành viên

- Mỗi nhiệm vụ có một người phụ trách chính; không cùng push vào branch của nhau nếu chưa thống nhất.
- Báo trước khi sửa file chung như README, dependency hoặc cấu hình pipeline.
- PR nhỏ, tập trung, được review sớm; không chờ cả giai đoạn huấn luyện lớn hoàn thành mới chia sẻ code.
- Thành viên 2/3/4 có thể chuẩn bị dữ liệu, cấu hình, giao diện trong lúc chờ model; ghi rõ phần còn chờ kiểm thử.
- Không push thẳng `main`, force push branch chung, hoặc tự đổi cấu trúc dự án khi nhóm chưa thống nhất.

## Checklist trước mỗi commit và PR

- [ ] Đúng branch nhiệm vụ.
- [ ] Chỉ stage thay đổi liên quan, đã đọc `git diff --cached`.
- [ ] Không có secret, model, dữ liệu lớn hoặc môi trường ảo.
- [ ] Message nói rõ việc thay đổi; đã kiểm tra phù hợp với phần việc.
- [ ] PR ghi kết quả thực tế, phần chưa kiểm thử và artifact liên quan.

Tham khảo: [GitHub flow](https://docs.github.com/en/get-started/using-github/github-flow), [bảo vệ branch](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches).

[Quay về mục lục](README.md)
