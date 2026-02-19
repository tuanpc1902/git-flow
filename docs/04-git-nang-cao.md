# 04 - Git Nâng Cao

## 1. Stash (Lưu tạm thay đổi)

### 1.1 Khi nào dùng Stash?

Khi bạn đang code dở nhưng cần **chuyển branch gấp**:

```
Tình huống:
- Bạn đang code feature/login (chưa xong, chưa muốn commit)
- Có bug critical trên main cần fix ngay
- → Stash code đang dở → chuyển branch → fix bug → quay lại → pop stash
```

### 1.2 Các lệnh Stash

```bash
# Lưu tạm tất cả thay đổi (tracked files)
$ git stash
> Saved working directory and index state WIP on feature/login: a1b2c3d feat: add login

# Lưu với message mô tả (KHUYẾN NGHỊ)
$ git stash push -m "WIP: form login đang làm nửa chừng"

# Lưu kèm cả file mới (untracked)
$ git stash -u
# hoặc
$ git stash --include-untracked

# Lưu cả file trong .gitignore
$ git stash -a
# hoặc
$ git stash --all

# Stash chỉ 1 số file cụ thể
$ git stash push -m "stash login form" -- src/login.js src/login.css

# Stash interactive (chọn từng phần)
$ git stash push -p
```

### 1.3 Xem danh sách Stash

```bash
$ git stash list
> stash@{0}: On feature/login: WIP: form login đang làm nửa chừng
> stash@{1}: WIP on main: e4f5g6h fix: typo
> stash@{2}: On feature/signup: WIP: registration form

# Xem chi tiết stash
$ git stash show
> src/login.js | 15 +++++++++++++++
> src/login.css | 8 ++++++++
> 2 files changed, 23 insertions(+)

# Xem diff chi tiết
$ git stash show -p
$ git stash show -p stash@{1}
```

### 1.4 Khôi phục Stash

```bash
# Lấy stash mới nhất ra VÀ xóa khỏi stash list
$ git stash pop

# Lấy stash mới nhất ra, KHÔNG xóa khỏi stash list
$ git stash apply

# Lấy stash cụ thể
$ git stash pop stash@{2}
$ git stash apply stash@{1}

# Xóa 1 stash
$ git stash drop stash@{1}

# Xóa TẤT CẢ stash (⚠️ không thể hoàn tác!)
$ git stash clear

# Tạo branch từ stash
$ git stash branch feature/login-v2 stash@{0}
# → Tạo branch mới + apply stash + xóa stash
```

### 💡 Tips: Stash Workflow

```bash
# 1. Đang code feature/login dở dang
$ git stash push -m "WIP: login form đang làm"

# 2. Chuyển sang fix bug
$ git checkout main
$ git checkout -b hotfix/critical-bug
# ... fix bug ...
$ git commit -m "hotfix: fix critical bug"
$ git push origin hotfix/critical-bug

# 3. Quay lại feature
$ git checkout feature/login
$ git stash pop
# → Code cũ được khôi phục, tiếp tục làm
```

---

## 2. Cherry-pick (Chọn commit cụ thể)

### 2.1 Cherry-pick là gì?

Lấy **1 commit cụ thể** từ branch khác áp dụng vào branch hiện tại:

```
main:          A ── B ── C
                         ↑ cherry-pick
feature:       D ── E ── F ── G

→ Chỉ lấy commit F, không lấy D, E, G
```

### 2.2 Sử dụng Cherry-pick

```bash
# Cherry-pick 1 commit
$ git cherry-pick abc1234

# Cherry-pick nhiều commit
$ git cherry-pick abc1234 def5678 ghi9012

# Cherry-pick một range commits
$ git cherry-pick abc1234..ghi9012    # Không bao gồm abc1234
$ git cherry-pick abc1234^..ghi9012   # Bao gồm abc1234

# Cherry-pick không tự commit (chỉ stage thay đổi)
$ git cherry-pick --no-commit abc1234

# Khi bị conflict:
# 1. Fix conflict
# 2. git add <file>
# 3. git cherry-pick --continue

# Hủy cherry-pick
$ git cherry-pick --abort
```

