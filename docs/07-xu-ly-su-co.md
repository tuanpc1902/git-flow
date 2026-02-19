# 07 - Xử lý sự cố (Troubleshooting)

> Tổng hợp các tình huống thường gặp và cách giải quyết.

---

## 1. Commit nhầm

### 1.1 Sửa commit message cuối cùng

```bash
# CHƯA push
$ git commit --amend -m "feat: message mới đúng"

# ĐÃ push (cần force push - chỉ khi branch của mình)
$ git commit --amend -m "feat: message mới đúng"
$ git push --force-with-lease origin feature/my-branch
```

### 1.2 Quên add file vào commit cuối

```bash
$ git add forgotten-file.js
$ git commit --amend --no-edit
# Nếu đã push: git push --force-with-lease
```

### 1.3 Commit nhầm branch

```bash
# Vừa commit xong (1 commit) trên main, muốn chuyển sang feature
# Bước 1: Ghi nhớ hash
$ git log --oneline -1
> abc1234 feat: commit nhầm vào main

# Bước 2: Chuyển sang branch đúng
$ git checkout feature/login
$ git cherry-pick abc1234

# Bước 3: Xóa commit trên main
$ git checkout main
$ git reset --hard HEAD~1

# ⚠️ Nếu đã push main lên remote, dùng revert thay vì reset:
$ git revert abc1234
```

### 1.4 Commit nhầm file nhạy cảm (secrets, .env)

```bash
# ⚠️ QUAN TRỌNG: Nếu đã push, mật khẩu/key bị lộ → PHẢI đổi ngay!

# CHƯA push - Xóa file khỏi commit cuối:
$ git reset --soft HEAD~1
$ git reset HEAD secrets.json
$ echo "secrets.json" >> .gitignore
$ git add .gitignore
$ git commit -m "chore: add secrets to gitignore"

# ĐÃ push - Xóa khỏi toàn bộ lịch sử:
# Dùng git-filter-repo (cài: pip install git-filter-repo)
$ git filter-repo --path secrets.json --invert-paths

# Hoặc dùng BFG Repo-Cleaner:
$ bfg --delete-files secrets.json
$ git reflog expire --expire=now --all
$ git gc --prune=now --aggressive
$ git push --force --all
```

---

## 2. Merge/Rebase sự cố

### 2.1 Hủy merge đang có conflict

```bash
$ git merge --abort
```

### 2.2 Hủy rebase đang có conflict

```bash
$ git rebase --abort
```

### 2.3 Đã merge nhầm branch

```bash
# CHƯA push:
$ git reset --hard HEAD~1     # Nếu merge commit là commit cuối

# ĐÃ push:
$ git revert -m 1 <merge-commit-hash>
$ git push origin main
```

### 2.4 Rebase gây lỗi, muốn quay lại

```bash
# Dùng reflog để tìm trạng thái trước rebase
$ git reflog
> abc1234 HEAD@{0}: rebase (finish): ...
> def5678 HEAD@{1}: rebase (start): ...
> ghi9012 HEAD@{2}: commit: last commit before rebase  ← Đây!

$ git reset --hard ghi9012
```

### 2.5 Conflict quá nhiều, không biết fix

```bash
# Cách 1: Abort và thử cách khác
$ git merge --abort

# Cách 2: Accept tất cả từ 1 phía
$ git checkout --theirs .    # Giữ code của branch đang merge
$ git checkout --ours .      # Giữ code của branch hiện tại
$ git add .
$ git commit

# Cách 3: Dùng mergetool
$ git mergetool
# Cấu hình VS Code là mergetool:
$ git config --global merge.tool vscode
$ git config --global mergetool.vscode.cmd 'code --wait $MERGED'
```

---

## 3. Push/Pull sự cố

### 3.1 Push bị rejected

```bash
$ git push origin main
> ! [rejected] main -> main (non-fast-forward)

# Nguyên nhân: Remote có commit mà local chưa có

# Giải pháp 1: Pull rồi push
$ git pull origin main
$ git push origin main

# Giải pháp 2: Pull rebase (gọn hơn)
$ git pull --rebase origin main
$ git push origin main
```

### 3.2 Push bị rejected vì Protected Branch

