# 08 - Cheatsheet (Bảng tra cứu nhanh)

> Bookmark trang này! Tra cứu nhanh khi cần.

---

## Cấu hình

```bash
git config --global user.name "Tên"           # Set tên
git config --global user.email "email"         # Set email
git config --global core.editor "code --wait"  # Set editor
git config --global init.defaultBranch main    # Branch mặc định
git config --global pull.rebase true           # Pull = rebase
git config --global core.autocrlf true         # Line ending (Win)
git config --list                              # Xem tất cả config
```

---

## Khởi tạo & Clone

```bash
git init                                       # Khởi tạo repo mới
git clone <url>                                # Clone repo
git clone <url> <folder>                       # Clone vào folder khác
git clone -b <branch> <url>                    # Clone 1 branch
git clone --depth 1 <url>                      # Shallow clone
```

---

## Staging & Commit

```bash
git status                                     # Xem trạng thái
git status -s                                  # Trạng thái rút gọn
git add <file>                                 # Stage file
git add .                                      # Stage tất cả
git add -p                                     # Stage interactive
git commit -m "message"                        # Commit
git commit -am "message"                       # Add tracked + commit
git commit --amend -m "new msg"                # Sửa commit cuối
git commit --amend --no-edit                   # Thêm file vào commit cuối
```

---

## Xem lịch sử

```bash
git log                                        # Log đầy đủ
git log --oneline                              # Log 1 dòng
git log --oneline --graph --all                # Log với graph
git log -N                                     # N commit gần nhất
git log --author="name"                        # Lọc theo author
git log --grep="keyword"                       # Tìm trong message
git log --since="2024-01-01"                   # Lọc theo thời gian
git log -- <file>                              # Log của 1 file
git log -p                                     # Log + diff
git show <commit>                              # Chi tiết 1 commit
git blame <file>                               # Ai sửa dòng nào
git reflog                                     # Lịch sử mọi thao tác
```

---

## Diff (So sánh)

```bash
git diff                                       # Working vs Staging
git diff --staged                              # Staging vs Last commit
git diff <branch1> <branch2>                   # So 2 branch
git diff <commit1> <commit2>                   # So 2 commit
git diff --name-only                           # Chỉ tên file
git diff --stat                                # Thống kê tóm tắt
```

---

## Branch

```bash
git branch                                     # Xem branch local
git branch -a                                  # Xem tất cả
git branch -v                                  # Branch + commit cuối
git branch <name>                              # Tạo branch
git checkout -b <name>                         # Tạo + chuyển
git switch -c <name>                           # Tạo + chuyển (mới)
git checkout <branch>                          # Chuyển branch
git switch <branch>                            # Chuyển branch (mới)
git checkout -                                 # Branch trước đó
git branch -m <new-name>                       # Đổi tên
git branch -d <name>                           # Xóa (đã merge)
git branch -D <name>                           # Xóa (force)
git push origin --delete <name>                # Xóa branch remote
```

---

## Merge & Rebase

```bash
git merge <branch>                             # Merge
git merge --no-ff <branch>                     # Merge (luôn tạo commit)
git merge --abort                              # Hủy merge
git rebase <branch>                            # Rebase
git rebase -i HEAD~N                           # Interactive rebase
git rebase --continue                          # Tiếp tục sau conflict
git rebase --abort                             # Hủy rebase
```

---

## Remote

```bash
git remote -v                                  # Xem remotes
git remote add <name> <url>                    # Thêm remote
git remote remove <name>                       # Xóa remote
git remote set-url <name> <url>                # Đổi URL
git fetch origin                               # Fetch
git fetch --prune                              # Fetch + xóa ref cũ
git pull origin <branch>                       # Pull (merge)
git pull --rebase origin <branch>              # Pull (rebase)
git push origin <branch>                       # Push
git push -u origin <branch>                    # Push + set upstream
git push --force-with-lease                    # Force push an toàn
git push --tags                                # Push tags
```

---

## Stash

```bash
git stash                                      # Lưu tạm
git stash push -m "message"                    # Lưu tạm + message
git stash -u                                   # Lưu cả untracked
git stash list                                 # Xem danh sách
git stash show                                 # Xem thay đổi
git stash show -p                              # Xem diff chi tiết
git stash pop                                  # Lấy ra + xóa
git stash apply                                # Lấy ra + giữ
git stash drop stash@{N}                       # Xóa 1 stash
git stash clear                                # Xóa tất cả
```

---

## Hoàn tác (Undo)

```bash
git restore <file>                             # Hủy thay đổi file
git restore --staged <file>                    # Unstage file
git restore .                                  # Hủy tất cả thay đổi
git reset --soft HEAD~1                        # Undo commit, giữ staging
git reset HEAD~1                               # Undo commit, giữ working
git reset --hard HEAD~1                        # ⚠️ Undo commit, xóa hết
git revert <commit>                            # Tạo commit đảo ngược
git cherry-pick <commit>                       # Copy commit cụ thể
git clean -fd                                  # Xóa untracked files
```

---

## Tag

```bash
git tag                                        # Xem tags
git tag -a v1.0 -m "message"                   # Tạo annotated tag
git tag v1.0                                   # Tạo lightweight tag
git show v1.0                                  # Chi tiết tag
git push origin v1.0                           # Push 1 tag
git push --tags                                # Push tất cả
git tag -d v1.0                                # Xóa tag local
git push origin --delete v1.0                  # Xóa tag remote
```

---

## Khôi phục & Debug

```bash
git reflog                                     # Xem lịch sử thao tác
git reset --hard HEAD@{N}                      # Quay lại trạng thái N
git bisect start                               # Bắt đầu tìm bug
git bisect bad                                 # Đánh dấu có bug
git bisect good <commit>                       # Đánh dấu không bug
git bisect reset                               # Kết thúc bisect
```

---

## .gitignore

```gitignore
*.log                    # Bỏ qua theo extension
node_modules/            # Bỏ qua thư mục
.env                     # Bỏ qua file cụ thể
!important.log           # Ngoại lệ
**/build/                # Bỏ qua ở mọi nơi
doc/**/*.pdf             # Bỏ qua đệ quy
```

```bash
git rm --cached <file>                         # Untrack file đã tracked
git rm -r --cached .                           # Untrack tất cả → re-add
```

---

## Alias hữu ích

```bash
git config --global alias.s "status -s"
git config --global alias.co "checkout"
git config --global alias.br "branch"
git config --global alias.ci "commit"
git config --global alias.lg "log --oneline --graph --all --decorate"
git config --global alias.last "log -1 HEAD --stat"
git config --global alias.unstage "restore --staged"
git config --global alias.undo "reset --soft HEAD~1"
```

---

## Quy trình hàng ngày (Daily Flow)

```bash
# Sáng:
git checkout main && git pull origin main
git checkout -b feature/TASK-123-description

# Trong ngày:
git add . && git commit -m "feat: description"
git push -u origin feature/TASK-123-description

# Trước MR:
git fetch origin && git rebase origin/main
git push --force-with-lease

# Sau merge:
git checkout main && git pull
git branch -d feature/TASK-123-description
git fetch --prune
```

---

← [07 - Xử lý sự cố](07-xu-ly-su-co.md) | **Tiếp theo**: [09 - Bài tập thực hành](09-bai-tap-thuc-hanh.md) →