### 2.3 Khi nào dùng Cherry-pick?

```bash
# Scenario 1: Hot fix từ develop sang main
# Bug đã fix trên develop (commit xyz789), cần đưa lên main ngay
$ git checkout main
$ git cherry-pick xyz789

# Scenario 2: Lấy 1 feature nhỏ từ branch khác
$ git checkout feature/dashboard
$ git cherry-pick abc123    # Lấy commit utility function từ branch khác

# Scenario 3: Commit nhầm branch
# Đã commit trên main, nhưng nên ở feature branch
$ git log --oneline -1   # Ghi nhớ hash: abc123
$ git checkout feature/login
$ git cherry-pick abc123
$ git checkout main
$ git reset --hard HEAD~1   # Xóa commit nhầm trên main
```

---

## 3. Reset (Quay lại commit trước)

### ⚠️ CẢNH BÁO: Reset có thể mất code! Hiểu rõ trước khi dùng.

### 3.1 Ba chế độ Reset

```bash
# Cấu trúc:
$ git reset [--soft | --mixed | --hard] <commit>
```

```
                     Working Dir    Staging    Repository
─────────────────────────────────────────────────────────
git reset --soft       Giữ          Giữ       ← Quay lại
git reset --mixed      Giữ          Reset     ← Quay lại    (mặc định)
git reset --hard       Reset        Reset     ← Quay lại    ⚠️ MẤT CODE
```

### 3.2 Ví dụ cụ thể

```bash
# Giả sử lịch sử:
# A ── B ── C ── D (HEAD)

# === --soft: Quay lại C, giữ thay đổi ở staging ===
$ git reset --soft HEAD~1
# Kết quả: HEAD ở C, thay đổi của D nằm trong staging area
# Hữu ích khi: Muốn sửa commit message hoặc gộp commits

# === --mixed (mặc định): Quay lại C, giữ thay đổi ở working dir ===
$ git reset HEAD~1
# hoặc
$ git reset --mixed HEAD~1
# Kết quả: HEAD ở C, thay đổi của D nằm trong working directory
# Hữu ích khi: Muốn stage lại từ đầu

# === --hard: Quay lại C, XÓA HẾT thay đổi ===
$ git reset --hard HEAD~1
# ⚠️ Kết quả: HEAD ở C, thay đổi của D bị XÓA HOÀN TOÀN
# Hữu ích khi: Muốn bỏ commit hoàn toàn
```

### 3.3 Các cách chỉ định commit

```bash
# HEAD~N: N commit trước HEAD
$ git reset --soft HEAD~1     # 1 commit trước
$ git reset --soft HEAD~3     # 3 commit trước

# Commit hash
$ git reset --soft abc1234

# Branch name
$ git reset --soft origin/main
```

### 3.4 Reset vs Revert

```bash
# Reset: XÓA commit (viết lại lịch sử) - dùng cho local
#   A ── B ── C ── D
#   → git reset --hard HEAD~2
#   A ── B

# Revert: TẠO commit đảo ngược (giữ lịch sử) - dùng cho remote
#   A ── B ── C ── D
#   → git revert HEAD~1
#   A ── B ── C ── D ── D'    (D' đảo ngược C)
```

---

## 4. Revert (Hoàn tác an toàn)

```bash
# Revert commit cuối
$ git revert HEAD

# Revert commit cụ thể
$ git revert abc1234

# Revert nhiều commit
$ git revert abc1234..def5678

# Revert merge commit
$ git revert -m 1 <merge-commit-hash>
# -m 1: giữ parent 1 (thường là main)

# Revert không tự commit
$ git revert --no-commit abc1234

# Hủy revert
$ git revert --abort
```

**Khi nào dùng Reset vs Revert?**