```bash
$ git push origin main
> remote: GitLab: You are not allowed to push code to protected branches

# Giải pháp: Phải tạo MR
$ git checkout -b feature/fix-from-main
$ git push -u origin feature/fix-from-main
# → Tạo MR trên GitLab
```

### 3.3 Pull bị conflict

```bash
$ git pull origin main
> CONFLICT (content): Merge conflict in src/app.js

# Fix conflict → add → commit
$ git add src/app.js
$ git commit -m "fix: resolve merge conflict"

# Hoặc abort pull:
$ git merge --abort
```

### 3.4 Push quá chậm (repo lớn)

```bash
# Kiểm tra kích thước repo
$ git count-objects -vH

# Push từng batch
$ git push origin main --no-thin

# Dùng shallow clone cho CI/CD
$ git clone --depth 1 <url>
```

---

## 4. File & Working Directory sự cố

### 4.1 Mất file sau khi checkout/reset

```bash
# Dùng reflog để tìm lại
$ git reflog
$ git checkout <commit-hash> -- path/to/file
```

### 4.2 File đã xóa nhưng cần khôi phục

```bash
# Tìm commit cuối chứa file
$ git log --all --full-history -- path/to/deleted-file.js

# Khôi phục file từ commit đó
$ git checkout <commit-hash>^ -- path/to/deleted-file.js
```

### 4.3 Thay đổi bị mất sau reset --hard

```bash
# Nếu đã commit trước đó:
$ git reflog
$ git reset --hard HEAD@{N}

# Nếu CHƯA commit (chỉ modified):
# 😢 Rất khó khôi phục. Check:
# 1. IDE local history (VS Code: File → Local History)
# 2. Text editor backup/swap files
# 3. OS file recovery tools
```

### 4.4 .gitignore không hoạt động

```bash
# File đã tracked trước khi thêm vào .gitignore
# Cần xóa khỏi tracking:
$ git rm --cached <file>
$ git commit -m "chore: untrack ignored files"

# Xóa tất cả file đang tracked nhưng nên ignore:
$ git rm -r --cached .
$ git add .
$ git commit -m "chore: apply .gitignore"
```

### 4.5 Line ending issues (Windows/Mac/Linux)

```bash
# Triệu chứng: git diff hiển thị toàn bộ file thay đổi
# Nguyên nhân: Khác line ending (CRLF vs LF)

# Fix cho Windows:
$ git config --global core.autocrlf true

# Fix cho Mac/Linux:
$ git config --global core.autocrlf input

# Tạo file .gitattributes ở root project:
# ────────────────────────
# .gitattributes
* text=auto
*.js text eol=lf
*.jsx text eol=lf
*.ts text eol=lf
*.css text eol=lf
*.md text eol=lf
*.json text eol=lf
*.png binary
*.jpg binary
*.gif binary
# ────────────────────────

# Áp dụng cho file hiện tại:
$ git add --renormalize .
$ git commit -m "chore: normalize line endings"
```

---

## 5. Authentication sự cố

### 5.1 SSH key không hoạt động

```bash
# Test SSH connection
$ ssh -T git@gitlab.com
> Permission denied (publickey)

# Kiểm tra SSH agent đang chạy
$ eval "$(ssh-agent -s)"
$ ssh-add ~/.ssh/id_ed25519

# Kiểm tra key đúng
$ ssh-add -l

# Debug SSH
$ ssh -vT git@gitlab.com

# Kiểm tra key đã thêm vào GitLab
# → User Settings → SSH Keys
```

### 5.2 HTTPS credential bị sai

```bash
# Xóa credential đã lưu
# Windows:
# → Control Panel → Credential Manager → xóa entry git

# macOS:
$ git credential-osxkeychain erase

# Linux:
$ git config --global --unset credential.helper
# Hoặc xóa file ~/.git-credentials

# Set lại credential
$ git config --global credential.helper store
$ git pull   # Nhập lại username/password
```

### 5.3 Permission denied

```bash
# Kiểm tra remote URL
$ git remote -v

# Đổi từ HTTPS sang SSH (hoặc ngược lại)
$ git remote set-url origin git@gitlab.com:team/project.git

# Kiểm tra quyền trên GitLab:
# Project → Members → kiểm tra role của bạn
# Guest < Reporter < Developer < Maintainer < Owner
```