| | Reset | Revert |
|---|---|---|
| Viết lại lịch sử | Có | Không |
| An toàn cho shared branch | ❌ | ✅ |
| Dùng khi | Chưa push | Đã push |
| Tạo commit mới | Không | Có |

---

## 5. Reflog (Lịch sử mọi thao tác - "Mạng lưới an toàn")

### 5.1 Reflog là gì?

Reflog ghi lại **MỌI thao tác** bạn làm với HEAD, kể cả reset, rebase, checkout. Đây là **cứu cánh cuối cùng** khi bạn lỡ tay.

```bash
$ git reflog
> a1b2c3d HEAD@{0}: commit: feat: thêm login
> e4f5g6h HEAD@{1}: checkout: moving from main to feature/login
> i7j8k9l HEAD@{2}: reset: moving to HEAD~3     ← Reset nhầm!
> m1n2o3p HEAD@{3}: commit: feat: quan trọng     ← Commit "biến mất"
> q5r6s7t HEAD@{4}: commit: feat: cũng quan trọng
```

### 5.2 Khôi phục sau khi Reset nhầm

```bash
# Oops! Vừa reset --hard mất code
$ git reset --hard HEAD~3
# 😱 3 commit biến mất!

# Đừng hoảng! Dùng reflog:
$ git reflog
> abc1234 HEAD@{0}: reset: moving to HEAD~3
> def5678 HEAD@{1}: commit: commit quan trọng 3
> ghi9012 HEAD@{2}: commit: commit quan trọng 2
> jkl3456 HEAD@{3}: commit: commit quan trọng 1

# Quay lại trạng thái trước reset:
$ git reset --hard def5678
# ✅ Code đã quay lại!

# Hoặc dùng HEAD@{N}:
$ git reset --hard HEAD@{1}
```

### 5.3 Khôi phục branch đã xóa

```bash
# Xóa branch nhầm
$ git branch -D feature/important
> Deleted branch feature/important (was abc1234)

# Khôi phục:
$ git reflog | grep "feature/important"
$ git checkout -b feature/important abc1234
# ✅ Branch đã quay lại!
```

---

## 6. Bisect (Tìm commit gây bug)

### 6.1 Khi nào dùng Bisect?

Khi bạn biết code **đang lỗi** nhưng không biết **commit nào gây ra**. Bisect dùng **binary search** để tìm nhanh.

```bash
# Bắt đầu bisect
$ git bisect start

# Đánh dấu commit hiện tại là BAD (có bug)
$ git bisect bad

# Đánh dấu commit cũ mà bạn biết chắc không bug
$ git bisect good abc1234

# Git sẽ checkout commit giữa, bạn test:
> Bisecting: 5 revisions left to test after this
> [def5678] feat: some change

# Test code, nếu có bug:
$ git bisect bad

# Nếu không có bug:
$ git bisect good

# Lặp lại cho đến khi tìm được:
> abc1234 is the first bad commit
> commit abc1234
> Author: Someone
> Date: ...
> 
>     feat: this commit introduced the bug

# Kết thúc bisect (quay lại HEAD ban đầu)
$ git bisect reset
```

### 6.2 Bisect tự động (với script)

```bash
# Tự động bisect bằng test script
$ git bisect start HEAD abc1234
$ git bisect run npm test
# Git tự động chạy "npm test" ở mỗi commit
# Pass (exit 0) = good, Fail (exit != 0) = bad
```

---

## 7. Worktree (Nhiều working directory)

### 7.1 Khi nào dùng Worktree?

Khi bạn cần làm việc trên **2 branch cùng lúc** mà không muốn stash/commit dở dang.

```bash
# Tạo worktree cho branch khác
$ git worktree add ../project-hotfix hotfix/critical-bug
# → Tạo thư mục ../project-hotfix chứa code của branch hotfix

# Tạo worktree với branch mới
$ git worktree add -b feature/new ../project-feature

# Xem danh sách worktrees
$ git worktree list
> /path/project              abc1234 [main]
> /path/project-hotfix       def5678 [hotfix/critical-bug]
> /path/project-feature      ghi9012 [feature/new]

# Xóa worktree
$ git worktree remove ../project-hotfix

# Dọn dẹp worktree đã xóa thủ công
$ git worktree prune
```

---

## 8. Submodule (Repo trong repo)

### 8.1 Khi nào dùng Submodule?

Khi project của bạn phụ thuộc vào **repo bên ngoài** (thư viện chung, shared components).

```bash
# Thêm submodule
$ git submodule add git@gitlab.com:team/shared-lib.git libs/shared-lib

# Clone repo có submodule
$ git clone --recurse-submodules git@gitlab.com:team/project.git

# Nếu đã clone rồi, init submodule:
$ git submodule init
$ git submodule update

# hoặc gộp:
$ git submodule update --init --recursive

# Cập nhật submodule lên version mới nhất
$ git submodule update --remote

# Xem trạng thái submodule
$ git submodule status
```

---

## 9. Alias (Shortcut cho lệnh dài)

### 9.1 Tạo Alias

```bash
# Alias đơn giản
$ git config --global alias.s "status -s"
$ git config --global alias.co "checkout"
$ git config --global alias.br "branch"
$ git config --global alias.ci "commit"

# Dùng:
$ git s       # = git status -s
$ git co main # = git checkout main

# Alias phức tạp
$ git config --global alias.lg "log --oneline --graph --all --decorate"
$ git config --global alias.last "log -1 HEAD --stat"
$ git config --global alias.unstage "restore --staged"
$ git config --global alias.undo "reset --soft HEAD~1"

# Alias với shell command (bắt đầu bằng !)
$ git config --global alias.cleanup '!git branch --merged | grep -v main | xargs git branch -d'

# Dùng:
$ git lg        # Log đẹp với graph
$ git last      # Xem commit cuối
$ git unstage . # Unstage tất cả
$ git undo      # Undo commit cuối (giữ changes)
$ git cleanup   # Xóa branch đã merge
```

### 9.2 Alias khuyến nghị cho developer

```bash
# Thêm vào ~/.gitconfig:
[alias]
    s = status -s
    co = checkout
    br = branch
    ci = commit
    lg = log --oneline --graph --all --decorate
    last = log -1 HEAD --stat
    unstage = restore --staged
    undo = reset --soft HEAD~1
    amend = commit --amend --no-edit
    wip = !git add -A && git commit -m 'WIP'
    save = !git add -A && git commit -m 'SAVEPOINT'
    branches = branch -a
    tags = tag -l
    stashes = stash list
    remotes = remote -v
    df = diff
    dfs = diff --staged
    pullr = pull --rebase
```

---

## 10. Tóm tắt

| Lệnh | Mô tả | Mức độ an toàn |
|-------|--------|----------------|
| `git stash` | Lưu tạm thay đổi | ✅ An toàn |
| `git stash pop` | Khôi phục stash | ✅ An toàn |
| `git cherry-pick` | Lấy commit cụ thể | ✅ An toàn |
| `git revert` | Hoàn tác (tạo commit mới) | ✅ An toàn |
| `git reset --soft` | Quay lại, giữ staging | ⚠️ Thay đổi lịch sử |
| `git reset --mixed` | Quay lại, giữ working dir | ⚠️ Thay đổi lịch sử |
| `git reset --hard` | Quay lại, xóa hết | ❌ Nguy hiểm |
| `git reflog` | Xem lịch sử mọi thao tác | ✅ Chỉ đọc |
| `git bisect` | Tìm commit gây bug | ✅ An toàn |
| `git worktree` | Nhiều thư mục làm việc | ✅ An toàn |

---

← [03 - Remote Repository](03-remote-repository.md) | **Tiếp theo**: [05 - GitLab](05-gitlab.md) →