---

## 6. Detached HEAD

### 6.1 Detached HEAD là gì?

```bash
# Khi bạn checkout 1 commit (không phải branch):
$ git checkout abc1234
> You are in 'detached HEAD' state

# Hoặc checkout tag:
$ git checkout v1.0.0

# HEAD không trỏ vào branch nào → commit mới sẽ "mồ côi"
```

### 6.2 Cách thoát Detached HEAD

```bash
# Quay lại branch:
$ git checkout main

# Nếu đã commit gì đó ở detached HEAD và muốn giữ:
$ git checkout -b save-my-work    # Tạo branch mới tại đây
```

---

## 7. Git LFS (Large File Storage)

### 7.1 Khi nào cần?

Khi project có file lớn (>100MB): video, dataset, binary files...

```bash
# Cài Git LFS
$ git lfs install

# Track file lớn
$ git lfs track "*.psd"
$ git lfs track "*.zip"
$ git lfs track "data/*.csv"

# Đảm bảo .gitattributes được commit
$ git add .gitattributes
$ git commit -m "chore: setup Git LFS"

# Push bình thường
$ git add large-file.psd
$ git commit -m "add design file"
$ git push

# Xem file đang track bằng LFS
$ git lfs ls-files
```

---

## 8. Performance sự cố

### 8.1 Git chạy chậm

```bash
# Dọn dẹp repo
$ git gc                    # Garbage collection
$ git gc --aggressive       # Dọn kỹ hơn (chậm)

# Prune objects cũ
$ git prune

# Kiểm tra kích thước
$ git count-objects -vH

# Nếu repo quá lớn, dùng shallow clone:
$ git clone --depth 10 <url>

# Dùng sparse checkout (chỉ checkout 1 phần):
$ git clone --no-checkout <url>
$ cd project
$ git sparse-checkout init --cone
$ git sparse-checkout set src/my-folder
$ git checkout main
```

### 8.2 Fetch/Pull quá lâu

```bash
# Fetch chỉ branch cần
$ git fetch origin main

# Prune remote branches đã xóa
$ git remote prune origin
# hoặc
$ git fetch --prune
```

---

## 9. Bảng tra nhanh sự cố

| Sự cố | Giải pháp nhanh |
|--------|----------------|
| Commit message sai | `git commit --amend -m "msg mới"` |
| Quên add file | `git add file && git commit --amend --no-edit` |
| Commit nhầm branch | `cherry-pick` sang branch đúng + `reset` branch sai |
| Push bị rejected | `git pull --rebase && git push` |
| Merge conflict | Mở file → fix → `git add` → `git commit` |
| Muốn hủy merge | `git merge --abort` |
| Muốn hủy rebase | `git rebase --abort` |
| Mất code sau reset | `git reflog` → `git reset --hard HEAD@{N}` |
| File đã xóa cần phục hồi | `git checkout <hash>^ -- file` |
| .gitignore không work | `git rm --cached file` |
| Detached HEAD | `git checkout main` (hoặc tên branch) |
| SSH không kết nối | `ssh -vT git@gitlab.com` để debug |
| Push lên protected branch | Tạo MR thay vì push trực tiếp |

---

## 10. Checklist "Trước khi hoảng loạn"

```
Khi gặp lỗi Git, làm theo thứ tự:

1️⃣  ĐỪNG HOẢNG - Git hiếm khi mất dữ liệu thật sự
2️⃣  git status   → Xem trạng thái hiện tại
3️⃣  git log      → Xem lịch sử commits
4️⃣  git reflog   → Xem MỌI thao tác (kể cả đã "xóa")
5️⃣  git stash    → Lưu tạm thay đổi nếu cần
6️⃣  Tìm giải pháp trong tài liệu này
7️⃣  Google: "git <mô tả vấn đề>"
8️⃣  Hỏi đồng nghiệp trước khi force push!
```

---

← [06 - Workflow & Best Practices](06-workflow-best-practices.md) | **Tiếp theo**: [08 - Cheatsheet](08-cheatsheet.md) →
